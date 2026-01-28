# Specifica di Rete – Event Transport Layer (ETL)  
## Rete Domotica Cablate Event-Driven con Gateway MQTT

---

## 1. Introduzione

Questa specifica descrive il **livello di comunicazione** di una rete domotica cablata composta da ~40 nodi embedded, progettata per essere:

- **affidabile**
- **a bassa latenza**
- **resiliente**
- **funzionante anche senza gateway o cloud**

La rete è basata su **bus cablato condiviso (RS-485)** e adotta un **modello event-driven**, in cui i nodi reagiscono a eventi pubblicati sul bus secondo una logica programmata in fase di configurazione.

Un **gateway MQTT** osserva il traffico di rete per esporre lo stato a Home Assistant (HA) e può anche **iniettare comandi espliciti** provenienti dall’esterno.  
La rete deve rimanere stabile e coerente anche in presenza del gateway, ma **non deve dipendere da esso** per il funzionamento locale.

---

## 2. Obiettivi e requisiti principali (indice)

La rete deve soddisfare i seguenti requisiti:

1. **Architettura event-driven**
2. **Supporto a due semantiche di evento**
   - eventi dichiarativi
   - comandi espliciti
3. **QoS selezionabile (QoS0 / QoS1)**
4. **Consegna affidabile dei comandi (ACK mirati)**
5. **Multicast affidabile senza tempeste di ACK**
6. **Bassa latenza per eventi critici (< 50 ms)**
7. **Multi-master senza nodo centrale**
8. **Gestione collisioni e priorità**
9. **Idempotenza e deduplica**
10. **Heartbeat e rilevamento nodi**
11. **Osservabilità dello stato di rete**
12. **Integrazione con Home Assistant tramite MQTT**
13. **Assenza di consenso forte (no RAFT)**
14. **Semplicità e assenza di over-engineering**

---

## 3. Architettura generale della rete

### 3.1 Bus fisico
- Bus cablato **RS-485 half-duplex**
- Tutti i nodi sono connessi allo stesso mezzo condiviso
- Ogni frame trasmesso è ricevuto da tutti i nodi

### 3.2 Modello logico
- **Pub/Sub su bus condiviso**
- Tutti ascoltano sempre
- Un nodo reagisce **solo** se:
  - il messaggio è semanticamente rilevante per lui
  - e, se richiesto, risponde con ACK

---

## 4. Modello event-driven

La rete è **event-driven**, non command-driven puro.

Un messaggio sul bus rappresenta un **evento** che descrive *cosa è successo* oppure *cosa deve essere fatto*, non una sequenza di azioni da coordinare a runtime.

La complessità è spostata:
- **in fase di programmazione/configurazione**
- non in fase di esecuzione

---

## 5. Due semantiche di evento

### 5.1 Eventi dichiarativi (event-driven locali)

Sono eventi che descrivono un fatto, non un’azione.

**Esempi:**
- `BTN_CUCINA_PRESSED`
- `SENSOR_PRESENCE_ON`
- `TEMP_ABOVE_23`

**Caratteristiche:**
- payload minimo o nullo
- non descrivono cosa fare
- ogni nodo che ha una binding locale decide autonomamente l’azione
- tipicamente generati da pulsanti e sensori
- usati per logica stabile e cablata

**Affidabilità:**
- QoS0 o QoS1
- se QoS1, il mittente conosce a priori l’elenco dei nodi che devono rispondere

---

### 5.2 Eventi espliciti (comandi diretti)

Sono eventi che descrivono esplicitamente un’azione.

**Esempi:**
- `CMD_LIGHT_SET { target=LUCE_1, value=OFF }`
- `CMD_COVER_SET { target=TAPPARELLA_3, pos=60 }`

**Caratteristiche:**
- payload esplicito
- tipicamente un solo destinatario
- non usano binding “scene”
- spesso generati dal gateway MQTT / Home Assistant

**Affidabilità:**
- quasi sempre QoS1
- ACK atteso solo dal nodo target

---

## 6. QoS e affidabilità

### 6.1 QoS0 – Best effort
- Nessun ACK
- Nessun retry
- Usato per:
  - heartbeat
  - stato periodico
  - telemetria
  - debug

### 6.2 QoS1 – Consegna affidabile
- Richiede ACK da tutti i destinatari attesi
- Retry automatico
- Deduplica tramite sequence number

**Definizione di successo QoS1:**
> Il mittente ha ricevuto ACK da tutti i nodi attesi.

---

## 7. ACK multicast affidabile

### 7.1 ACK mirati
- Un nodo invia ACK **solo se**:
  - l’evento è rilevante per lui
  - QoS = 1

### 7.2 Slot temporali deterministici
Per evitare collisioni:
- gli ACK sono inviati in slot temporali deterministici
- lo slot dipende solo da informazioni locali e invarianti (es. `node_id`)

Questo garantisce:
- assenza di ACK storm
- bassa latenza
- assenza di coordinamento esplicito

### 7.3 Finestra ACK
- Ogni evento QoS1 apre una finestra temporale di ACK
- Durante questa finestra:
  - solo ACK ed eventi urgenti possono essere trasmessi
  - il traffico di fondo è sospeso

---

## 8. Sequence number e deduplica

Ogni nodo mantiene un contatore locale `seq_tx`.

L’identità di un messaggio è: (src_id, seq)


Serve per:
- deduplica (retry senza doppia esecuzione)
- correlazione ACK
- diagnostica

L’idempotenza applicativa è consigliata, ma **non sostituisce** la deduplica a livello trasporto.

---

## 9. Latenza e priorità

### 9.1 Obiettivi di latenza
- Eventi critici: **< 50 ms**
- Emergenze: precedenza assoluta, latenza non hard real-time

### 9.2 Classi di priorità
- EMERG
- CMD
- CFG
- BG (heartbeat, stato)

La priorità influenza:
- backoff
- accesso al mezzo
- possibilità di trasmettere durante finestre ACK

---

## 10. Multi-master e gestione collisioni

- Tutti i nodi possono trasmettere
- Accesso al mezzo tramite:
  - rilevamento bus idle
  - backoff deterministico pesato dalla priorità
- Collisioni rilevate indirettamente:
  - CRC errati
  - mancanza di ACK

---

## 11. Heartbeat e liveness

Ogni nodo invia periodicamente:
- heartbeat (QoS0)

Ogni nodo mantiene una tabella di liveness:
- `last_seen[node_id]`

Stati:
- **alive**: visto recentemente
- **suspect**: ACK mancati
- **dead**: heartbeat assente oltre soglia

Il recovery è automatico alla ricezione di nuovi messaggi.

---

## 12. Stato della rete e osservabilità

- I nodi pubblicano stato periodico (QoS0)
- Il gateway MQTT osserva il bus (sniffer)
- Lo stato è **eventually consistent**
- Non esiste una verità globale istantanea

Questo modello è sufficiente per:
- dashboard
- automazioni
- monitoraggio

---

## 13. Integrazione con Home Assistant

- Home Assistant invia **comandi separati per ogni entità**
- Il gateway MQTT traduce:
  - MQTT → eventi espliciti sul bus
- Il gateway è:
  - un nodo come gli altri
  - soggetto alle stesse regole di QoS, ACK, priorità

La rete deve funzionare **anche senza gateway**.

---

## 14. Assenza di consenso forte

La rete **non implementa**:
- RAFT
- Paxos
- consenso globale

Motivazioni:
- non serve consistenza forte
- i dati sono osservazioni, non decisioni
- la complessità supererebbe i benefici

Il sistema adotta **eventual consistency + affidabilità dei comandi**.

---

## 15. Principi guida finali

- Affidabilità dove serve (comandi)
- Best effort dove basta (stato)
- Eventi piccoli e frequenti
- Complessità spostata in fase di configurazione
- Nessuna assunzione di conoscenza globale perfetta
- Robustezza > perfezione teorica

---

## 16. Conclusione

Questa architettura fornisce una rete:
- stabile
- prevedibile
- estendibile
- compatibile con Home Assistant
- adatta a durare nel tempo

È un compromesso consapevole tra teoria dei sistemi distribuiti e pratica embedded reale.

