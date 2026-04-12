# 05 — Discovery operativa: da Node ID Matter a IPv6

## Obiettivo

Capire chi risolve davvero il mapping:

**Node ID Matter -> IPv6 del dispositivo**

---

## 1. Cosa NON fa Thread

Thread non risolve il Node ID Matter.

Thread:
- instrada pacchetti IP;
- fa routing nella mesh;
- ma non sa cosa sia il Node ID Matter come identità applicativa.

---

## 2. Cosa fa Matter

Matter usa una discovery operativa.

Il controller non dice semplicemente:
> “manda il pacchetto al Node ID 1”

Prima deve scoprire:
- quale nome operativo corrisponde a quel nodo;
- quale indirizzo IPv6 e quale porta usare.

Solo dopo può inviare il comando.

---

## 3. Sequenza logica

1. il controller conosce il Node ID Matter;
2. esegue la discovery operativa;
3. ottiene IPv6 + porta;
4. poi Thread/IP fa il routing fino al nodo.

---

## 4. Perché serve il Border Router nel modello standard

Nel modello standard Matter over Thread:
- i dispositivi Thread non usano direttamente la classica discovery IP come su Wi-Fi;
- il modello usa un sistema proxied;
- il Border Router gioca un ruolo chiave nel rendere disponibile questa discovery tra mondo Thread e controller/IP.

Per questo il BR non serve solo “per uscire su Internet”.
Serve anche per il percorso standard della discovery.

---

## 5. Errore tipico

Errore:
> “Se la rete è isolata e non mi serve Internet, allora non mi serve il Border Router”.

Corretto:
> Anche senza Internet, nel modello standard il BR è comunque molto utile per la parte di discovery e commissioning esterno.

---

## 6. Punto fondamentale

Node ID Matter e IPv6 sono due cose diverse:

- Node ID Matter = identità logica nella fabric
- IPv6 = indirizzo di rete

La discovery operativa collega queste due cose.
