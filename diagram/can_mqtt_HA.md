```mermaid
sequenceDiagram
    participant N as Nodo Event Generator
    participant G as Gateway MQTT
    participant B as MQTT Broker
    participant HA as Home Assistant

    N->>G: EVENT_FRAME (Evento_X)
    G->>G: Osservazione traffico\n(decodifica evento)
    G->>B: Publish evento
    B->>HA: Notify evento
    HA->>HA: Aggiornamento UI / Stato logico
