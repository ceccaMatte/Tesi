# Requisiti della piattaforma firmware

## 1. Scopo del documento
Questo documento definisce i requisiti, le linee guida e i principi architetturali comuni alla piattaforma firmware delle schede del sistema domotico.

Lo scopo del documento è fornire una base tecnica condivisa per la realizzazione delle diverse schede del progetto, in modo da garantire:

- robustezza
- prevedibilità
- manutenibilità
- osservabilità
- riuso delle soluzioni comuni
- coerenza architetturale tra i diversi firmware

Questo documento non definisce ancora nel dettaglio il comportamento applicativo delle singole schede, né la distribuzione finale dei ruoli logici sui componenti fisici.

---

## 2. Obiettivi della piattaforma firmware
La piattaforma firmware deve costituire una base comune solida per tutte le schede del sistema.

In particolare deve perseguire i seguenti obiettivi:

- robustezza nel funzionamento continuo
- capacità di recovery automatica in caso di fault
- comportamento deterministico e verificabile
- gestione chiara dello stato interno
- facilità di debug e diagnostica
- supporto alla persistenza dei dati rilevanti
- aggiornabilità nel tempo
- riutilizzabilità dei moduli software tra schede diverse

La piattaforma deve essere pensata per un contesto embedded distribuito, con requisiti di affidabilità superiori a quelli di un software puramente sperimentale o prototipale.

---

## 3. Principi generali di progettazione firmware

### 3.1 Chiarezza delle responsabilità
Ogni modulo firmware deve avere responsabilità chiare e ben delimitate.  
Va evitata la concentrazione di logiche eterogenee nello stesso componente.

### 3.2 Separazione dei livelli
Devono essere tenuti distinti, per quanto possibile:

- accesso hardware
- gestione dello stato
- logica di controllo
- comunicazione
- persistenza
- diagnostica

### 3.3 Determinismo e prevedibilità
Il comportamento del firmware deve essere il più possibile deterministico, osservabile e comprensibile.  
Le transizioni importanti dello stato della scheda non devono dipendere da logiche implicite o difficili da tracciare.

### 3.4 Riutilizzo e modularità
Le funzionalità comuni tra schede devono essere implementate in modo modulare e riutilizzabile, evitando duplicazioni non necessarie.

### 3.5 Gestione esplicita degli errori
Gli errori non devono essere ignorati o assorbiti silenziosamente.  
Quando possibile, devono essere:

- rilevati
- confinati
- resi osservabili
- recuperati automaticamente oppure segnalati in modo chiaro

### 3.6 Stato come responsabilità esplicita
Lo stato della scheda deve essere gestito in modo centralizzato e coerente, evitando copie locali ridondanti e fonti multiple di verità.

### 3.7 Uso appropriato delle FSM
L’uso di macchine a stati finiti è consigliato per modellare comportamenti con stati operativi distinti, transizioni esplicite, timeout, recovery o modalità degradate.

Non è invece necessario forzare l’uso di FSM per sequenze puramente lineari, deterministiche e procedurali.

### 3.8 Inizializzazione ordinata
La sequenza di boot deve essere ordinata, esplicita e verificabile.  
Qualora il boot includa modalità distinte, ad esempio:

- avvio normale
- fast recovery
- safe mode
- validazione OTA
- rollback

è preferibile modellare tali comportamenti tramite una FSM o una gerarchia limitata di FSM.

---

## 4. Gestione della memoria

### 4.1 Principio generale
L’uso di memoria dinamica deve essere evitato il più possibile nel runtime ordinario delle schede.

### 4.2 Uso ammesso solo in casi controllati
La memoria dinamica può essere ammessa solo in casi eccezionali e ben motivati, a condizione che siano chiari e verificabili:

- ownership
- ciclo di vita delle allocazioni
- assenza di memory leak
- comportamento in caso di saturazione
- impatto sul sistema nel lungo periodo

### 4.3 Preferenza per strutture preallocate
Dove possibile devono essere preferiti:

- buffer preallocati
- strutture dati statiche
- code a capacità nota
- gestione esplicita della saturazione

### 4.4 Saturazione controllata
In caso di esaurimento delle risorse non si devono produrre comportamenti casuali o non osservabili.  
La saturazione deve essere gestita in modo prevedibile, ad esempio con:

- rifiuto di nuove richieste
- scarto controllato
- segnalazione diagnostica
- fallback esplicito

---

## 5. Robustezza runtime

### 5.1 Watchdog
Ogni scheda deve adottare meccanismi di watchdog adatti a rilevare blocchi o comportamenti anomali del sistema.

La strategia concreta può includere il controllo di:

- task
- loop principali
- sezioni critiche
- condizioni di stallo significative

### 5.2 Recovery automatica
In presenza di fault recoverable, la piattaforma deve favorire il ripristino automatico del funzionamento senza richiedere intervento manuale.

### 5.3 Graceful degradation
Quando un sottosistema fallisce, il resto della scheda deve rimanere operativo per quanto possibile, almeno per consentire:

- recovery
- diagnostica
- segnalazione dello stato di errore

esempio se una scheda controlla `2 luci 3 spine e 4 pulsanti` e si bruciano s pine i pulsanti e le luci devono continuare a funzioanre e semplicemnte deve essere segnalato l'errore delle due spine

### 5.4 Osservabilità dei fault
I fault significativi devono essere rilevabili e riportabili all’esterno, o almeno registrabili localmente in modo utile alla manutenzione.

---

## 6. Reboot e recovery rapida

### 6.1 Fast reboot
La piattaforma deve supportare una modalità di reboot rapido orientata al recupero veloce della scheda a seguito di reset, watchdog o fault runtime.

### 6.2 Obiettivo della recovery rapida
L’obiettivo della recovery rapida è permettere alla scheda di tornare operativa in tempi molto ridotti, cercando di riprendere da uno stato coerente e vicino a quello precedente al reset.

Il target prestazionale preciso dovrà essere validato sul sistema reale; un valore inferiore a 100 ms è considerato un obiettivo ambizioso ma desiderabile.

### 6.3 Snapshot dello stato essenziale
Per abilitare il fast reboot, la piattaforma può prevedere il salvataggio periodico di uno snapshot dello stato essenziale della macchina in memoria adatta alla sopravvivenza ai reset software.

Tale snapshot non deve contenere indiscriminatamente tutta la RAM, ma solo le informazioni essenziali a descrivere lo stato della scheda, ad esempio:

- stato delle FSM principali
- stato delle uscite
- timer significativi
- contesto operativo minimo necessario al recovery

### 6.4 Distinzione tra recovery rapida e persistenza
La recovery rapida e la persistenza dei dati non devono essere confuse.  
Il fast reboot serve a recuperare rapidamente il contesto operativo volatile.  
La persistenza serve invece a salvare informazioni che devono sopravvivere a spegnimenti e perdita alimentazione.

---

## 7. Persistenza e gestione power-loss

### 7.1 Distinzione tra dati statici e dati operativi
La piattaforma deve distinguere chiaramente tra:

- parametri di configurazione che cambiano raramente (i parametri della scheda)
- dati operativi che cambiano frequentemente (es ore di funzioanmnto delle spine, stato delle uscite)
- stato volatile utile solo al recovery rapido (timer attivi)

### 7.2 Salvataggio persistente dei parametri stabili
I parametri che cambiano raramente possono essere salvati su storage persistente in modo diretto e tradizionale.

### 7.3 Gestione dei dati ad alta frequenza di aggiornamento
Per dati che cambiano frequentemente nel tempo, ad esempio:

- ore di funzionamento
- contatori runtime
- accumuli di utilizzo

deve essere adottata una strategia che eviti usura eccessiva dello storage persistente.

### 7.4 Gestione della perdita alimentazione
Ogni scheda dovrebbe prevedere, ove possibile, un meccanismo per rilevare una condizione di perdita alimentazione o brownout e tentare il salvataggio delle informazioni essenziali prima dello spegnimento completo.

La fattibilità e l’affidabilità reale di tale strategia dipenderanno dal margine energetico disponibile e dalla tecnologia di memoria utilizzata.

solitamnte si ottinee con una rete di condensatori in parallelo all'alimnetazione per garantire una sorta di batteri atampone per dare il tempo necessario al salvataggio delle informazioni essenziali

### 7.5 Persistenza minima ragionata
Il sistema non deve inseguire la persistenza totale indiscriminata, ma deve salvare in modo persistente solo ciò che è realmente necessario mantenere nel tempo.

---

## 8. Concorrenza e accesso alle risorse

### 8.1 Uso disciplinato della concorrenza
L’uso della concorrenza deve seguire regole chiare, evitando accessi disordinati e non tracciabili alle risorse condivise.

### 8.2 Ownership delle periferiche
Ogni periferica o risorsa condivisa dovrebbe avere un owner logico chiaro.  
I moduli che necessitano di usarla devono interagire attraverso interfacce ben definite, evitando accessi concorrenti arbitrari.

### 8.3 Producer-consumer sulle risorse condivise
Per risorse come seriali, bus o canali di output condivisi, è preferibile adottare pattern di tipo producer-consumer, in cui:

- i moduli producono richieste
- un componente dedicato gestisce l’accesso reale alla risorsa

### 8.4 Serializzazione delle operazioni
Laddove necessario, le operazioni concorrenti devono essere serialize in modo esplicito, per evitare corse critiche, corruzione dello stato o contese difficili da diagnosticare.

### 8.5 Evitare logica distribuita in più task sulla stessa risorsa
Più task non dovrebbero implementare in modo indipendente logiche parziali sulla stessa periferica o sullo stesso dato condiviso.

---

## 9. Logging, debug e osservabilità firmware

### 9.1 Logging strutturato
La piattaforma deve prevedere un sistema di logging strutturato, utile a supportare sviluppo, test, commissioning e manutenzione in campo.

### 9.2 Tag e filtraggio
I log dovrebbero poter essere categorizzati tramite TAG o meccanismi equivalenti, così da permettere il filtraggio selettivo delle informazioni rilevanti.

### 9.3 Livelli di verbosità
Il sistema di logging dovrebbe supportare livelli di verbosità diversi, così da distinguere tra:

- produzione
- diagnostica ordinaria
- debug approfondito

### 9.4 Riduzione dell’impatto in produzione
Deve essere possibile ridurre o limitare l’impatto dei log in produzione, evitando che la diagnostica comprometta il comportamento runtime normale.

### 9.5 Osservabilità locale e remota
Quando utile, le informazioni di diagnostica dovrebbero poter essere rese disponibili sia localmente sia tramite i meccanismi di osservabilità del sistema.

---

## 10. Gestione centralizzata di stato, input e output

### 10.1 Single Source of Truth
La piattaforma deve favorire un modello in cui lo stato significativo della scheda abbia una fonte di verità unica o chiaramente centralizzata.

### 10.2 Evitare duplicazioni dello stato
Non dovrebbero esistere copie locali indipendenti degli stessi dati significativi, specialmente quando tali dati influenzano comportamento, temporizzazioni o transizioni logiche.

### 10.3 Astrazione degli input
Gli input devono essere gestiti tramite astrazioni di livello superiore rispetto alla semplice lettura del pin, così da esporre in modo uniforme comportamenti come:

- pressione
- rilascio
- pressione breve
- pressione lunga
- trigger temporali
- condizioni logiche osservabili

### 10.4 Astrazione degli output
Gli output devono essere gestiti tramite interfacce che rendano semplici e coerenti operazioni come:

- attivazione persistente
- attivazione temporizzata
- spegnimento
- blink
- settaggio di livello o stato

### 10.5 Timer e logiche temporizzate standardizzate
Le logiche temporizzate non dovrebbero essere reinventate in ogni modulo.  
La piattaforma deve favorire primitive comuni, riusabili e ben definite.

### 10.6 Supporto a osservatori e reattività interna
Una gestione centralizzata dello stato può favorire pattern osservativi o event-driven interni alla scheda, semplificando sincronizzazione, UI locali e aggiornamento coerente dei moduli.

---

## 11. Requisiti OTA di piattaforma

### 11.1 Obiettivo generale
La piattaforma deve supportare aggiornamenti OTA robusti, verificabili e il più possibile sicuri rispetto a installazioni incomplete, corruzione dei dati o firmware non validi.

### 11.2 Principi generali OTA
La gestione OTA deve prevedere, per quanto possibile:

- validazione dell’integrità del contenuto scaricato
- verifica di dimensioni e compatibilità
- meccanismi di rollback
- tracciamento dello stato dell’aggiornamento
- separazione chiara tra software attivo e software candidato

### 11.3 Architettura preferibile
L’architettura preferibile prevede un componente di boot dedicato, stabile e il meno possibile soggetto ad aggiornamento, con il compito di:

- gestire la connettività necessaria all’OTA
- contattare il server di aggiornamento
- scaricare gli artefatti necessari
- validarne integrità e correttezza
- gestire selezione, commit o rollback della versione software

In questa visione, la struttura della memoria dovrebbe favorire la duplicazione delle immagini e delle componenti persistenti necessarie a consentire rollback e ripristino coerente della macchina.

### 11.4 Obiettivo della duplicazione
La duplicazione non deve limitarsi, idealmente, al solo firmware eseguibile, ma dovrebbe estendersi alle componenti software e persistenti rilevanti per ripristinare in modo coerente la macchina dopo un aggiornamento fallito o un software non valido.

### 11.5 Validazione del software installato
Il componente di boot può garantire la correttezza formale del download e dell’installazione, ma la validazione finale del software può richiedere verifiche eseguite dal firmware installato stesso, ad esempio su:

- montaggio del filesystem
- disponibilità delle risorse necessarie
- correttezza di sottosistemi applicativi essenziali

### 11.6 Alternativa architetturale
Qualora non sia possibile adottare un boot component dedicato con responsabilità OTA estese, il firmware applicativo installato deve farsi carico direttamente della gestione del download, della validazione e della sicurezza dell’aggiornamento.

### 11.7 Coordinamento OTA tra molte schede
Nel sistema complessivo potrà essere necessario adottare strategie di aggiornamento scaglionato, a gruppi o sottoinsiemi di nodi, per evitare sovraccarichi di rete o problemi infrastrutturali durante aggiornamenti massivi.

La strategia concreta di coordinamento resta da definire a livello di sistema.

---

## 12. Vincoli e decisioni aperte

### 12.1 Intensità del vincolo anti-memoria dinamica
L’uso della memoria dinamica è fortemente scoraggiato, ma il perimetro preciso delle eccezioni ammissibili dovrà essere valutato in relazione a librerie, stack di rete e moduli specifici.

### 12.2 Target reale del fast reboot
Il target di reboot rapido dovrà essere validato in funzione di:

- hardware scelto
- costo del restore
- quantità di stato da recuperare
- vincoli del sistema operativo

### 12.3 Strategia concreta di brownout save
La reale efficacia della persistenza su perdita alimentazione dipenderà dalla capacità di rilevazione, dal margine energetico disponibile e dalle caratteristiche dello storage.

### 12.4 Profondità dell’architettura OTA
La fattibilità della duplicazione estesa di firmware, storage, web e metadati dipenderà da memoria disponibile, complessità e target hardware effettivi.

### 12.5 Scelta del perimetro delle FSM
L’uso delle FSM deve essere ampio ma non eccessivo.  
Resta da definire meglio in quali sottosistemi esse debbano essere considerate obbligatorie, consigliate o non necessarie.

### 12.6 Bilanciamento tra robustezza e complessità
Molti dei principi esposti in questo documento puntano alla massima robustezza, ma dovranno essere bilanciati con costi di implementazione, risorse hardware disponibili e necessità reali del prodotto finale.

---

## 13. Gestione della connettività Wi-Fi

### 13.1 Obiettivo generale
Le schede che supportano connettività Wi-Fi devono adottare una gestione della connessione robusta, asincrona e non bloccante, tale da non compromettere il normale funzionamento runtime della scheda.

La gestione Wi-Fi deve essere progettata come sottosistema autonomo, capace di tentare connessioni, rilevare disconnessioni, eseguire retry e cambiare strategia senza introdurre blocchi nelle altre funzionalità della scheda.

### 13.2 Principio di asincronia
La gestione della connettività Wi-Fi non deve basarsi su attese bloccanti o loop di connessione che interrompano il normale comportamento della macchina.

Le operazioni di:

- scansione reti
- tentativo di connessione
- riconnessione
- fallback
- cambio rete

devono essere gestite in modo asincrono, con stato esplicito e progressione controllata nel tempo.

### 13.3 Modalità di funzionamento
La piattaforma deve supportare almeno due modalità logiche di gestione della connettività Wi-Fi.

#### 13.3.1 Modalità orientata alla rete
In questa modalità, la priorità della scheda è essere connessa a una specifica rete definita dall’utente o dalla configurazione.

La scheda deve quindi:

- cercare la rete target specificata
- tentare la connessione preferibilmente a quella rete
- considerare la mancata connessione a tale rete come condizione significativa
- evitare, salvo policy esplicite, di collegarsi automaticamente a reti alternative solo perché disponibili

Questa modalità è adatta ai casi in cui la rete selezionata ha un significato preciso per l’installazione.

#### 13.3.2 Modalità orientata alla connessione
In questa modalità, la priorità della scheda non è essere collegata a una rete specifica, ma essere connessa a una delle reti note e autorizzate disponibili.

La scheda deve quindi:

- mantenere in memoria un insieme di reti note
- cercare le reti disponibili nell’ambiente
- collegarsi a una rete valida tra quelle memorizzate, secondo una politica di selezione definita
- poter cambiare rete in base al contesto, purché resti all’interno dell’insieme delle reti autorizzate

Questa modalità è concettualmente simile al comportamento di dispositivi mobili che si connettono dinamicamente alla migliore rete conosciuta disponibile.

### 13.4 Reti di fallback hardcoded
La piattaforma deve poter prevedere la presenza di una o più reti hardcoded di fallback, utilizzabili come risorsa di emergenza o di recupero.

Queste reti non sostituiscono la configurazione dell’utente, ma costituiscono un meccanismo di continuità operativa in assenza delle reti normalmente previste.

### 13.5 Strategia di fallback
Quando la scheda non riesce a trovare o utilizzare le reti definite dall’utente, deve poter passare a una strategia di fallback orientata alla connessione.

In tale condizione la scheda può:

- abbandonare temporaneamente la ricerca esclusiva delle reti utente
- cercare una rete disponibile tra quelle hardcoded autorizzate
- tentare la connessione a tali reti secondo una priorità definita

Questo comportamento deve essere esplicito e controllato, non implicito o casuale.

### 13.6 Persistenza e priorità delle reti
La piattaforma deve distinguere chiaramente tra:

- reti configurate dall’utente
- reti memorizzate per uso ordinario
- reti hardcoded di fallback

La politica di priorità tra queste categorie deve essere definita in modo esplicito, così da rendere prevedibile il comportamento della scheda nei diversi scenari di connessione.

### 13.7 Stabilità della connessione
La gestione Wi-Fi deve privilegiare la stabilità complessiva della scheda rispetto a cambi di rete troppo frequenti o aggressivi.

In particolare devono essere evitati, per quanto possibile:

- retry troppo rapidi e continui
- oscillazioni frequenti tra reti diverse
- scansioni eccessive che degradano il funzionamento runtime
- riconnessioni che monopolizzano CPU o memoria

### 13.8 Disconnessione e recovery
La perdita della connessione Wi-Fi non deve compromettere il funzionamento essenziale della scheda.

Il sottosistema Wi-Fi deve poter:

- rilevare la disconnessione
- aggiornare il proprio stato interno
- tentare il recovery secondo la modalità attiva
- rendere osservabile lo stato della connettività al resto del firmware

### 13.9 Separazione tra connettività e logica applicativa
La logica applicativa della scheda non deve dipendere in modo rigido dai dettagli del processo di connessione Wi-Fi.

Il resto del firmware deve interagire con il sottosistema Wi-Fi tramite uno stato chiaro e una interfaccia stabile, ad esempio sapendo se la scheda è:

- non inizializzata
- in scansione
- in connessione
- connessa
- in fallback
- in errore
- senza rete disponibile

### 13.10 Decisioni aperte
Restano da definire in una fase successiva:

- la politica concreta di selezione tra più reti valide
- le soglie e i tempi di retry
- il comportamento preciso di uscita dal fallback
- l’eventuale esposizione all’utente dello stato della strategia Wi-Fi
- la possibilità di distinguere tra fallback temporaneo e fallback persistente