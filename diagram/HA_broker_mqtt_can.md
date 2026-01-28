```mermaid
sequenceDiagram
    participant HA as Home Assistant
    participant B as MQTT Broker
    participant G as Gateway MQTT
    participant N as Nodo Destinatario

    HA->>B: Publish CMD (Topic comando)
    B->>G: Notify comando
    G->>G: Traduzione MQTT → CAN
    G->>N: CMD_FRAME (Comando diretto)
