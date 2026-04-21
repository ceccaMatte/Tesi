**File system manager**
- lo scopo sarà quello di semplificare al minimo utilizzo del file sistem quindi tutto quello che riguarda lettura e scirttura di file persistenti. 
  - Comandi a cui rispodere
  - paramaetri imposazioni utenti
  - password e reti wifi

**wifi manager**
- lo scopo è quello di gestire in maniera semplice la connettività. il suo scopo sarà quello di garantire che l'esp sia sempre connessa alla rete wifi quidni gestione della recconect. 
- Deve esporre due modalità di connessione 
  - **SingleNetwork** ovvero si connette solo alla rete data dall'utente, utile nel caso in cui i vioglia connettermi solo ad una rete specifica per esempio fare da webserver e servire pagine locali
  - **ConnectionFirst** in queste modalità lo scopo è essere online quinid può connettersi a qualsiasi rete indicata dall'utente perchè lo scopo è quello di restare connessi.