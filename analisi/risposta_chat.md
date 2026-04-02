spiegami cosa intendi per fail-safe per le azioni fisiche
ti rispondo alle integrazioni
- gestione manual override questo è un NON porblema nelseso che se io ho imposato una scena che dice luce_1 On e tapperlla 50% e luce_2 On e mentre sto eseguendo la scena e magri non è ancorastato completata esempio al luce_1 è oN e la tapparella è al 50% ma naca la luce_2 ad accendersi, ma arriva un evento che il cui effetto è luce_2 OFF allora quello che si fa è un merge temporale nel senso l'ultimo richiesta di stato che arriva è quella che prevale.
- gestione stati temporanei anche questo è un no problmea del senso se io mando eveto_1 accesnioe luce 1 per X min io emetto evento e poi io monitoro lo stato. allora io pullbico lo stato in cui magari oltre a luce ON inidco anche il tempo in cui pemango nello stato es 10 indica 10min qudini è un infomrazione che posso anche ocmunicare, ma poi lato gateway posso scegliere se monitorare solo lo stato della luce nel senso on o off e igonare il fatto temporale assumendo che la scheda lo gestirà bene o anche ocnrollare che il valore riportato del tmepo rimanente acceso prima dello spegnimento sia corretto, poi da decidere a posteriori, ma si va citato il fatto che lo stato anche se binario possa necessitare di una indicazione di durata temporale (anche se credo che lo lascerò come gestione interna al nodo) 
- gestione profili / scene contestuali questo invece non è un blmea perchè ogni vento di fatto è come se fosse una scena un evnt che ha come conseguenza può esser evista come una scena che ha un unica conseguenza qudini distinzione tra evento singolo signolo e scena per me ha poco senso
- questa cosa la lascerei configurabile, nel senso post reset io posso decidere su farsi che la spina ritoni allo stato precedente al reset o torni in uno stato di default
- dell'intera linea o del singolo device? il singolo device sarebbe bello < 200ms
- scalabilità nel mio caso ne ho 45 in un caso gnerale sarebbe bello se la strutura ne supportasse 128 gateway incluso. poi ovvimanet valuteremo nel concret s epossibile o se dobbiamo scendere a 64
- no non serve che i nodi siano sincronizzatitra loor anche sarebbe estremamnete difficile (o meglio no mi vengono in mente scenari che richiedano sinconizzazione temporale)
- cosa intendi con per scenari, log, allarmi, correlazione eventi?

- idealmnet sarebbe bello comunicazione ibrida calbalte e wrless, ma per ora va benissiomo cabalta o wirless non entrambe assieme (va comunqe studiato nel pratico come raggiungere questo risultato)

Si condivido che serva un documneto a parte per la sezione 7
- in merito alla 7 si centro la scoperta di un nodo mancante va scoperta a run time tramite heart breath e la sostituazone non è alro che il rinserimento in corsa di un nodo che era andato down, o te per rimozione intedni proprio eliminazione totale di un nodo dalla rete? in quel caso quello si traduce in una riscirttura della mappa globale per i nodi

la perte 9 inmerito alla gestione dei guasti è importante e dovrà essere affrontata ma stiam aprendo davvero molti apsettiin parallelo faccio fatica a seguirli tutti in modo chiaro

si son d'accrod punto 10 biogna dire chiaramnte quali punti sono ancora aprti per ora


1) sicurezza e utorizzazionei no è un problma che dobbiamo porci adesso ma puoi citarlo tra le cose che andrebbero chiarite
2) come ho già detto ogni device deve essere scelto dall'instalatorese cosa fare ci dovranno essere 3 modalità 
   1. ripristino stato precedne se la luce era accesa torna accesa
   2. default dopo un blackout un device torna allo stato di defult
   3. cston l'installatore può scegliere quale stato far imposatre post balckout
3) la getione die conflitto come ho detto va gestita mediante merge tenporale l'ultimo comando che arriva decide lo stato
4) per conportamnte offlne intedno che il gateway non è un elemento forndamntale è un cordinatore che migliora la velocità delle comunicazioni della casa, ma non è indispensabile per il funzionamneto del sistema, ogni elento è si utile ma non indispiensabile, se muore un nod il resto della rete restie e non collassa semplicemente ovviamnte le capacità legate a quel nodo smettono di essre disponibili, un link degradato es filo tagliato comprometterà il nodo connesso al quel cavo, sa vinee recisa la dorsale princibale risano attivi i doni a monte e moriranno tutti quelli a valle
5) no ho capito cosa intedni


