# Studio di Fattibilità — Trasporto del Protocollo Matter su Bus Cablato Multi-nodo

**Autore:** Matteo Ceccarelli  
**Data:** Aprile 2026  
**Contesto:** Tesi magistrale in Ingegneria Informatica  

---

## Executive Summary

Matter è uno standard domotico open source della CSA (Connectivity Standards Alliance) che impone IPv6 come unico trasporto. Nativamente supporta tre physical layer: Ethernet IEEE 802.3, Wi-Fi 802.11 e Thread IEEE 802.15.4. Nessuno di questi è un bus condiviso multi-drop cablato adatto a installazioni in rack con molti nodi a breve distanza.

Lo scenario di riferimento di questo studio è: **45 nodi domotici in rack da 1 metro**, comunicazione esclusivamente cablata, IPv6 obbligatorio (requisito nativo di Matter). Sono state valutate cinque soluzioni alternative a Ethernet standard con switch. Il risultato principale è che nessuna soluzione esistente risolve il problema in modo completo e documentato: il mondo embedded non ha ancora prodotto un bus condiviso multi-drop con IPv6 e Matter nativi.

---

## 1. Definizione del Problema

### 1.1 Requisiti tecnici del protocollo Matter

Matter utilizza IPv6 come layer di rete obbligatorio. I principali servizi che dipendono dalla rete sono:

- **mDNS (Multicast DNS):** discovery dei dispositivi tramite indirizzi multicast IPv6 `ff02::fb` su porta 5353
- **NDP (Neighbor Discovery Protocol):** risoluzione degli indirizzi link-layer tramite messaggi Neighbor Solicitation/Advertisement
- **CASE (Certificate Authenticated Session Establishment):** handshake crittografico ECDH su curva P-256 per stabilire sessioni sicure tra nodi, con messaggi fino a ~900 byte
- **Commissioning:** fase di onboarding tramite BLE o Wi-Fi per configurare le credenziali di rete

### 1.2 Scenario fisico

| Parametro | Valore |
|---|---|
| Numero di nodi | 45 |
| Lunghezza rack | ~1 metro |
| Distanza inter-nodo media | ~2.2 cm |
| MCU target | ESP32-C6 (Wi-Fi + BLE + CAN) |
| Requisito IP | IPv6 link-local obbligatorio |
| Requisito protocollo | Matter 1.x |

Il vincolo principale è la **densità**: 45 nodi in 1 metro rappresenta il caso d'uso peggiore per qualsiasi bus condiviso che imponga requisiti elettrici sul singolo segmento.

---

## 2. Soluzioni Valutate

### Soluzione 0 — Ethernet Standard + Switch Managed (baseline)

#### Descrizione

Ogni nodo monta un chip W5500 (MAC+PHY, interfaccia SPI) collegato tramite patch cable da 30 cm a uno switch managed 48 porte con IGMP Snooping abilitato. Lo switch gestisce il multicast IPv6 in modo selettivo, evitando la saturazione del bus.

#### Analisi

Questa è la soluzione di riferimento tecnologicamente matura. Matter e IPv6 funzionano senza alcuna modifica allo stack applicativo. Il chip W5500 ha driver stabili in ESP-IDF. Il costo totale hardware è stimato in circa 80€ per lo switch più ~3.5€/nodo per il W5500.

Il principale svantaggio è architetturale: richiede un apparato di rete aggiuntivo (lo switch), una topologia a stella con patch cable individuali per ogni nodo, e una dipendenza da un singolo punto di concentrazione. Non è un bus condiviso nel senso tradizionale del termine.

#### Fattibilità

✅ **Immediata.** Zero sviluppo software necessario. È la baseline rispetto alla quale tutte le soluzioni alternative si confrontano.

---

### Soluzione 1 — 10BASE-T1S (Single Pair Ethernet multidrop)

#### Descrizione

IEEE 802.3cg (2019) definisce 10BASE-T1S come Ethernet a tutti gli effetti su singolo doppino in topologia bus multidrop. Il meccanismo PLCA (Physical Layer Collision Avoidance) coordina l'accesso al mezzo tramite un beacon periodico del nodo coordinatore, garantendo accesso deterministico. Il chip LAN8650 di Microchip integra MAC, PHY e controller PLCA con interfaccia SPI verso il microcontrollore.

**Vantaggi principali:**
- Ethernet IEEE 802.3 nativo: Matter e IPv6 funzionano senza modifiche allo stack
- Singolo doppino per tutti i nodi, nessuno switch
- Driver Open Alliance TC6 open source, porting in corso per ESP-IDF
- Comunicazione deterministica configurabile tramite PLCA burst mode e Multi-ID

#### Limite critico: capacitanza stacked

Ogni nodo connesso al bus aggiunge al segnale una capacitanza parassita dovuta a PHY (~9 pF), ESD (<1 pF) e Common Mode Choke (<10 pF), per un totale di **18–25 pF per nodo**.

Lo standard IEEE 802.3cg è stato progettato per un massimo di **8 nodi su 25 metri**, con limite di capacitanza di 25 pF/nodo. L'analisi sperimentale condotta da Ford e presentata all'IEEE Ethernet Tech Day (2024) dimostra che:

- Con 8 nodi a 0.75 m di distanza: eye height ≥ 500 mV su tutti i nodi ✅
- Con 12 nodi a 0.75 m: eye height scende a 465 mV su un nodo (sotto soglia) ❌
- Con 15 nodi a 0.75 m: eye height 485 mV su nodo medio ❌
- Oltre 20 nodi: "Needs Further Testing and Investigation" (Ford, 2024)

Per il caso in esame, con 45 nodi a ~2.2 cm di distanza media:

\[ C_{totale} = 45 \times 20\,\text{pF} = 900\,\text{pF} \]

contro il limite progettuale di 200 pF. Il problema è **elettrico e non si risolve riducendo la velocità di trasmissione**, in quanto la degradazione dell'eye height è dovuta alla resistenza DC del cavo e al carico capacitivo, indipendenti dalla frequenza di simbolo.

**Possibile mitigazione:** segmentazione in 5 bus da 9 nodi ciascuno, collegati tramite switch 10BASE-T1S (es. Microchip LAN9662, NXP SJA1110). Tuttavia questi switch sono rari, costosi e progettati per automotive, rendendo la soluzione più complessa della baseline Ethernet.

#### Fattibilità

⚠️ **Fisicamente problematica** con 45 nodi in rack da 1 metro. Richiederebbe validazione sperimentale con analisi SI (Signal Integrity) su prototipo reale. La segmentazione è tecnicamente possibile ma aumenta costo e complessità.

---

### Soluzione 2 — 6LoCAN su CAN-FD

#### Descrizione

6LoCAN è un protocollo di adattamento IPv6 su bus CAN definito nel draft IETF `draft-wachter-6lo-can-00` (Alexander Wachter, TU Graz, ottobre 2019) e descritto in dettaglio nella relativa tesi magistrale (febbraio 2020). Il protocollo definisce:

- Uno schema di indirizzamento a **14 bit** per nodo, mappato nei 29 bit dell'identificatore CAN esteso
- Frammentazione e riassemblaggio tramite **ISO-TP** (ISO 15765-2), con supporto payload fino a 4095 byte
- Compressione header IPv6 tramite **IPHC** (RFC 6282), che può ridurre l'header IPv6+UDP da 48 byte a soli 4 byte in scenari link-local ottimali
- Un meccanismo di **Link-Layer Duplicate Address Detection (LLDAD)** per l'assegnazione dinamica degli indirizzi
- Un nodo **Ethernet Border Translator** per connettere segmenti 6LoCAN a reti Ethernet/IPv6 esterne

Con CAN-FD (64 byte di payload per frame), la maggior parte dei pacchetti IPv6 compressi con IPHC entra in un singolo frame, eliminando la frammentazione ISO-TP nei casi comuni.

#### Vantaggi

- CAN-FD è il physical layer ideale per rack densi: fino a 110 nodi su singolo bus, segnalazione differenziale robusta, nessuna limitazione di capacitanza analoga a 10BASE-T1S
- Arbitraggio hardware CSMA/CR integrato nel silicio di ogni controller CAN
- A 1 metro di lunghezza bus, CAN-FD supporta fino a **8 Mbps di data rate** — il massimo fisicamente possibile
- Costo transceiver CAN ~1€/nodo (SN65HVD230)
- Base teorica e proof-of-concept già esistenti (tesi Wachter + implementazione Zephyr)

#### Problemi aperti

**1. Stato del draft IETF e dell'implementazione:**
Il draft IETF è scaduto ad aprile 2020 e non è mai stato rinnovato né adottato da alcun Working Group. L'implementazione di riferimento in Zephyr RTOS è non funzionante dalla versione 2.3.0 (2021) per un null pointer dereference non corretto nelle API CAN aggiornate.

**2. Multicast mDNS su bus broadcast:**
Matter utilizza intensivamente il multicast IPv6 (mDNS `ff02::fb`, NDP solicited-node `ff02::1:ffXX:XXXX`). Su bus CAN ogni frame multicast viene ricevuto fisicamente da tutti i nodi. Il mapping degli indirizzi multicast IPv6 ai 14 bit del CAN ID (ultimi 14 bit del gruppo) può generare collisioni di hash tra gruppi diversi. Con 45 nodi che effettuano periodic announcements mDNS, il bus rischia saturazione prima che arrivi traffico applicativo utile.

**Possibile mitigazione:** mDNS proxy centralizzato sull'Ethernet Border Translator, che gestisce il discovery per conto di tutti i nodi CAN.

**3. Commissioning senza BLE/WiFi:**
Matter richiede BLE o Wi-Fi per la fase di onboarding standard. Alternativa praticabile: **commissioning on-network** (direttamente via IPv6 se il nodo è già sulla rete tramite Border Translator) oppure **pre-provisioning firmware** statico per installazioni rack fisse.

**4. CASE handshake e timeout:**
I messaggi crittografici CASE (~900 byte) richiedono almeno 15 frame ISO-TP su CAN-FD. Matter impone timeout di 1 secondo per il completamento dell'handshake. Con 45 nodi che si commissionano simultaneamente al boot, il bus può saturarsi causando scadenze dei timeout.

**Possibile mitigazione:** boot sequenziale dei nodi con delay programmato.

**5. Network driver custom:**
Lo stack `esp-matter` utilizza `esp_netif` orientato a interfacce WiFi ed Ethernet standard. Serve un driver custom che esponga IPv6 verso Matter SDK ma utilizzi 6LoCAN su CAN-FD come layer fisico sottostante. Richiede conoscenza approfondita delle API interne di ESP-IDF a livello di `esp_netif` e lwIP.

#### Fattibilità

⚠️ **Tecnicamente percorribile** ma richiede sviluppo significativo. Stimato 4–8 mesi di lavoro. È l'approccio architetturalmente più elegante — singolo cavo CAN-FD, nessuno switch, IPv6 e Matter nativi su ogni nodo — ma nessun progetto lo ha completato e reso pubblico.

---

### Soluzione 3 — Convertitore Ethernet ↔ CAN

#### Descrizione

Un gateway che traduce frame Ethernet provenienti da ogni nodo ESP32 in messaggi CAN-FD sul bus condiviso, e viceversa.

#### Motivo di esclusione

Ethernet è un protocollo commutato: il gateway dovrebbe mantenere una tabella di indirizzamento MAC per 45 nodi, gestire la frammentazione da MTU Ethernet (1500 byte) in frame CAN-FD (64 byte), e instradare unicast e multicast tra i nodi. In pratica si reimplementa uno switch Ethernet utilizzando chip CAN meno efficienti, con latenza aggiuntiva e senza driver maturi disponibili. La complessità implementativa supera quella della baseline Ethernet senza offrire vantaggi concreti.

#### Fattibilità

❌ **Non raccomandata.**

---

### Soluzione 4 — Gateway CAN-Thread con Matter Bridge (approccio aggregatore)

#### Descrizione

Architettura ibrida che evita il problema di portare IPv6 su CAN. I nodi communicano tramite un **protocollo proprietario su CAN-FD** — frame compatti con identificatore nodo, tipo di periferica e valore (es. 8 byte totali per stato tapparella o livello luce). Una scheda gateway centrale funge da **Matter Bridge**, che è un device type ufficialmente supportato dalla specifica Matter 1.x.

Il Bridge aggrega lo stato di tutti i nodi CAN e lo espone al controller Matter come dispositivo unico con centinaia di endpoint, ciascuno rappresentante una periferica fisica (luci, tapparelle, prese, sensori). Dal punto di vista del controller Matter esterno (Apple Home, Google Home, Home Assistant), il Bridge appare come un normale device Matter con endpoint individuali commissionnati e certificati.

Questo approccio rispecchia esattamente quello già adottato in produzione dai gateway commerciali KNX→Matter (es. 1Home) e Zigbee→Matter.

#### Vantaggi

- Nessun problema IPv6 su CAN: il gateway è il solo nodo con stack IP
- Il protocollo CAN proprietario è completamente custom, semplice e deterministico
- I nodi CAN restano MCU leggeri senza stack IP, con firmware minimal
- Compatibilità Matter piena lato controller: tutti i controller Matter vedono il bridge come device nativo
- Tecnica consolidata in produzione (KNX→Matter, Zigbee→Matter, Z-Wave→Matter)
- Sviluppo relativamente lineare: `esp-matter` Bridge device type è documentato nel SDK

#### Svantaggi

- Il bridge è un **single point of failure**: se cade, tutti i 45 nodi spariscono dall'ecosistema Matter
- **Non è "Matter nativo" per ogni nodo**: i nodi CAN non hanno identità Matter propria, né certificato CASE individuale, né possono essere commissionnati singolarmente
- Il protocollo CAN proprietario non è interoperabile con altri vendor
- Matter ha limiti pratici sul numero di endpoint per singolo device (verificare limite SDK)
- Non risolve il problema architetturale di fondo: resta un sistema eterogeneo con protocollo proprietario sul bus

#### Fattibilità

✅ **Alta** — è la soluzione più realistica per un prototipo funzionante a breve termine. Richiede sviluppo del firmware Bridge (esp-matter) e del protocollo CAN proprietario, entrambi ben documentati.

---

## 3. Confronto Sinottico

| Criterio | Ethernet + Switch | 10BASE-T1S | 6LoCAN / CAN-FD | CAN-Thread Bridge |
|---|---|---|---|---|
| **Bus condiviso (no switch)** | ❌ | ✅ | ✅ | ✅ |
| **IPv6 nativo su ogni nodo** | ✅ | ✅ | ⚠️ da sviluppare | ❌ solo sul bridge |
| **Matter nativo su ogni nodo** | ✅ | ✅ | ⚠️ da sviluppare | ❌ solo sul bridge |
| **Sviluppo necessario** | Zero | Basso | Molto alto | Medio |
| **Costo HW per nodo (stimato)** | ~3.5€ (W5500) | ~4€ (LAN8650) | ~1€ (transceiver) | ~0.5€ (transceiver) |
| **Rischio tecnico** | Nessuno | Alto (fisica) | Molto alto | Basso |
| **Maturità tecnologica** | Produzione | Sperimentale | Ricerca | Proof-of-concept |
| **Scalabilità nodi** | Alta (switch) | Critica (>15 nodi) | Alta (>100 nodi CAN) | Media (limiti Bridge) |
| **Single point of failure** | Switch | Bus coordinator | Border Translator | Gateway Bridge |

---

## 4. Contesto: Stato dell'Arte su IPv6 su Bus Industriali

Non esiste ad oggi alcun progetto open source maturo che implementi IPv6 su un bus condiviso cablato multi-drop in modo production-ready. I riferimenti disponibili sono:

- **6LoCAN (Wachter, 2020):** unico lavoro accademico strutturato su IPv6/CAN. Proof-of-concept funzionante su Classical CAN con Zephyr 2.0, mai esteso a CAN-FD, non mantenuto
- **RFC 9453 (6lo Use Cases, 2022):** descrive casi d'uso IPv6 su reti constrained ma non include CAN come physical layer
- **KNX→Matter bridge (1Home, commerciale):** bridge proprietario, non IPv6 nativo su bus KNX
- **Matter over Thread:** IPv6 su 802.15.4 wireless — risolve il problema per ambienti wireless ma non per rack cablati

Il gap identificato in questo studio — **Matter nativo su bus cablato multi-drop senza switch** — è genuinamente inesplorato a livello di ricerca e assente dal mercato commerciale.

---

## 5. Raccomandazione

### Per prototipo funzionante a breve termine
**Soluzione 4 (CAN-Thread Bridge):** consente di sviluppare hardware dei nodi CAN-FD con logica semplice e di avere un sistema Matter funzionante tramite bridge, senza bloccarsi sulla complessità dello stack IPv6/CAN. Ideale per una prima validazione del sistema.

### Per contributo di ricerca originale
**Soluzione 2 (6LoCAN su CAN-FD):** è il problema aperto più interessante, con una base accademica dalla quale partire (tesi Wachter, draft IETF) e un impatto potenziale elevato nell'ambito IIoT e domotica professionale. I problemi da risolvere (mDNS su bus broadcast, CASE timeout, netif driver custom) sono ingegneristicamente affrontabili ma richiedono mesi di sviluppo.

### Soluzione di produzione immediata
**Soluzione 0 (Ethernet + Switch):** funziona oggi, zero sviluppo, compatibilità Matter garantita. È il riferimento tecnico rispetto al quale tutte le soluzioni alternative si confrontano e giustificano.

