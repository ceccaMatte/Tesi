# 07 — Schema rapido e domande tipiche

## Schema mentale rapidissimo

### Livello 1 — Rete Thread
- Leader
- Router
- REED
- Child

### Livello 2 — Commissioning Thread
- Joiner
- Commissioner
- Active Commissioner
- Border Agent

### Livello 3 — Collegamento IP / Matter
- Border Router
- Matter Commissioner
- Matter Controller
- Matter Node

---

## Domande tipiche

### “Il Border Agent è il Commissioner?”
No.
Il Border Agent è il punto di accesso.
Il Commissioner è chi usa quel punto di accesso.

### “Il Joiner è il Commissioner?”
No.
Il Joiner è il nodo nuovo che vuole entrare.
Il Commissioner è chi lo fa entrare.

### “Il Leader Thread decide chi entra?”
No.
Il Leader è un ruolo della rete Thread.
Il commissioning è gestito dal Commissioner.

### “Se ho più Border Router, chi è quello giusto?”
Non ce n'è uno unico “capo”.
Più BR possono coesistere.
La rete Thread sceglie il proprio Leader, non il proprio unico BR.

### “Il Commissioner resta sempre attivo?”
Non necessariamente.
È spesso un ruolo temporaneo, usato quando serve fare commissioning.

### “Entrare in Thread significa essere già in Matter?”
No.
Prima commissioning Thread.
Poi commissioning Matter.

---

## Frase finale da ricordare

**Joiner = vuole entrare**
**Commissioner = lo fa entrare**
**Border Agent = fa entrare il Commissioner**
**Border Router = collega Thread al resto della rete IP**
**Leader = coordina la partizione Thread**
