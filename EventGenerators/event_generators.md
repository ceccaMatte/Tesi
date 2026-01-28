# Event Generators

## Introduzione

Gli **Event Generators** sono componenti di dominio interni a un Nodo il cui compito esclusivo è **rilevare il verificarsi di una condizione** e **renderla osservabile dal sistema sotto forma di evento**.

Essi rappresentano il punto di ingresso dell’informazione nel sistema distribuito e costituiscono l’origine di ogni reazione e comportamento osservabile.

---

## Scopo

Lo scopo di un Event Generator è trasformare il mondo fisico o logico locale in **eventi semantici**, permettendo al sistema di reagire a ciò che accade senza introdurre accoppiamento diretto tra sorgente e destinatari.

Un Event Generator:
- osserva una condizione
- quando la condizione si verifica, emette un evento

Non svolge alcun altro ruolo.

---

## Natura degli eventi

Gli eventi generati sono **atomici**, **descrittivi** e **privi di intenzione operativa**.

Un evento risponde esclusivamente alla domanda:

> *Che cosa è successo?*

Esempi di eventi:
- pressione o rilascio di un pulsante
- pressione prolungata
- scadenza di un timer
- superamento di una soglia
- rilevamento di un suono
- segnalazione di una condizione di errore

Un evento:
- non contiene azioni
- non contiene destinatari
- non implica un comportamento

---

## Caratteristiche fondamentali

Un Event Generator è caratterizzato dalle seguenti proprietà:

- **Passività**  
  Non agisce sul sistema, ma osserva e segnala.

- **Determinismo**  
  A parità di condizione osservata, genera sempre lo stesso evento.

- **Agnosticismo**  
  Non conosce chi utilizzerà l’evento né quali effetti produrrà.

- **Indipendenza dalla logica**  
  Non contiene regole causa–effetto.

- **Riutilizzabilità**  
  Lo stesso evento può essere associato a comportamenti diversi in contesti diversi.

---

## Ruolo nel sistema

Nel sistema complessivo, gli Event Generators:

- rappresentano l’**origine degli eventi**
- disaccoppiano il mondo fisico dalla logica di automazione
- abilitano un modello **event-driven distribuito**
- consentono la composizione dinamica dei comportamenti

Essi forniscono informazione, non intelligenza.

---

## Relazione con il Nodo

- Gli Event Generators **appartengono al Nodo**
- Il Nodo espone verso il sistema gli eventi che essi generano
- Il Nodo decide se e come reagire agli eventi, tramite regole configurate

L’Event Generator non ha visibilità né controllo sul comportamento del Nodo.

---

## Relazione con Device / Actuators

- Gli Event Generators **non comandano i Device**
- Non eseguono azioni
- Non modificano direttamente lo stato fisico

I Device reagiscono esclusivamente alle azioni invocate dal Nodo a seguito di eventi.

---

## Esclusioni esplicite

Per progetto, un Event Generator **non**:

- esegue azioni
- prende decisioni
- conosce la logica del sistema
- conosce altri nodi
- interpreta il significato degli eventi

---

## Sintesi

Un Event Generator è una sorgente di verità locale che rende osservabile un accadimento, senza interpretarlo né utilizzarlo.

La sua semplicità è una proprietà essenziale per garantire disaccoppiamento, scalabilità e chiarezza architetturale del sistema.

