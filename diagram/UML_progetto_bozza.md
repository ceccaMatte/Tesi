```mermaid
classDiagram
    direction TB

    %% =========================
    %% TIPI DI EVENTI / COMANDI
    %% =========================

    class EventoDescrittivo {
        <<event>>
        +semantic_id
    }

    class EventoDichiarativo {
        <<event>>
        +target_device
        +desired_state
    }

    class CmdScheda {
        <<command>>
        +node_id
        +payload
    }

    %% =========================
    %% MESSAGGI
    %% =========================

    class Messaggio {
        +src_id
        +seq
        +qos
        +priority
        +payload
    }

    class Ack {
        +status
    }

    class Stato {
        +device_state
    }

    class Heartbeat {
        +uptime
    }

    class RispostaDiagnostica {
        +details
    }

    %% Relazioni tipo -> messaggio
    EventoDescrittivo --> Messaggio
    EventoDichiarativo --> Messaggio
    CmdScheda --> Messaggio

    Stato --> RispostaDiagnostica
    Ack --> RispostaDiagnostica
    Heartbeat --> RispostaDiagnostica

    RispostaDiagnostica --> Messaggio

    %% =========================
    %% NODO E COMPONENTI INTERNI
    %% =========================

    class Nodo {
        +node_id
        +reaction_engine
        +config
    }

    class EventGenerator {
        <<internal>>
    }

    class Device {
        <<actuator>>
        +state
    }

    Nodo "1" o-- "*" EventGenerator
    Nodo "1" o-- "*" Device

    %% =========================
    %% DEVICE SPECIALIZZATI
    %% =========================

    class Luce
    class Spina
    class Tapparella

    Device <|-- Luce
    Device <|-- Spina
    Device <|-- Tapparella

    %% =========================
    %% SCHEDA (RUOLO FISICO)
    %% =========================

    class Scheda {
        +hw_id
        +bus_interface
    }

    Scheda "1" o-- "1" Nodo

    %% =========================
    %% TIPO DI SCHEDA
    %% =========================

    class SchedaStandard
    class SchedaTapparella

    Scheda <|-- SchedaStandard
    Scheda <|-- SchedaTapparella

    %% =========================
    %% BUS E PROTOCOLLO
    %% =========================

    class ProtocolloBus {
        +framing
        +crc
        +mac
        +qos
    }

    Scheda "*" --> ProtocolloBus : comunica

    %% =========================
    %% ATTORI ESTERNI
    %% =========================

    class GatewayMQTT
    class Programmatore
    class SchedaDisplay

    GatewayMQTT --> ProtocolloBus
    Programmatore --> ProtocolloBus
    SchedaDisplay --> ProtocolloBus

    %% =========================
    %% FLUSSI LOGICI
    %% =========================

    Messaggio --> Scheda : ricevuto
    Scheda --> Messaggio : inviato
