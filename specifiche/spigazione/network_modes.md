# Modalità di Funzionamento del Bus di Comunicazione

Questo documento descrive le **modalità operative** e i **macro-comportamenti** supportati dal bus di comunicazione della rete domotica embedded.

Le modalità non rappresentano implementazioni differenti del protocollo, ma **diversi regimi di utilizzo dello stesso bus**, alcuni mutuamente esclusivi nel tempo, altri attivabili come overlay a runtime.

---

## Indice delle modalità

1. Programmazione della rete  
2. Runtime event-driven (base comune)  
   2.1 Runtime con sottosistema di allarme  
   2.2 Runtime senza gateway  
   2.3 Modalità Debug  
   2.4 Runtime con gateway  

---

## 1. Programmazione della rete

### Caratteristiche principali

- Modalità **temporanea** e **non operativa**
- Architettura **master–slave**
- Un solo master di programmazione
- Eventi runtime sospesi
- Comunicazione deterministica e sequenziale
- Distribuzione delle mappe evento → azione
- Non fa parte del funzionamento a regime

### Descrizione

La fase di **programmazione della rete** è il momento in cui il sistema viene configurato dal punto di vista logico.  
In questa fase non avviene automazione, ma **definizione delle regole** che governeranno il comportamento futuro della rete.

Durante la programmazione:
- la rete **non è event-driven**
- viene adottato un modello **centralizzato**
- esiste un **master di programmazione**
- tutti gli altri nodi operano come slave passivi

Il master:
- scopre i nodi presenti
- interroga le loro capacità
- costruisce la mappa globale degli eventi
- distribuisce a ogni nodo le regole locali evento → azione

Ogni nodo riceve:
- la propria configurazione operativa
- una conoscenza globale *passiva* degli eventi presenti nella rete

Questa conoscenza globale consente, a runtime:
- migliore rispetto delle finestre temporali
- riduzione delle collisioni
- comportamento più prevedibile

Al termine della programmazione:
- il master scompare
- la rete torna distribuita
- le regole entrano in vigore

---

## 2. Runtime Event-Driven (base comune)

### Caratteristiche principali

- Architettura **distribuita**
- Rete **multi-master**
- Nessun nodo privilegiato
- Comunicazione **event-driven**
- Tutti ascoltano tutto
- Ogni nodo reagisce solo se coinvolto
- Affidabilità applicata solo dove necessario

### Descrizione

Il runtime event-driven rappresenta il **comportamento di base** della rete durante il funzionamento normale.

In questo regime:
- qualsiasi nodo può trasmettere
- non esiste un coordinatore centrale
- non esiste scheduling globale
- non esiste consenso distribuito

Il bus trasporta eventi e comandi, e ogni nodo:
- riceve tutti i messaggi
- decide localmente se sono rilevanti
- reagisce solo agli eventi per cui è stato configurato

Tutte le modalità operative a regime **derivano da questo modello** e non lo modificano.

---

## 2.1 Runtime con sottosistema di allarme

### Caratteristiche principali

- Estensione funzionale del runtime
- Cooperazione locale tra nodi
- Condivisione dati non critici
- Messaggi QoS0 a bassa priorità
- Nessun coordinatore
- Decisione di allarme locale

### Descrizione

Il sottosistema di allarme introduce una forma di cooperazione locale tra nodi (ad esempio sensori acustici su finestre), mantenendo invariata l’architettura della rete.

Oltre ai messaggi standard:
- eventi
- ACK
- heartbeat

circolano anche:
- dati di rumore
- feature audio
- informazioni statistiche locali

Questi messaggi:
- non sono affidabili
- non sono critici
- non bloccano il bus
- non influenzano la logica generale

La decisione finale di allarme avviene **localmente** o a livello di gruppo ristretto.  
Solo l’esito (`ALARM_DETECTED`) viene promosso a evento ad alta priorità.

---

## 2.2 Runtime senza gateway

### Caratteristiche principali

- Nessun gateway MQTT
- Nessuna pubblicazione periodica di stato
- Traffico ridotto al minimo
- Funzionamento completamente offline
- Massima autonomia della rete

### Descrizione

In questa configurazione la rete opera **senza alcun nodo osservatore esterno**.

Il traffico sul bus è limitato a:
- eventi generati dai nodi
- ACK per i messaggi affidabili
- heartbeat per la liveness

Non vengono inviati:
- stati periodici
- telemetria continua
- informazioni di monitoraggio

Questa modalità è pensata per:
- ridurre il rumore sul bus
- abbassare la probabilità di collisioni
- garantire massima reattività
- consentire funzionamento senza infrastrutture esterne

---

## 2.3 Modalità Debug

### Caratteristiche principali

- Modalità **aggiuntiva**
- Attivabile a runtime
- Non modifica l’architettura
- Non introduce un master
- Non sospende il funzionamento normale
- Aumenta l’osservabilità
- Persistente con timeout di sicurezza

### Descrizione

La modalità Debug è un **overlay osservativo** che può essere abilitato sia su singoli nodi sia sull’intera rete.

Il Debug:
- non cambia cosa fa un nodo
- non altera la logica evento → azione
- non è necessario per inviare comandi diretti

Il suo effetto principale è rendere i nodi **più esplicativi** nelle risposte, ad esempio:
- ACK più dettagliati
- indicazione dell’evento che ha causato un’azione
- identificazione del nodo sorgente
- informazioni diagnostiche aggiuntive

Il Debug può essere:
- disabilitato
- abilitato permanentemente
- abilitato temporaneamente per un intervallo configurabile

### Persistenza e avvio

- Se il Debug era attivo prima del reset:
  - il nodo parte in Debug temporizzato
- Se non era attivo:
  - il nodo parte in modalità normale
- Allo scadere del timeout:
  - il nodo torna automaticamente a regime

Questa scelta garantisce osservabilità all’avvio senza rischio di Debug dimenticato attivo.

La modalità Debug è volutamente **aperta a estensioni future**.

---

## 2.4 Runtime con gateway

### Caratteristiche principali

- Presenza di un gateway MQTT
- Gateway trattato come nodo normale
- Osservabilità verso l’esterno
- Comandi espliciti in ingresso
- Nessuna centralizzazione della logica

### Descrizione

In questa configurazione la rete è affiancata da un gateway che:
- ascolta il traffico sul bus
- pubblica lo stato verso sistemi esterni
- riceve comandi da Home Assistant
- inietta comandi espliciti nella rete

Il gateway:
- ha un proprio node_id
- segue le stesse regole di QoS e priorità
- non coordina gli altri nodi
- non possiede logica globale

In questa modalità convivono:
- eventi descrittivi, interpretati localmente dai nodi
- comandi dichiarativi, che esprimono direttamente lo stato desiderato degli attuatori

Le scene esterne vengono tradotte in:
- sequenze di comandi indipendenti
- senza atomicità globale

La rete rimane **autonoma e funzionante** anche in assenza del gateway.

---

## Sintesi finale

Il bus supporta:

- una modalità di **definizione** (programmazione)
- un runtime **distribuito e event-driven**
- estensioni **locali e osservative**
- integrazione **non invasiva** con sistemi esterni

Il mezzo fisico e il protocollo restano invariati.  
Cambia **come vengono usati**, non **cosa sono**.
