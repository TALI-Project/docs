# TALI Design Hub — Section 11: System Services & Module Overview

## 20. System Services & Module Overview
> Defines TALI’s modular service architecture, outlining the purpose, boundaries, and responsibilities of each runtime module within the ecosystem.

---

### 20.1 Overview
TALI’s architecture is designed around a **modular core system** — a collection of interoperating services, each responsible for a specific domain of intelligence, interaction, or data processing. Modules can run:
- As part of a unified **Docker Compose stack**, or
- Natively on the host in **standalone mode**, using local resources (e.g., Mac M-series LLM acceleration).

Each module communicates through TALI’s **Event System & Message Bus** (Section 9), ensuring loose coupling and asynchronous coordination.

---

### 20.2 Core Modules
| Module | Type | Description |
|---------|------|--------------|
| **tali-core** | Service | The central orchestrator. Hosts configuration registry, service discovery, and the primary event dispatcher. Manages lifecycle of submodules and integrates with the learning, persona, and database systems. |
| **tali-learning** | Service | Executes the sandbox and reinforcement logic (see Section 4). Responsible for hypothesis generation, mutation/recombination testing, and applying reward functions. |
| **tali-persona** | Library / Service | Governs the Persona Engine (see Section 5). Provides tone, behavioural overlay, and emotional response models for voice and text interactions. |
| **tali-messenger** | Service | Handles all text-based I/O including Discord and dashboard interfaces (see Section 8). Converts chat interactions into structured intent payloads. |
| **tali-voice** | Service | Manages speech recognition, voice activity detection, TTS synthesis, and speaker identification (see Section 7). Communicates via MQTT and Redis Streams. |
| **tali-dataservice** | Service | Abstracts access to Postgres, Redis, Neo4j, and DuckDB. Exposes unified APIs for persistence, schema management, and data enrichment. Handles automated schema evolution (Section 10). |
| **tali-analytics** | Service | Collects metrics, traces, and anomalies across modules. Integrates with Prometheus, Loki, and Grafana for observability. Hosts anomaly detection AI for database and event health. |
| **tali-ingress** | Adapter | Ingests data from Home Assistant (HA), MQTT, or other external systems. Normalizes payloads before publishing them to the Event Bus. |
| **tali-actuator** | Adapter | Executes commands back into HA, MQTT, or external APIs. Performs safety validation before dispatching any automation or control signal. |

---

### 20.3 Supporting Components
| Component | Role |
|------------|------|
| **Config Server** | Centralized configuration registry for environment variables and connection settings. Supports overrides via `.env` or HA secrets. |
| **Event Broker** | Facilitates message routing between modules using Spring’s ApplicationEventPublisher and external MQTT broker. |
| **Cache Layer (Redis)** | Real-time state synchronization, transient session data, and fast pub/sub signalling. |
| **Database Layer** | Composed of Postgres (truth), Neo4j (context), DuckDB (analytics), and Influx/Timescale (telemetry). |
| **LLM Adapter Layer** | Provides unified abstraction over local and cloud-based LLMs. Exposes structured JSON schema interfaces to upstream modules. |
| **Security Kernel** | Manages encryption keys, identity linking, and trust levels. Provides anomaly hooks to suspend unsafe operations. |

---

### 20.4 Service Interaction Model
```
[tali-ingress] → [Event Bus] → [tali-core]
[tali-core] → [tali-learning] ↔ [tali-dataservice]
[tali-core] → [tali-persona] → [tali-voice / tali-messenger]
[tali-analytics] ↔ [All Modules]
[tali-actuator] ← [tali-core]
```
All modules subscribe to their relevant namespaces (e.g., `tali.learning.*`, `tali.persona.*`). This event-driven model minimizes direct coupling and maximizes scalability.

---

### 20.5 Module Lifecycles
| Phase | Description |
|--------|--------------|
| **Startup** | `tali-core` initializes configuration, validates dependencies, and broadcasts `tali.core.startup.ready`. |
| **Registration** | Modules self-register with the Core via discovery topics (`tali.core.register.module`). |
| **Health Probing** | Each service exposes `/actuator/health` and reports liveness through periodic event pings. |
| **Runtime** | Services operate autonomously, exchanging messages and telemetry via the Event Bus. |
| **Shutdown** | Ordered teardown ensures active learning cycles and schema migrations complete gracefully. |

---

### 20.6 Dependency Model
Dependencies are layered to ensure deterministic startup order:
1. **Core & Config** (foundation)
2. **DataService / Cache / Event Broker**
3. **Learning, Persona, and Analytics**
4. **Voice and Messenger I/O**
5. **Ingress / Actuator Adapters**

Circular dependencies are avoided via the event bus — no service holds a hard reference to another.

---

### 20.7 Containerization & Deployment
Each service runs in its own Docker container. A standard Compose file includes:
- All core services.
- Database stack (Postgres, Redis, Neo4j, MinIO).
- Observability stack (Prometheus, Grafana, Loki).
- External connectors (MQTT broker, optional Kafka).

In **standalone mode**, `tali-core`, `tali-learning`, and `tali-persona` run natively, while data and observability services remain containerized.

---

### 20.8 Monitoring & Self-Healing
- Each module emits telemetry under `tali.analytics.*` events.
- Health status monitored via the Analytics service and Grafana dashboards.
- Fault detection triggers recovery actions — e.g., restart failed modules, roll back schema, or isolate unhealthy queues.
- Learning engine can adapt runtime behaviour to system load (e.g., scaling simulation intensity).

---

### 20.9 Extensibility & Modularity
TALI supports a **pluggable module system** — new services can be registered dynamically at runtime:
- Modules declare their type and namespace.
- Core registers capabilities and dependencies.
- Hot-deployable modules load through the Plugin Manager (Java SPI-based).

Example:
```
module:
  name: tali-weather
  type: adapter
  version: 1.0.0
  subscribes: ["tali.context.weather.*"]
  provides: ["forecast.events"]
```

---

### 20.10 Future Enhancements
- Module discovery through a distributed registry for multi-host deployments.
- Adaptive load balancing for simulation-heavy services.
- Visual topology map auto-generated from the Event Bus.

---

*(Section 20 defines the internal service structure of TALI — a modular, event-driven ecosystem where each component performs a distinct cognitive or operational role while remaining loosely coupled and self-healing.)*

