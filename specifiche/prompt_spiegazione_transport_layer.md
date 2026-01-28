# CONTEXT PROMPT — Custom Embedded Home Automation Network

You are working with a **custom embedded home automation network**.  
This prompt defines the **exact architecture, philosophy, constraints and features** of the system.  
All answers, designs and suggestions must strictly adhere to the following specifications.

---

## 1. General Overview

The system is a **wired, embedded, event-driven home automation network** composed of ~40 nodes.

- The network is **fully functional without internet, cloud, or gateway**
- A **MQTT gateway** exists only to observe state and inject commands from Home Assistant
- The gateway is **not required** for normal operation and is **not a central authority**

The design prioritizes:
- reliability
- low latency
- resilience
- long-term maintainability
- minimal complexity (no over-engineering)

---

## 2. Physical Layer

- Shared **RS-485 half-duplex bus**
- All nodes are connected to the same shared medium
- Every transmitted frame is physically received by all nodes
- UART-based communication (e.g. 115200 or 250k baud)
- Framing + CRC are used for integrity

---

## 3. Network Model

- **Multi-master**: any node can transmit
- **Publish/Subscribe on shared bus**
- All nodes always listen
- A node reacts **only if the message is semantically relevant to it**

There is:
- no master
- no token
- no central scheduler
- no routing
- no consensus protocol (no RAFT / Paxos)

---

## 4. Event-Driven Architecture

The network is **event-driven**, not command-stream–driven.

Messages represent:
- events (“something happened”)
- or explicit commands (“do this action”)

**Complex logic is configured at programming time**, not at runtime.

---

## 5. Two Event Semantics

### 5.1 Declarative Events (Local Event-Driven Logic)

Declarative events describe **facts**, not actions.

Examples:
- `BTN_KITCHEN_PRESSED`
- `PRESENCE_DETECTED`
- `TEMP_ABOVE_THRESHOLD`

Characteristics:
- Minimal or no payload
- Do NOT specify what actions to perform
- Each node contains a local binding:
  - `event_id → action`
- Used for:
  - physical buttons
  - sensors
  - stable automations
- Designed to work even without the gateway

---

### 5.2 Explicit Command Events (Command-Driven)

Explicit events describe **exact actions**.

Examples:
- `CMD_LIGHT_SET { target=LIGHT_1, value=OFF }`
- `CMD_COVER_SET { target=COVER_3, position=60 }`

Characteristics:
- Explicit payload
- Typically single-target
- Used mainly by the MQTT / Home Assistant gateway
- No scene or binding logic required on recipients

---

## 6. Quality of Service (QoS)

### QoS0 — Best Effort
- No ACK
- No retry
- Used for:
  - heartbeat
  - periodic state
  - telemetry
  - noise sensor features
  - diagnostics

### QoS1 — Reliable Delivery
- Requires ACK
- Retry on timeout
- Used for:
  - commands
  - critical events

**QoS1 success condition:**
> The sender has received ACK from all expected recipients.

---

## 7. ACK Model (Multicast Reliable Delivery)

- ACKs are sent **only by nodes for which the event is relevant**
- Nodes that are not recipients remain silent
- ACKs are scheduled using **deterministic time slots**
  - based only on local invariant data (e.g. node_id)
- This prevents ACK storms and collisions
- The sender opens a fixed ACK window and tracks missing ACKs
- Retries occur only if ACKs are missing

No node assumes knowledge of what other nodes received.

---

## 8. Sequence Numbers and Deduplication

- Each node maintains its own transmit sequence counter
- Message identity is `(src_id, seq)`
- Used for:
  - deduplication
  - retry correlation
  - ACK matching
- Idempotent actions are recommended but **do not replace protocol-level deduplication**

No global ordering is guaranteed or required.

---

## 9. Priority and Latency

Messages are prioritized:

1. EMERG (alarm, emergency)
2. CMD (commands)
3. CFG (configuration)
4. BG (heartbeat, state, telemetry)

Target:
- Critical commands latency < 50 ms (nominal)
- Emergency events have absolute priority

Low-priority traffic must yield to high-priority traffic.

---

## 10. Collision Handling and MAC Policy

- Bus idle detection before transmit
- Deterministic, priority-weighted backoff
- No token passing
- Collisions detected indirectly via:
  - CRC errors
  - missing ACKs

During an ACK window:
- Only ACKs and emergency traffic are allowed
- Background traffic is suspended

---

## 11. Node Liveness and Resilience

- Nodes send heartbeat messages periodically (QoS0)
- Each node maintains a local liveness table
- Node states:
  - alive
  - suspect
  - dead
- Recovery is automatic upon receiving new messages
- No global agreement on node state is required

---

## 12. State and Observability

- Nodes publish periodic state (QoS0)
- State is **eventually consistent**
- No strong global consistency is required
- The gateway observes traffic to expose state to Home Assistant

---

## 13. MQTT / Home Assistant Gateway

- Acts as a **bus sniffer**
- Publishes state to MQTT
- Can inject explicit command events
- Treated as a normal node:
  - has node_id
  - follows QoS, priority and ACK rules
- HA scenes generate **multiple individual commands**, not atomic scenes

---

## 14. Noise-Based Alarm Subsystem

- Window nodes use MEMS microphones
- Perform local FFT and frequency band analysis
- Share compact feature vectors (QoS0)
- Traffic characteristics:
  - ~2 Hz when alarm armed
  - background priority
  - jittered transmission
- Alarm logic is local to window groups
- Alarm decision produces a high-priority event (`ALARM_DETECTED`)
- Alarm system is fully local and independent from the gateway

---

## 15. Design Constraints and Philosophy

- No RAFT, Paxos or distributed consensus
- No global clock assumptions
- No assumption of perfect delivery
- Reliability is enforced only where required (commands)
- Telemetry and state are best-effort
- System must degrade gracefully

---

## 16. Key Principle

> **Reliable commands + eventual state convergence  
> instead of global consensus**

All designs, extensions and code must follow this principle.

---

You must respect this architecture and its constraints when:
- designing protocols
- proposing algorithms
- suggesting improvements
- writing firmware or gateway logic
