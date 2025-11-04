# TALI Design Hub — Section 2: System Architecture & Sandboxing

## 2. System Architecture
> Defines how TALI is structured internally and how it interfaces with Home Assistant and other systems.

### 2.1 Core Modules (Proposed)
| Module | Responsibilities | Notes |
|---------|-------------------|------|
| **Core Orchestrator** | Boot, lifecycle, scheduling, health, config management. | Single authority for start/stop of everything. |
| **Sandbox Manager** | Creates/tears down isolated execution sandboxes; enforces quotas/timeouts. | Isolation mode configurable per sandbox (thread ⟶ classloader ⟶ process). |
| **Message Bus** | In‑process pub/sub for domain events (ingest → learn → act). | Pluggable transports later; default in‑memory. |
| **Integration Layer** | Connectors for MQTT, HA API, DBs; normalizes events. | All external I/O goes through here. |
| **Learning Engine** | Pattern mining, hypothesis generation, evaluation. | Runs inside a sandbox by default. |
| **Persona Engine** | Dialogue strategy, tone shaping, prompt routing to speech/alerts. | No TTS vendor assumption here. |
| **Toolbox Runtime** | Executes generated tools/skills; maintains capability registry. | Tools are versioned and signed by Core. |
| **Persistence Layer** | Abstraction for state, features, models, artifacts. | Postgres/Timeseries/etc. are implementation details. |
| **Observability** | Metrics, logs, traces, audit, event journal. | First‑class: required for learning & safety. |

### 2.2 Runtime & Deployment Model (Proposed)
TALI runs as a **single process** with **multiple isolation domains** (sandboxes). The default favors **fast iteration and rich in‑memory signals** over distributed complexity. External services (MQTT broker, DBs, Grafana, etc.) remain separate containers.

**Why single process?**
- Lower latency between ingest → learn → act.
- Easier transactional rollbacks across modules.
- Simpler local debugging and hot‑reload.

### 2.3 Event & Data Flow (Proposed)
```
[External Sources]
  MQTT / HA API / DB History
         │
         ▼
   [Integration Layer] ──▶ Normalized Events (E*)
         │                         │
         │                         ├──▶ [Event Journal] (append‑only)
         │                         └──▶ [Feature Store] (derived features)
         ▼
      [Message Bus]
         │
         ├──▶ [Learning Engine ▣ sandbox]
         │        ├─▶ Hypotheses H*
         │        └─▶ Metrics & Feedback
         │
         ├──▶ [Toolbox Runtime ▣ sandbox]
         │        └─▶ Actions A* (MQTT/API)
         │
         └──▶ [Persona Engine ▣ sandbox]
                  └─▶ Notifications/Prompts
```

**Journaling as a safety net:** Every normalized event is journaled before consumption. Sandboxes can be rewound/replayed deterministically for testing.

---

## 2.4 Sandboxing (Proposed)
TALI supports **three isolation tiers**, selectable per workload:

1. **Thread Isolation** – fastest; shared classloader/memory; safe for read‑only analytics.
2. **Classloader Isolation** – modules load under distinct classloaders; controlled API surface via interfaces; enables hot‑swap and unload.
3. **Process Isolation** – separate OS process with IPC; highest safety for untrusted or runaway code, at a performance cost.

**Default policy:** Learning & Toolbox run in **Classloader Isolation**; Persona may run in Thread Isolation; experimental code or vendor binaries run in Process Isolation.

### 2.5 Sandbox Contract
All sandboxes implement a common lifecycle:
- `init(context)` → receives capability tokens (no raw sockets/files by default)
- `start()` → begins consuming events from bus (scoped topics)
- `heartbeat()` → liveness + health data
- `snapshot()` / `restore(snapshot)` → state checkpointing
- `stop(reason)` → graceful shutdown

**Capabilities (least privilege):**
- Event Read (scoped topics)
- Action Emit (scoped domains: mqtt://topic/*, ha://service/*)
- Storage Access (namespaced KV, blob, tables)
- Time Budget (CPU wall/CPU time)
- Network (off by default; proxied through Integration Layer when needed)

---

### 2.6 Safety, Quotas & Circuit Breakers
- **Resource quotas:** per‑sandbox CPU time, memory soft/hard limits, action rate limits.
- **Timeouts:** execution step wall‑time; cascading timeouts to avoid thundering herds.
- **Circuit breakers:** trips on consecutive failures or error ratio; auto‑half‑open probes.
- **Kill switches:** user or Core can disable any sandbox or capability at runtime.
- **Audit trail:** every action is recorded with hypothesis/tool version and inputs.

### 2.7 Failure Handling & Recovery
- **Crash‑only design:** failures trigger automatic teardown + clean re‑create.
- **Rewind & replay:** reproduce bugs deterministically from the Event Journal.
- **Quarantine mode:** repeatedly failing sandboxes are disabled and require manual re‑enable.
- **Fallback hierarchy:** if MQTT action fails → try HA API; if both fail → notify Persona.

### 2.8 Config & Hot‑Reload
- Single canonical config; changes are versioned and broadcast on the bus.
- Classloader sandboxes can be hot‑reloaded on artifact change.
- Feature flags gate risky algorithms/tools; staged rollout per capability.

### 2.9 Observability & Debuggability
- **Metrics:** per‑sandbox CPU/mem, queue lag, hypothesis accuracy, action success rate.
- **Logs:** structured JSON with sandbox id, capability, correlation id.
- **Traces:** ingest→learn→act spans with baggage for hypothesis/tool version.
- **Inspectors:** live views: event stream tail, capability registry, sandbox health.

---

### 2.10 Extensibility Hooks (Planned)
- Plugin hooks for Java AppDaemon modules to register sandbox-safe capabilities.
- Developer introspection API for inspecting event pipelines and sandbox health.
- Dynamic class reloading via Core Orchestrator for live development.

---

*(Section 2 captures TALI’s modular architecture, sandbox execution model, safety constraints, and runtime design philosophy.)*

