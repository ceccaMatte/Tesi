# 04 — Border Router, Border Agent e Commissioner

## Obiettivo

Questa è la parte che crea più confusione. Qui separiamo bene tre cose diverse.

---

## 1. Border Router

Il Border Router è il nodo che collega la rete Thread ad altre reti IP come Wi-Fi o Ethernet.

Compiti principali:
- fare da ponte IPv6 tra Thread e altre reti IP;
- supportare discovery bidirezionale nel modello standard;
- permettere commissioning esterno;
- aiutare merge e connettività su infrastruttura IP.

Importante:
- in una rete Thread possono esserci più Border Router;
- non è un ruolo unico come il Leader.

---

## 2. Border Agent

Il Border Agent è un servizio esposto dal Border Router.

Compito:
- permettere a un commissioner esterno di scoprire la rete Thread e collegarsi ad essa per fare commissioning.

Il Border Agent:
- non è il commissioner;
- non decide lui chi entra;
- è il punto di accesso.

Pensa al Border Agent come a una porta.

---

## 3. Commissioner

Il Commissioner è il soggetto che autorizza l'ingresso di nuovi Joiner nella rete Thread.

Compiti:
- diventare Active Commissioner;
- autorizzare i Joiner;
- gestire la fase di commissioning Thread.

Quindi:
- Border Agent = porta
- Commissioner = amministratore temporaneo che usa quella porta

---

## 4. Errore tipico

Errore:
> “Il Border Agent è la funzione commissioner del Border Router”.

Corretto:
> No. Il Border Agent espone l'accesso. Il Commissioner usa quell'accesso.

---

## 5. Commissioner attivo

Nella rete Thread c'è un solo Active Commissioner alla volta.

Questo non significa che esista un solo Border Router.
Significa che, in quel momento, una sola entità sta svolgendo il ruolo di commissioner attivo.

---

## 6. Se il Border Router usato per il commissioning cade

Se il BR tramite cui il commissioner si è collegato va down:
- la sessione di commissioning può interrompersi;
- la rete Thread però non per forza collassa;
- se ci sono altri BR, il commissioner può ricollegarsi tramite un altro Border Agent.

Quindi la ridondanza dei BR aumenta la robustezza del sistema.
