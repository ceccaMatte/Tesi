# 02 — Commissioning Thread

## Obiettivo

Capire come un nuovo nodo entra nella rete Thread.

Questa fase è **prima** del commissioning Matter.

Un nodo può entrare nella rete Thread e non essere ancora nella fabric Matter.

---

## 1. Attori coinvolti

### Joiner
È il nodo nuovo che vuole entrare nella rete Thread.

### Commissioner
È chi autorizza il Joiner ad entrare.

### Border Agent
È il servizio che permette a un commissioner esterno di collegarsi alla rete Thread.

### Border Router
È il nodo che ospita il Border Agent e collega Thread a una rete IP esterna.

---

## 2. Cosa NON è automatico

Un Joiner che si accende **non entra automaticamente** nella rete Thread.

Per entrare servono:
- un commissioner attivo o attivabile;
- dati di commissioning corretti;
- una procedura di ammissione.

Quindi il commissioning Thread è spesso:
- manuale;
- semi-automatico;
- orchestrato da un'app, hub o controller.

---

## 3. Sequenza concreta

## Passo 1 — la rete Thread esiste già
Hai una rete Thread già formata.

Può avere:
- un Leader;
- uno o più Router;
- uno o più Border Router.

## Passo 2 — si accende il nuovo nodo
Il nuovo nodo si accende.

In questo momento:
- non è ancora membro della rete;
- è solo un potenziale Joiner.

## Passo 3 — il commissioner deve esistere
Se non esiste già un Active Commissioner, qualcuno deve attivarlo.

Questa parte spesso richiede una azione umana:
- aprire un'app;
- premere un pulsante;
- avviare una procedura da hub o gateway.

## Passo 4 — il commissioner si collega tramite Border Agent
Se il commissioner è esterno alla mesh Thread, si collega alla rete passando dal Border Agent.

## Passo 5 — petition
Il commissioner chiede alla rete Thread di diventare l'Active Commissioner.

Importante:
- qui **non** sta entrando un nodo nuovo;
- qui sta entrando in funzione l'amministratore temporaneo che farà entrare il Joiner.

## Passo 6 — autorizzazione del Joiner
Il commissioner configura la rete per ammettere quel Joiner specifico.

Tipicamente questo richiede un dato condiviso come il PSKd.

## Passo 7 — il Joiner esegue la procedura Joiner
Il dispositivo nuovo esegue la procedura di ingresso.

## Passo 8 — ingresso nella rete Thread
Se tutto va bene:
- entra nella mesh;
- riceve connettività Thread;
- ottiene indirizzi IPv6 Thread;
- diventa un nodo della rete.

---

## 4. Cosa ottieni alla fine

Alla fine del commissioning Thread, il nodo:

- è nella rete Thread;
- può comunicare a livello rete;
- ma non è ancora necessariamente un nodo Matter operativo nella fabric.

---

## 5. Errore tipico da evitare

Errore:
> “Il nodo nuovo entra direttamente in Matter”.

Corretto:
1. prima entra in **Thread**;
2. poi viene aggiunto a **Matter**.
