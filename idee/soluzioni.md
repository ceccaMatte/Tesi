come risolbvere i 3 problemi principali
1) rete unica wirless wired a livello di layer di trasporto ovvero aver eun unica rete mesh dove i nodi possono vedersi o cablati o wirless es due nodi non si vedono come ponte radio allora li collego con un cavo così si vedono
2) resolver nome logico su matter es NODE_1 con ipv6 corripondente quidni come fare una rete distribuita dove ogni nodo abbia la possibilità di risolvere il proprio ipv6
3) come parlare ipV6 su rs485 / canbus

le soluzioni che mi sembrano più fattibili sono le seguenti
1) NON posso relaizzare una rete mesh ibrida wirles wired unico modo possibile è astrarre dal layer fisico e vedrle come un unica rete nel layer applicativo quidni fisicamnete non ho tutti i dispositivi sotto la stessa partizione thread ma vedo tutti i miei dispositivi come appartenenti alla stessa fabbric qundi li unisco come unico domino dal punto id vista di layer applicativo con Matter
2) come creare una rete completamnete distribuita per sistemare il problema del resolver nome logico <-> IpV6? Semplicemnte non posso fare una rete completamente distribuita e lo risolvo con la ridondanza metto più nodi che assolvono a questo compito quidni non ho una ret perfettamente distribuita ma non mi interesse perchè risolvo con ridondanza per risolvere il problema del SPOF
3) per parlare IPv6 su rs485 / canbus esistono già soluzioni studiate open source. La migliore sarebbe usare Zephyr per usare 6LoCAN Zephyr è un RTOS tipo FreeRTOS compatibile con esp32 che implementa già nativamente IPv6 over canbus
