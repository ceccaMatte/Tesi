# Device / Actuators

## Introduzione

I **Device / Actuators** sono componenti di dominio interni a un Nodo responsabili della **modifica dello stato del mondo fisico**. Rappresentano l’unico punto del sistema in cui avviene un’azione concreta e osservabile sull’ambiente.

Un Device non prende decisioni, non genera eventi di comando e non contiene logica di automazione. Esegue esclusivamente azioni richieste dal Nodo e dichiara lo stato reale risultante.

---

## Scopo

Lo scopo di un Device è:

- eseguire azioni fisiche locali
- mantenere uno stato coerente e osservabile
- rendere il sistema informato sull’esito reale delle azioni eseguite

Il Device costituisce la **fonte di verità sullo stato fisico**, non sulla logica che ha portato a quello stato.

---

## Natura delle azioni

Un Device espone un insieme **finito e ben definito di azioni**, che rappresentano ciò che è fisicamente in grado di fare.

Esempi:
- luce: ON, OFF, ON\_TEMP
- tapparella: UP, DOWN, STOP, MOVE\_TO
- carico di potenza: ON, OFF
- dimmer: SET\_LEVEL

Le azioni:
- sono locali
- non vengono mai trasmesse sul bus
- non sono eventi

---

## Stato del Device

Ogni Device mantiene uno **stato osservabile**, che rappresenta la condizione reale del dispositivo fisico.

Lo stato:
- viene aggiornato a seguito di un’azione
- riflette il risultato effettivo (successo, errore, indisponibilità)
- viene pubblicato verso il sistema

Il sistema non deduce lo stato dalle azioni: lo **riceve esplicitamente dal Device**.

---

## Caratteristiche fondamentali

Un Device / Actuator è caratterizzato dalle seguenti proprietà:

- **Reattività**  
  Esegue azioni solo quando invocato dal Nodo.

- **Determinismo**  
  A parità di azione e condizioni fisiche, produce lo stesso stato.

- **Responsabilità dello stato**  
  È l’unica fonte attendibile sul proprio stato reale.

- **Isolamento logico**  
  Non conosce eventi, regole o altri nodi.

- **Limitatezza funzionale**  
  Può fare solo ciò che il suo hardware consente.

---

## Ruolo nel sistema

Nel sistema complessivo, i Device:

- rappresentano il **livello di attuazione fisica**
- separano il controllo logico dall’hardware
- garantiscono osservabilità tramite la pubblicazione dello stato
- consentono il disaccoppiamento tra evento e azione

I Device non partecipano alla logica distribuita: ne sono l’effetto finale.

---

## Relazione con il Nodo

- I Device **appartengono al Nodo**
- Il Nodo invoca le azioni sui Device
- Il Nodo espone verso il sistema lo stato dei Device

Il Device non osserva il bus e non interagisce direttamente con altri componenti.

---

## Relazione con Event Generators

- I Device **non generano eventi di comando**
- Non reagiscono direttamente agli Event Generators
- Ogni relazione evento → azione è mediata dal Nodo

Questo garantisce la separazione netta tra osservazione e attuazione.

---

## Esclusioni esplicite

Per progetto, un Device / Actuator **non**:

- prende decisioni
- conosce la logica di automazione
- interpreta eventi
- invia comandi sulla rete
- conosce altri Device o Nodi

---

## Sintesi

Un Device / Actuator è un esecutore fisico deterministico che:

- riceve richieste di azione dal Nodo
- modifica il mondo reale
- dichiara lo stato risultante

La sua semplicità è essenziale per mantenere il sistema osservabile, prevedibile e scalabile.

