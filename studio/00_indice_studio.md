# Appunti — Matter over Thread

Questa cartella raccoglie appunti separati per argomento, pensati per chiarire i ruoli logici, il commissioning e il funzionamento pratico di una rete Matter over Thread.

## Ordine consigliato di studio

1. `01_ruoli_logici_thread_matter.md`
2. `02_commissioning_thread.md`
3. `03_commissioning_matter.md`
4. `04_border_router_border_agent_commissioner.md`
5. `05_discovery_operativa_nodeid_ipv6.md`
6. `06_multi_border_router_split_merge.md`
7. `07_schema_rapido_e_domande_tipiche.md`

## Obiettivo degli appunti

Capire bene:

- quali sono i ruoli logici distinti;
- quali ruoli appartengono a Thread;
- quali ruoli appartengono a Matter;
- cosa fa davvero il Border Router;
- cosa fa il Border Agent;
- cos'è un Commissioner;
- chi autorizza l'ingresso nella rete;
- come avviene il mapping tra Node ID Matter e IPv6;
- perché più Border Router ridondanti sono una soluzione standard e robusta.

## Regola mentale fondamentale

Quando analizzi un componente, chiediti sempre:

1. **Che ruolo ha nella rete Thread?**
2. **Che ruolo ha nel commissioning Thread?**
3. **Che ruolo ha nel livello Matter/IP?**

Questa tripla distinzione evita quasi tutte le confusioni.
