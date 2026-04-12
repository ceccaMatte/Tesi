# 03 — Commissioning Matter

## Obiettivo

Capire cosa succede **dopo** che il nodo è già entrato nella rete Thread.

Questa fase riguarda la **fabric Matter**, non la mesh Thread.

---

## 1. Distinzione chiave

### Commissioning Thread
Serve a far entrare il dispositivo nella rete Thread.

### Commissioning Matter
Serve a far entrare il dispositivo nella fabric Matter.

Quindi:

- Thread = accesso alla rete
- Matter = accesso al sistema applicativo e di sicurezza Matter

---

## 2. Cosa ottiene il nodo durante il commissioning Matter

Quando viene aggiunto a Matter, il nodo ottiene:

- appartenenza a una fabric;
- Node ID Matter;
- operational credentials;
- possibilità di usare CASE e la normale comunicazione Matter.

---

## 3. Ruoli coinvolti

### Matter Commissioner
È il soggetto che aggiunge il dispositivo alla fabric Matter.

### Matter Controller
È il soggetto che poi lo controllerà.

Molto spesso nella pratica lo stesso ecosistema o software fa entrambe le cose.

---

## 4. Sequenza semplificata

1. il nodo è già nella rete Thread;
2. viene individuato come dispositivo commissionabile Matter;
3. il Matter Commissioner lo autentica;
4. gli assegna credenziali e Node ID;
5. il nodo entra nella fabric;
6. da quel momento diventa un vero nodo Matter operativo.

---

## 5. Errore tipico da evitare

Errore:
> “Se il nodo è nella rete Thread, allora è già pronto come nodo Matter”.

Corretto:
> No. Essere nella rete Thread non basta. Deve ancora entrare nella fabric Matter.
