# TALI Design Hub — Section 3: Technical Stack & Environment

## 3. Technical Stack & Environment
> Defines the concrete technologies, frameworks, and runtime environment used by TALI.

### 3.1 Language & Runtime
- **Primary Language:** Java 25 (LTS)
- **Concurrency Model:** Virtual Threads (Project Loom) for lightweight, high-throughput task handling.
- **Runtime Model:** Monolithic core with sandboxed modules under an Event Driven Architecture (EDA).
- **Hot Deployment:** Classloader-based reloading for rapid iteration.
- **Build Tool:** Gradle 9 (Kotlin DSL) with modular source sets for each subsystem.

### 3.2 Frameworks & Libraries
| Category | Technology | Purpose |
|-----------|-------------|----------|
| **Application Framework** | Spring Boot 3.5.7 | Core orchestration, dependency injection, actuator metrics. |
| **Event Framework** | Spring Application Events + Reactor | Internal event-driven communication and reactive flow control. |
| **Messaging** | Eclipse Paho / HiveMQ Client | MQTT communication between HA and TALI. |
| **Data Serialization** | Jackson + Protobuf | Structured serialization of learning models and messages. |
| **Sandboxing & Security** | Custom TALI Sandbox API | Capability-based isolation and execution lifecycle. |
| **Logging / Tracing** | Logback + OpenTelemetry | Structured logs, distributed tracing, and metric export. |
| **Scripting / Plugins** | Java-based AppDaemon Layer | Allows developers to build custom automations in Java. |
| **Configuration Management** | Spring Config + YAML | Unified configuration, environment profiles, and secrets. |

### 3.3 Databases & Storage
TALI employs a multi-database strategy optimized for both state consistency and contextual learning.

| Layer | Technology | Purpose |
|--------|-------------|----------|
| **Source of Truth (Relational)** | PostgreSQL | Central relational database storing entities, events, automations, and learned state. |
| **Cache / Real-time State** | Redis | Provides low-latency caching and live state reflection for fast lookups. |
| **Graph Context Store** | Neo4j | Maps relationships between entities, occupants, automations, and behaviours for contextual inference. |
| **Local Analytical Engine** | DuckDB | Embedded analytical database for ad-hoc learning queries and feature extraction. |
| **Object Storage** | MinIO | Stores binary artifacts such as trained models, snapshots, and plugin JARs. |
| **External (via Home Assistant)** | InfluxDB + TimescaleDB | Time-series and event history data hosted by HA; TALI connects directly for analysis and learning backfill. |

### 3.4 Message & Event System
| Component | Technology | Description |
|------------|-------------|--------------|
| **Primary Bus** | MQTT (EMQX or Mosquitto) | Primary observation and action transport for HA events. |
| **Internal Event Bus** | Spring Event Publisher + Reactor | Core EDA backbone; events flow between subsystems asynchronously. |
| **Optional Analytics Queue** | Kafka (future) | For replayable analytical workloads or distributed model training. |

### 3.5 Observability Stack
TALI ships with **observability out of the box**. The same bundle runs via Docker Compose in both containerized and standalone modes.

| Component | Technology | Purpose |
|------------|-------------|----------|
| **Metrics & Dashboards** | Grafana | Unified UI for system, learning, and home metrics.
| **Metrics Collection** | Prometheus + Micrometer (in-app) | Scrapes JVM, app, and custom metrics from TALI Core.
| **Logs** | Loki + Promtail | Centralized structured logs with labels (sandbox, capability, correlation id).
| **Traces** | OpenTelemetry SDK (in-app) + OTel Collector + Tempo | End-to-end tracing (ingest → learn → act) stored in Tempo.
| **Health/Uptime** | Uptime Kuma | External availability checks for core services.

#### 3.5.1 Default Dashboards & Alerts
- **System Health:** JVM, virtual thread pools, sandbox quotas, GC, heap.
- **Event Pipeline:** ingest rate, queue lag, hypothesis accuracy, action success.
- **Integrations:** MQTT throughput, HA API latency/error rates.
- **Storage:** Postgres/Redis/Neo4j health, connection pools, slow queries.
- **Alerts:**
  - Sandbox crash loop (quarantine threshold exceeded)
  - Hypothesis false-positive spike
  - MQTT action error ratio > N% over 5m
  - HA API fallback frequency spike

### 3.6 Containerization & Deployment
TALI supports **two operational modes** — containerized and standalone — both designed for local-first intelligence with **the same observability bundle** included in Docker Compose. Containerization & Deployment
TALI supports **two operational modes** — containerized and standalone — both designed for local-first intelligence with shared supporting infrastructure.

#### 3.6.1 Docker Compose Mode
In this mode, TALI runs fully containerized for consistent deployment and isolation.

- **Container Platform:** Docker Compose (multi-container setup for local-first deployment)
- **Images:**
  - `tali-core` — Java 25 application container.
  - `postgres`, `redis`, `neo4j`, `minio` — core persistence services.
  - `mqtt`, `grafana`, `prometheus`, `loki`, `uptime-kuma` — observability and monitoring stack.
- **External Dependencies (via HA instance):** InfluxDB and TimescaleDB (queried remotely).
- **Networking:** All containers share a common internal bridge network for low-latency communication.
- **Volumes:** Persistent volumes for Postgres, Redis, Neo4j, and MinIO data.
- **Resource Constraints:** Defined per container to prevent resource starvation.

#### 3.6.2 Standalone Mode
In this configuration, **TALI Core** runs natively on the host machine (e.g., macOS with M‑series chips) to leverage **local LLM performance** and direct hardware access. All other supporting systems — databases, brokers, and observability services — continue to run in Docker for isolation and reproducibility.

- **Core Runtime:** Native JVM execution of TALI for maximum hardware utilization.
- **Supporting Services:** Same Docker Compose stack as containerized mode, minus the `tali-core` container.
- **Communication:** Native process connects to Docker network using host bridge for database and broker access.
- **Use Case:** Ideal for power users or developers who want tighter integration with local resources (e.g., on‑device LLMs or GPUs) while retaining consistency in data and metrics.

### 3.7 Security & Privacy
- **Local-first design:** All computation and learning occur locally by default.
- **Cloud Offload:** With user consent, TALI may offload anonymized model-retraining tasks to cloud LLMs (PII redacted).
- **Secrets Management:** Managed via environment variables or Docker secrets.
- **Data Retention:** Configurable policies for historical data pruning and HA backfill duration.

### 3.8 Developer Experience
- **Hot reload and local development via Docker Compose.**
- **Gradle tasks for sandbox testing, rebuilds, and dependency graphs.**
- **Integrated Grafana dashboards for introspection.**
- **Structured logs for tracing learning loops and event propagation.**
- **AppDaemon Java SDK for plugin authoring with full IDE autocompletion and sandbox API access.**

---

*(Section 3 formalizes TALI’s modern, event-driven technical foundation — built on Java 25, Spring Boot 3.5.7, and a multi-database stack for learning, context, and performance.)*

