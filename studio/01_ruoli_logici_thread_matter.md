# 01 — Ruoli logici in Thread e Matter

## Perché serve distinguere i ruoli

Il problema principale nello studio di Matter over Thread è che sulla stessa scheda fisica possono convivere più funzioni logiche.

Se non distingui bene i livelli, finisci per confondere:

- ruolo nella mesh Thread;
- ruolo nel commissioning Thread;
- ruolo nel livello Matter;
- ruolo di accesso verso reti esterne.

Per questo conviene dividere tutto in gruppi.

---

## 1. Ruoli della rete Thread

Questi ruoli descrivono **come il nodo vive nella mesh Thread**.

### Leader
È il nodo che, dentro una partizione Thread, mantiene certe informazioni di rete condivise e coordina aspetti interni della partizione.

Caratteristiche:
- ogni partizione Thread ha un solo Leader;
- il Leader è un ruolo della rete Thread;
- non è il commissioner;
- non è il border router per definizione;
- non è il controller Matter.

### Router
È un nodo Thread che partecipa attivamente al routing nella mesh.

Caratteristiche:
- inoltra traffico;
- può avere child sotto di sé;
- può diventare Leader se necessario.

### REED
Router-Eligible End Device.

È un nodo che normalmente può stare come end device, ma può essere promosso a router se la rete lo richiede.

### Child / End Device
Nodo che appartiene alla rete ma non fa routing come un router Thread.

Può essere:
- MED
- SED
- altri tipi di end device, a seconda del livello di dettaglio energetico.

---

## 2. Ruoli del commissioning Thread

Questi ruoli riguardano **come un nuovo nodo entra nella rete Thread**.

### Joiner
È il dispositivo nuovo che vuole entrare nella rete Thread.

Caratteristiche:
- inizialmente non è ancora membro della rete;
- conosce i dati necessari al commissioning;
- chiede di essere ammesso.

### Commissioner
È il soggetto che autorizza l'ingresso del Joiner nella rete Thread.

Caratteristiche:
- è temporaneo;
- nella rete Thread ce n'è uno solo attivo alla volta;
- decide chi entra;
- non coincide col Joiner.

### Active Commissioner
È il commissioner attualmente attivo in quel momento.

Importante:
- non è un ruolo strutturale come il Leader;
- serve solo quando stai facendo commissioning.

### Border Agent
È il servizio che permette a un commissioner esterno di collegarsi alla rete Thread.

Non commissiona direttamente il nodo.
Fa solo da porta d'accesso alla rete per il commissioner.

---

## 3. Ruoli di collegamento e livello IP

### Border Router
È il nodo che collega la rete Thread ad altre reti IP, come Ethernet o Wi-Fi.

Caratteristiche:
- fa da ponte tra Thread e altre reti IPv6;
- può ospitare un Border Agent;
- può partecipare alla discovery standard SRP/mDNS nel modello classico.

---

## 4. Ruoli Matter

### Matter Node / Accessory
È il dispositivo che, una volta dentro la fabric Matter, espone cluster, endpoint e attributi Matter.

### Matter Commissioner
È il soggetto che aggiunge il dispositivo alla fabric Matter.

Importante:
- non è la stessa cosa del Commissioner Thread;
- può stare sullo stesso hardware di altri ruoli, ma logicamente è distinto.

### Matter Controller
È il soggetto che, dopo il commissioning Matter, controlla il dispositivo.

---

## 5. Regola pratica: stessa scheda, più ruoli

Una scheda fisica può contenere più ruoli logici.

Esempi:

### Esempio A
Scheda = semplice dispositivo Matter over Thread
- ruolo Thread: Child o Router
- commissioning Thread: Joiner quando entra
- ruolo Matter: Matter Node

### Esempio B
Scheda = Border Router dedicato
- ruolo Thread: tipicamente Router
- ruolo IP: Border Router
- servizio: Border Agent
- opzionalmente: può ospitare logiche di commissioner esterno

### Esempio C
Scheda = dispositivo ibrido
- ruolo Thread: Router
- ruolo IP: Border Router
- ruolo Matter: Matter Node
- servizio: Border Agent

Questo è possibile se l'hardware ha sia 802.15.4 sia un backhaul IP.

---

## 6. Frase finale da ricordare

Non chiederti solo “che cos'è questa scheda?”.

Chiediti sempre:

- che ruolo ha nella mesh?
- che ruolo ha nel commissioning?
- che ruolo ha nel livello Matter/IP?
