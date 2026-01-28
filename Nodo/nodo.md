# Nodo

## Introduzione

Il **Nodo** rappresenta l’unità fondamentale del sistema domotico distribuito. È un’entità autonoma collegata a un bus comune, progettata per operare come **agente reattivo programmabile**, privo di logica applicativa globale e consapevole esclusivamente delle proprie capacità locali.

Il Nodo costituisce il punto di incontro tra il mondo fisico (sensori e attuatori) e il sistema distribuito di eventi e stati, senza assumere alcun ruolo di coordinamento o decisione centrale.

---

## Ruolo del Nodo nel sistema

Il Nodo ha il compito di:

- Esporre un’identità univoca all’interno della rete
- Dichiarare le proprie capacità funzionali
- Generare eventi osservabili
- Reagire a eventi esterni secondo regole configurate
- Eseguire azioni locali su periferiche fisiche
- Pubblicare lo stato reale dei dispositivi che controlla

Il Nodo **non definisce la logica di automazione**: tale logica viene configurata dall’esterno durante una fase dedicata di programmazione.

---

## Principio fondamentale

> Il Nodo conosce **cosa può fare**, ma non **quando** né **perché** deve farlo.

Questo principio garantisce la separazione netta tra:

- **Capacità hardware e funzionali**, residenti nel Nodo
- **Logica causa–effetto**, definita a livello di sistema

---

## Modello concettuale del Nodo

Dal punto di vista architetturale, il Nodo può essere descritto come composto dai seguenti domini funzionali.

### Identità

Ogni Nodo possiede:

- Un **UID univoco** sulla rete
- Informazioni di versione (firmware, compatibilità)
- Uno stato operativo (boot, runtime, programmazione, errore)

L’UID è utilizzato esclusivamente per scoperta, configurazione e diagnostica, e non per l’indirizzamento diretto durante il funzionamento normale.

---

### Event Generators (cenno)

Gli **Event Generators** sono entità interne al Nodo in grado di rilevare eventi provenienti dal mondo fisico o logico locale e renderli osservabili sul bus.

Esempi tipici includono pulsanti, timer, sensori o condizioni di errore.

In questa sede, gli Event Generators vengono solo nominati: la loro specifica formale è trattata separatamente.

---

### Device / Actuators (cenno)

I **Device / Actuators** rappresentano le periferiche fisiche controllate dal Nodo, responsabili della modifica dello stato del mondo reale.

Ogni Device:

- Supporta un insieme finito di azioni
- Mantiene uno stato osservabile
- È responsabile della dichiarazione del proprio stato reale

Anche in questo caso, la descrizione dettagliata dei Device è demandata a una specifica dedicata.

---

### Reaction Engine

Il **Reaction Engine** è il componente logico centrale del Nodo. Il suo comportamento è puramente deterministico e consiste nel:

- Osservare gli eventi presenti sul bus
- Confrontarli con una tabella di reazioni configurata
- Eseguire una o più azioni locali quando una regola corrisponde
- Pubblicare lo stato aggiornato dei Device coinvolti

Il Reaction Engine **non prende decisioni autonome** e non introduce nuova logica: applica esclusivamente regole precedentemente caricate.

---

## Interfaccia del Nodo (contratto)

L’interfaccia del Nodo non è un’API procedurale, ma un **contratto comportamentale basato su messaggi**.

Il Nodo espone verso il sistema:

1. **Identificazione**
   - UID
   - Versione
   - Stato operativo

2. **Descrizione delle capacità**
   - Eventi generabili
   - Device disponibili
   - Azioni supportate

3. **Eventi runtime**
   - Eventi generati localmente

4. **Stati**
   - Stato corrente dei Device
   - Stati di errore o fault

5. **Canale di configurazione**
   - Ricezione delle regole evento → azione
   - Comandi di cambio modalità operativa

---

## Esclusioni esplicite

Per progetto, il Nodo **non** include:

- Semantica utente (nomi, etichette, contesto funzionale)
- Logica di automazione globale
- Coordinamento tra nodi
- Interfacce grafiche o di configurazione utente
- Integrazione diretta con sistemi esterni

Queste responsabilità sono intenzionalmente scorporate per mantenere il Nodo semplice, verificabile e riutilizzabile.

---

## Sintesi

Il Nodo è un elemento neutro, reattivo e programmabile, progettato per:

- Esporre capacità
- Reagire a eventi
- Modificare lo stato del mondo fisico
- Dichiarare in modo affidabile ciò che è realmente accaduto

Ogni altra forma di intelligenza o coordinamento è esterna al Nodo e viene applicata attraverso la configurazione del sistema.

