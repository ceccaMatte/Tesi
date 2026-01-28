# Tipi di Messaggi sulla Rete

Questo documento descrive i **tipi di messaggi** che possono circolare sulla rete e il loro ruolo nel funzionamento del sistema.

I messaggi sono classificati in base a:
- **semantica**
- **destinatario logico**
- **scopo funzionale**

La classificazione è indipendente dalla modalità operativa del bus  
(programmazione, runtime, debug, gateway).

---

## Indice dei tipi di messaggi

1. Eventi descrittivi  
2. Eventi dichiarativi / Comandi espliciti verso attuatori  
3. Comandi diretti al Nodo  
4. Messaggi di risposta e ACK  

---

## 1. Eventi descrittivi

### Caratteristiche principali

- Descrivono **ciò che è accaduto**
- Non indicano azioni da eseguire
- Non contengono logica
- Payload minimo o assente
- Destinatari **impliciti**
- Semantica stabile
- Utilizzati per automazioni locali

### Descrizione

Gli **eventi descrittivi** rappresentano fatti osservati nel sistema, come:

- pressione di un pulsante
- rilevamento di presenza
- superamento di una soglia
- evento di allarme

L’evento **non specifica cosa deve accadere** in risposta.  
Ogni nodo decide localmente se e come reagire, sulla base della propria configurazione evento → azione.

L’evento descrittivo è il fondamento del modello **event-driven** della rete.

---

### Chi può inviarli

- Qualsiasi nodo che rilevi un evento fisico o logico
- Sensori
- Attuatori con feedback
- Sottosistemi locali (es. antifurto)

Non esiste alcuna autorità centrale.

---

### Come risponde la rete

- I nodi interessati eseguono l’azione associata
- I nodi non interessati ignorano il messaggio
- Se l’evento è QoS1:
  - i nodi coinvolti inviano ACK
- In modalità Debug:
  - gli ACK possono includere informazioni diagnostiche

---

## 2. Eventi dichiarativi / Comandi espliciti verso attuatori

### Caratteristiche principali

- Esprimono uno **stato desiderato**
- Focalizzati sull’attuatore, non sul nodo
- Semantica esplicita
- Possono includere payload
- Destinatari determinabili
- Non richiedono logica evento → azione

### Descrizione

Gli eventi dichiarativi (o comandi espliciti) descrivono direttamente **cosa deve essere fatto**, ad esempio:

- accendere una luce
- impostare una posizione
- cambiare uno stato operativo

Il messaggio non rappresenta un evento astratto, ma un’istruzione chiara.

In questo modello:
- l’attuatore è il vero target logico
- il nodo che lo controlla reagisce perché **si riconosce destinatario**

---

### Chi può inviarli

- Gateway MQTT
- Home Assistant
- Qualsiasi nodo autorizzato sul bus
- Strumenti di manutenzione o test

Non esistono nodi privilegiati a runtime.

---

### Come risponde il nodo

- Il nodo che controlla l’attuatore:
  - applica il comando
  - invia ACK se richiesto dal QoS
- Gli altri nodi:
  - ignorano il messaggio

In modalità Debug:
- l’ACK può specificare:
  - l’azione eseguita
  - l’attuatore coinvolto
  - l’esito dell’operazione

---

## 3. Comandi diretti al Nodo

### Caratteristiche principali

- Indirizzati a un **node_id specifico**
- Riguardano il nodo come entità
- Indipendenti dal Debug
- Sempre disponibili a runtime
- Non attivano automazioni
- Non coinvolgono altri nodi

### Descrizione

I comandi diretti al nodo consentono di:
- interrogare lo stato interno
- modificare parametri locali
- eseguire auto-diagnostica
- effettuare test e manutenzione

Il focus del messaggio **non è l’attuatore**, ma il nodo stesso.

Esempi:
- richiesta consumi
- lettura parametri MAC
- modifica timeout o finestre
- avvio self-test
- configurazioni runtime

---

### Chi può inviarli

- Qualsiasi nodo sul bus
- Gateway
- Strumenti di configurazione
- Nodi di test o manutenzione

Il sistema non prevede privilegi a runtime.

---

### Come risponde il nodo

- Il nodo destinatario:
  - elabora il comando
  - invia una risposta diretta
- La risposta:
  - può fungere da ACK
  - può contenere dati strutturati
  - può indicare errori o fallimenti

Il fallimento:
- può essere silenzioso
- o esplicito
- dipende dalla configurazione e dal carico del bus

Alcuni comandi:
- producono effetti immediati
- altri richiedono reboot esplicito per applicare le modifiche

---

## 4. Messaggi di risposta e ACK

### Caratteristiche principali

- Utilizzati per affidabilità
- Inviati solo dai nodi coinvolti
- Programmati in finestre temporali
- Deterministici
- Verbosità variabile

### Descrizione

I messaggi di risposta e ACK servono a:
- confermare la ricezione di un messaggio
- garantire la consegna (QoS1)
- fornire informazioni aggiuntive

In condizioni normali:
- l’ACK è minimale

In modalità Debug:
- l’ACK può includere:
  - contesto
  - causa dell’azione
  - identificazione dell’evento scatenante
  - informazioni diagnostiche

---

### Chi li invia

- Solo i nodi per cui il messaggio è rilevante
- Mai i nodi non coinvolti

---

### Ruolo nel sistema

Gli ACK:
- non coordinano la rete
- non introducono consenso
- non forniscono stato globale

Servono esclusivamente a:
- garantire affidabilità dove necessario
- rendere il comportamento osservabile

---

## Sintesi

La rete supporta tre classi principali di messaggi funzionali:

- **eventi descrittivi**  
  → cosa è accaduto

- **eventi dichiarativi / comandi espliciti**  
  → cosa deve accadere

- **comandi diretti al nodo**  
  → cosa deve fare un nodo specifico

A queste si affiancano:
- messaggi di risposta
- ACK
- messaggi diagnostici

Questa separazione consente:
- semplicità locale
- flessibilità globale
- osservabilità
- evoluzione futura del protocollo
