# Gateway — Scopo, ruolo, funzionamento generale e confini

## Scopo
Il Gateway è il componente della rete domotica che ha il compito di collegare il sistema interno della casa con i sistemi esterni e di rendere osservabile il comportamento complessivo della rete.  
Il suo scopo principale è raccogliere eventi e stati provenienti dai Nodi, esporli verso l’esterno e supervisionare che il sistema distribuito stia evolvendo in modo coerente con la configurazione definita.

## Ruolo nel sistema
Il Gateway non è il componente che decide la logica della casa.  
Le relazioni evento → azione restano distribuite nei Nodi e vengono definite in fase di programmazione, mentre il Gateway mantiene una rappresentazione della configurazione sufficiente a interpretare correttamente ciò che osserva sulla rete.

Nel sistema il Gateway svolge quindi un doppio ruolo:

- punto di interconnessione tra rete domotica interna e sistemi esterni, ad esempio tramite MQTT verso Home Assistant
- supervisore del comportamento osservabile della rete, senza sostituirsi alla logica distribuita dei Nodi

## Funzionamento generale
Durante il funzionamento normale, i Nodi generano eventi, reagiscono localmente secondo la propria configurazione e pubblicano gli stati risultanti dei Device. Il Gateway osserva questi eventi e questi stati, costruisce una vista coerente dello stato della casa e la rende disponibile verso l’esterno.

Per poter svolgere questa funzione in modo affidabile, il Gateway deve conoscere:

- la configurazione distribuita della logica runtime
- la mappatura tra eventi, effetti attesi e Device coinvolti
- l’associazione tra Device logici e Nodi fisici che li controllano

In questo modo il Gateway non esegue la logica, ma è in grado di verificare se il comportamento osservato della rete è coerente con quello atteso.

## Confini
Il Gateway **fa**:

- osservazione degli eventi e degli stati pubblicati sulla rete
- mantenimento di una vista coerente dello stato complessivo del sistema
- esposizione dello stato e degli eventi verso sistemi esterni
- supervisione del comportamento osservabile della rete
- utilizzo della configurazione distribuita per interpretare correttamente ciò che avviene

Il Gateway **non fa**:

- non contiene la logica applicativa degli scenari
- non decide localmente come un Nodo deve reagire a un evento
- non controlla direttamente gli attuatori del mondo fisico
- non sostituisce i Nodi come fonte di verità sullo stato reale dei Device
- non richiede necessariamente che la funzione di programmazione sia fisicamente separata, ma tale funzione resta logicamente distinta dal runtime