# 06 — Più Border Router: split, merge, aggiunta e rimozione

## Obiettivo

Capire come funziona concretamente una rete con più Border Router ridondanti.

---

## 1. Stato normale

Supponi di avere:
- BR1
- BR2
- BR3

tutti con le stesse credenziali Thread.

In condizioni normali:
- appartengono alla stessa rete Thread;
- partecipano alla robustezza della rete;
- possono offrire più punti di accesso per commissioning e funzioni BR.

Questo è previsto dal modello standard.

---

## 2. Split della rete

Supponi che:
- BR1 finisca nella partizione A;
- BR2 finisca nella partizione B;
- BR3 magari resti con A o vada offline.

Se la connettività radio si perde, Thread crea due partizioni.

Ogni partizione avrà:
- il proprio Leader;
- i propri nodi;
- il proprio stato di rete Thread.

Ciascun Border Router continua a lavorare nella sua partizione.

Importante:
- non si elegge un solo BR globale;
- il ruolo unico è il Leader della partizione, non il Border Router.

---

## 3. Merge

Quando le partizioni tornano a vedersi:
- via radio;
- o tramite infrastruttura IP, se supportata;

Thread fonde automaticamente le partizioni.

Dopo il merge:
- torna a esserci una sola partizione;
- torna a esserci un solo Leader;
- i Border Router presenti operano di nuovo nello stesso contesto di rete.

---

## 4. Aggiunta di un nuovo Border Router

Se aggiungi BR4 con:
- stesso set di credenziali Thread;
- configurazione compatibile;

allora BR4 può unirsi alla stessa rete e aumentare la ridondanza.

---

## 5. Rimozione o guasto di un Border Router

Se un BR si spegne:
- la rete Thread non necessariamente collassa;
- se ne restano altri, la rete continua a funzionare;
- perdi ridondanza, ma non per forza il servizio.

---

## 6. Vantaggio della ridondanza

Il punto forte di più Border Router non è avere una rete “accademicamente perfettamente distribuita”.

Il punto forte è:
- riusare il modello standard;
- aumentare la robustezza;
- ridurre i single point of failure pratici;
- non inventare un protocollo custom.

Per un sistema reale, questa è spesso la scelta più sensata.
