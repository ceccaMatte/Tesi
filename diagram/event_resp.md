```mermaid
sequenceDiagram
    participant NE as Nodo Event Generator
    participant NA as Nodo Attuatore

    NE->>NA: EVENT_FRAME (Evento_X)
    NA->>NA: Verifica mapping evento→azione
    NA->>NA: Esecuzione azione
    NA-->>NE: STATE_FRAME (Nuovo stato)
