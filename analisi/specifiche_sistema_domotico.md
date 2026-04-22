# Specifiche di alto livello del sistema domotico

## 1. Scopo del documento
Questo documento definisce le specifiche di alto livello del sistema domotico oggetto del progetto.

Lo scopo del documento è descrivere:

- gli obiettivi del sistema
- le proprietà architetturali desiderate
- le capacità funzionali richieste
- i requisiti operativi e non funzionali
- i vincoli, le assunzioni e i punti ancora aperti

Questo documento non definisce ancora nel dettaglio:

- l’implementazione firmware delle singole schede
- i pattern software interni
- la distribuzione definitiva dei ruoli logici sulle schede fisiche
- i dettagli di protocollo o di algoritmo

Servirà come base per derivare, in una fase successiva:

- i ruoli logici del sistema
- i limiti fisici e tecnologici
- le strutture necessarie
- il mapping tra ruoli logici e componenti fisici

---

## 2. Obiettivo del sistema
L’obiettivo è realizzare un sistema domotico modulare, distribuito, riconfigurabile e osservabile, capace di funzionare in modo autonomo anche in assenza di connettività Internet e senza dipendere da un controllore centrale per la logica essenziale della casa.

Il sistema deve consentire il controllo e il monitoraggio della casa sia in locale sia da remoto, deve essere integrabile con sistemi esterni come Home Assistant e deve essere progettato con particolare attenzione a:

- robustezza
- affidabilità
- manutenzione
- diagnostica
- flessibilità di espansione
- aggiornabilità nel tempo

Il sistema deve inoltre poter essere installato sia in contesti con infrastruttura cablata sia in contesti in cui sia preferibile una maggiore semplicità installativa tramite comunicazione wireless.

---

## 3. Principi architetturali
Il sistema deve rispettare i seguenti principi architetturali.

### 3.1 Funzionamento distribuito
La logica fondamentale della casa non deve dipendere da un elemento centrale unico.  
Le reazioni agli eventi devono poter essere distribuite tra i nodi del sistema.

### 3.2 Funzionamento offline
La rete domotica deve poter funzionare correttamente anche in assenza di accesso a Internet.  
Il Wi-Fi e la connettività esterna non devono essere necessari per il funzionamento base della casa.

### 3.3 Funzionamento anche senza gateway
Il gateway non deve essere un elemento indispensabile al funzionamento essenziale del sistema.  
La sua assenza può ridurre supervisione, coordinamento e integrazione esterna, ma non deve impedire il funzionamento locale della casa.

### 3.4 Stato come fonte di verità
Il sistema deve basarsi sullo stato osservato e pubblicato dai nodi, e non su assunzioni ottimistiche legate alla sola ricezione dei messaggi. 

### 3.5 Auto-descrizione della rete
I nodi devono poter essere scoperti automaticamente e devono poter descrivere le proprie capacità, i device che controllano e gli eventi che possono generare.

### 3.6 Separazione tra programmazione e runtime
La configurazione della logica della casa deve essere logicamente separata dal funzionamento runtime.  
La logica evento → azione deve poter essere definita, aggiornata e distribuita senza richiedere necessariamente modifiche del firmware applicativo dei nodi.

### 3.7 Controllo locale prioritario
Le funzioni essenziali della casa devono poter essere eseguite localmente, ad esempio tramite pulsanti fisici o interfacce locali, anche in assenza dei sistemi esterni.

### 3.8 Comandi assoluti e comportamento idempotente
Il sistema deve privilegiare comandi assoluti e comportamenti idempotenti, evitando per quanto possibile azioni relative o ambigue come toggle o incrementi/decrementi dipendenti dal contesto.

### 3.9 Merge temporale dei conflitti
In caso di richieste concorrenti o contrastanti sullo stesso device, il sistema deve adottare una politica di merge temporale in cui l’ultima richiesta valida ricevuta determina lo stato desiderato corrente.

### 3.10 Separazione tra identità tecnica e semantica utente
Il sistema deve distinguere tra identificazione tecnica stabile dei device e semantica utente/installativa associata agli stessi, così da favorire configurazione, manutenzione, diagnostica e interfacce utente più chiare.

---

## 4. Capacità funzionali richieste

### 4.1 Controllo domotico di base
Il sistema deve permettere il controllo di dispositivi e funzioni domestiche, tra cui:

- luci
- spine
- dimmer
- tapparelle
- sensori ambientali
- controllo della temperatura / riscaldamento per ambiente

### 4.2 Interazione locale e remota
Il sistema deve poter essere controllato:

- localmente tramite pulsanti fisici
- localmente tramite tablet o pannelli installati nelle stanze
- da remoto tramite Home Assistant
- opzionalmente, tramite sistemi vocali esterni integrati attraverso Home Assistant

### 4.3 Scenari e automazioni
Il sistema deve permettere la programmazione di scenari e automazioni distribuite.  
In particolare deve essere possibile definire relazioni evento → effetti, ad esempio:

- pressione breve o lunga di un pulsante
- attivazione di sensori
- combinazioni di eventi
- scenari domestici predefiniti

Dal punto di vista logico, anche scenari complessi devono poter essere trattati come eventi che producono effetti su più device.

### 4.4 Diagnostica locale e globale
Il sistema deve supportare:

- self-check locale di ogni scheda
- verifica dello stato di funzionamento del proprio hardware
- comando di autodiagnostica globale della casa
- produzione di un report che identifichi con precisione quali schede e quali device risultano guasti o degradati

### 4.5 Discovery e riconfigurazione
Il sistema deve consentire:

- scansione della rete
- scoperta automatica dei nodi presenti
- lettura delle capacità di ogni nodo
- popolamento automatico dei device e degli eventi disponibili
- riconfigurazione della logica della casa
- aggiunta di nuovi nodi con impatto minimo sul funzionamento ordinario, ove possibile

### 4.6 Sicurezza / allarme distribuito
Il sistema deve poter integrare un sottosistema di allarme distribuito, ad esempio basato su sensori acustici installati presso le finestre.

L’obiettivo è consentire una valutazione comparativa dei segnali provenienti da punti diversi della casa, così da aumentare la sensibilità verso eventi locali anomali e ridurre i falsi positivi dovuti a rumori diffusi o comuni.

### 4.7 Energia e consumi
Il sistema deve poter supportare:

- rilevazione dei consumi elettrici
- monitoraggio in tempo reale della potenza assorbita
- identificazione dei principali carichi energivori
- generazione di notifiche al superamento di soglie configurate
- eventuale possibilità di intervento sui carichi controllabili

### 4.8 Supporto a stati arricchiti
Alcuni device possono richiedere stati non puramente binari, ma arricchiti con attributi ulteriori, ad esempio:

- livello
- posizione
- durata residua
- modalità operativa
- parametri ambientali

La gestione di tali attributi deve essere supportata dal modello del sistema.

---

## 5. Requisiti operativi e non funzionali

### 5.1 Latenza locale
Nel caso di interazione locale standard, ad esempio pressione di un pulsante e attivazione del relativo device, il tempo di risposta sul singolo device dovrebbe essere inferiore a 200 ms.

### 5.2 Latenza interna in condizioni gravose
Nel caso peggiore di traffico interno significativo, il tempo massimo di attesa per l’effetto osservabile all’interno della rete dovrebbe restare inferiore a 1 secondo.

### 5.3 Latenza per comandi esterni
Nel caso di controllo remoto tramite sistemi esterni, ad esempio Home Assistant, la latenza target dovrebbe essere inferiore a 1.5 secondi.

### 5.4 Scalabilità
Il sistema è pensato per supportare configurazioni reali dell’ordine di alcune decine di nodi.  
Come obiettivo architetturale, la struttura dovrebbe essere estendibile fino a circa 128 nodi complessivi, gateway incluso, compatibilmente con i vincoli reali del mezzo trasmissivo e dell’hardware scelto.

### 5.5 Aggiornabilità
Ogni scheda del sistema deve poter essere aggiornata via OTA.

### 5.6 Osservabilità
Lo stato della casa deve essere osservabile in modo coerente e sufficientemente completo, così da consentire controllo, supervisione, integrazione esterna e diagnostica.

### 5.7 Robustezza al degrado
Il sistema deve continuare a operare in modo degradato ma controllato anche in presenza di fault parziali, ad esempio:

- assenza del gateway
- perdita della connettività esterna
- guasto di un nodo
- degrado o interruzione di una porzione della rete

In tali casi il resto del sistema non deve collassare, ma devono semplicemente diventare indisponibili le capacità direttamente dipendenti dagli elementi guasti.

---

## 6. Comunicazione e connettività

### 6.1 Supporto a modalità cablata e wireless
Il sistema deve poter operare sia su infrastruttura cablata sia su infrastruttura wireless.

### 6.2 Preferenza del canale cablato
In presenza di collegamento cablato, questo costituisce il canale preferenziale per robustezza, stabilità e prestazioni di comunicazione.

### 6.3 Valore della modalità wireless
La comunicazione wireless è richiesta come modalità fondamentale per installazioni semplificate, retrofit e scenari plug-and-play.

### 6.4 Coesistenza delle modalità
Le schede devono essere progettate in modo da poter supportare entrambe le modalità di comunicazione.  
Nella fase iniziale del progetto non è necessario assumere l’uso simultaneo dei due mezzi all’interno dello stesso comportamento runtime; il modello concreto di coesistenza o complementarità verrà definito in una fase architetturale successiva.

---

## 7. Diagnostica, manutenzione e osservabilità

### 7.1 Self-check locale
Ogni scheda deve poter eseguire un self-check dei propri elementi controllati o monitorati, per quanto consentito dall’hardware disponibile.

### 7.2 Diagnostica globale della casa
Deve essere possibile lanciare una procedura di autodiagnostica globale della casa, che raccolga le informazioni provenienti da tutte le schede e restituisca un report strutturato.

### 7.3 Identificazione precisa del guasto
In presenza di un malfunzionamento, il sistema deve consentire di risalire con precisione:

- al nodo coinvolto
- al device coinvolto
- alla natura del problema, per quanto osservabile

### 7.4 Manutenzione in campo
Il sistema deve essere pensato per favorire manutenzione, troubleshooting e sostituzione dei componenti nel tempo.

---

## 8. Comportamento in condizioni anomale e post-riavvio

### 8.1 Gestione del post-blackout / post-reboot
Ogni device deve supportare una politica configurabile di comportamento al riavvio, con almeno le seguenti modalità:

1. ripristino dello stato precedente
2. ritorno a uno stato di default
3. impostazione di uno stato custom definito in configurazione

### 8.2 Persistenza del resto della rete
In caso di guasto o reboot di una singola scheda, il resto della rete deve continuare a funzionare entro i limiti delle capacità disponibili.

### 8.3 Impatto dei guasti di rete
In presenza di interruzioni fisiche della rete, ad esempio un collegamento reciso, devono restare operative tutte le porzioni di impianto che risultano ancora raggiungibili.

---

## 9. Integrazione con sistemi esterni

### 9.1 Integrazione con Home Assistant
Il sistema deve poter essere integrato con Home Assistant tramite un componente gateway in grado di esporre stato, eventi e possibilità di controllo.

### 9.2 Controllo remoto
Deve essere possibile controllare la casa da remoto attraverso tale integrazione.

### 9.3 Integrazione con ecosistemi vocali
Il sistema dovrebbe poter essere controllato anche tramite ecosistemi vocali esterni, ad esempio Google Home, tramite integrazione veicolata da Home Assistant.

### 9.4 Estensioni future
Il sistema potrà evolvere in futuro verso integrazioni più avanzate, ad esempio:

- accesso locale alle periferiche tramite interfacce per modelli di IA
- assistenza locale intelligente
- digital twin della casa
- utilizzo del digital twin per diagnostica e controllo evoluto

Queste funzionalità non costituiscono al momento il nucleo minimo del progetto, ma rappresentano possibili direzioni evolutive.

---

## 10. Vincoli, assunzioni e decisioni aperte

### 10.1 Tecnologia wireless
La tecnologia wireless definitiva non è ancora stata scelta.

### 10.2 Coesistenza cablato / wireless
Il modello concreto di utilizzo complementare o alternativo dei due mezzi di comunicazione deve ancora essere definito.

### 10.3 Hot-add dei nodi
L’aggiunta di nuovi nodi a caldo, senza fermo dell’intera casa, è un obiettivo desiderato ma richiede approfondimento tecnico e architetturale.

### 10.4 Profondità del self-check
La reale capacità di autodiagnostica dei device dipenderà dalle soluzioni hardware adottate e dai meccanismi di misura o feedback disponibili.

### 10.5 Gestione dell’OTA
La strategia concreta di aggiornamento OTA, compreso l’eventuale coordinamento tra molte schede e la gestione di aggiornamenti su sottoinsiemi di nodi, deve essere definita in una fase successiva.

### 10.6 Sicurezza e autorizzazioni
La gestione della sicurezza, delle autorizzazioni, dell’aggiunta di nodi, della programmazione della rete e del controllo remoto costituisce un tema importante ma non ancora approfondito in questo documento.

### 10.7 Sincronizzazione temporale
La necessità di una sincronizzazione temporale stretta tra nodi non è al momento considerata un requisito di base, ma potrà essere rivalutata qualora emergano scenari che lo richiedano.

### 10.8 Rimozione definitiva o sostituzione di nodi
La gestione strutturata della rimozione definitiva di nodi dalla rete o della loro sostituzione deve essere chiarita meglio nelle fasi successive di analisi.

---