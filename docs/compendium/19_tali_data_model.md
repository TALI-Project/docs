# TALI Design Hub — Section 10: Data Model & Storage Strategy

## 19. Data Model & Storage Strategy
> Defines TALI’s conceptual data architecture, including how information is structured, stored, and accessed across multiple databases. This section outlines strategy and purpose, not concrete schema definitions.

---

### 19.1 Overview
TALI’s data model is **polyglot** — each data store is chosen for its strengths. The system prioritizes scalability, fault isolation, and analytical depth rather than one-size-fits-all storage. Core principles:

- Use the right database for the right workload.
- Maintain clear data ownership boundaries between subsystems.
- Ensure all stores remain observable and queryable via unified metrics and dashboards.
- Preserve human interpretability *after* AI-focused design (TALI learns first, humans analyze second).

---

### 19.2 Storage Stack Overview
| Database | Role | Data Type | Persistence | Example Use |
|-----------|------|------------|--------------|--------------|
| **PostgreSQL** | System of record | Structured relational | Long-term | Sessions, residents, configurations, and logs |
| **Redis** | Real-time cache / bus | Key-value, ephemeral | Volatile | Voting state, live presence, sandbox signals |
| **Neo4j** | Context graph | Nodes & relationships | Persistent | Mapping context, devices, people, and correlations |
| **DuckDB** | Analytical engine | Columnar | Local transient | On-demand analytics and AI sampling |
| **InfluxDB** | Time-series (HA source) | Numeric, timestamped | External | Historical sensor and energy metrics |
| **TimescaleDB** | Hybrid relational time-series | Numeric, relational | External | Structured long-term HA event data |
| **Object Storage (MinIO/S3)** | Artifacts / model persistence | Binary | Durable | ML weights, sandbox results, logs, backups |

---

### 19.3 Data Domains
Each major subsystem owns and governs its own domain-specific data model.

| Domain | Primary Store | Description |
|---------|----------------|--------------|
| **Core & Config** | PostgreSQL | Environment setup, runtime configs, linked devices, resident metadata. |
| **Persona & Behaviour** | PostgreSQL + Redis | Overlay profiles, tone weights, behavioural logs, conversation context. |
| **Learning & Sandbox** | PostgreSQL + Object Storage | Simulation outcomes, reinforcement learning metrics, hypothesis logs. |
| **Context Graph** | Neo4j | Relationship mapping between people, devices, events, and automations. |
| **Telemetry & Metrics** | InfluxDB / TimescaleDB | Real-time measurements from HA and TALI sensors. |
| **Analytics Layer** | DuckDB | Query engine for aggregated insights and ML feature extraction. |

---

### 19.4 Conceptual Relationships
```
[Residents] ←→ [Devices] ←→ [Contexts]
      │              │             │
      ▼              ▼             ▼
  [Persona]     [Learning]     [Automation]
        \          |          /
         →  [Graph Context Layer (Neo4j)]
```

- **Residents** tie to **Persona overlays**, linked accounts, and trust scores.
- **Devices** connect to HA entities and rooms, forming part of the **Context Graph**.
- **Learning** nodes simulate and evolve automations based on real-world events.
- **Automation** edges capture cause-effect pathways for reinforcement feedback.

---

### 19.5 Data Ingestion & Flow
```
[Home Assistant / MQTT / Messenger] → [TALI Core Ingress] → [Redis Cache] → [PostgreSQL / Neo4j / DuckDB]
```
1. **Ingress Layer** receives events from external systems.
2. **Redis** temporarily buffers or relays transient state.
3. **Core Services** enrich and persist normalized events.
4. **Analytical Pipelines** (via DuckDB) perform local aggregations and feed model retraining.

---

### 19.6 Context Graph (Neo4j)
The graph database is used to represent relationships TALI discovers:
- *Who interacts with what* (person → device edges).
- *What depends on what* (automation → condition edges).
- *Where things happen* (room → sensor → context cluster).
- *Temporal reinforcement* (actions linked to feedback over time).

This allows queries such as:
> “Show all automations influenced by Maddie’s evening behaviour in the living room.”

---

### 19.7 Analytical Layer (DuckDB)
DuckDB is embedded for **local analytical queries** and model introspection. It operates as an in-process SQL engine with read access to recent Postgres snapshots and cached Redis datasets.

Used for:
- Exploratory feature generation.
- Validation of AI hypotheses against historical metrics.
- On-device learning or LLM fine-tuning insights.

Data processed here is transient — TALI regenerates it from canonical stores as needed.

---

### 19.8 Data Synchronization & Schema Evolution
TALI employs a self-managed **schema versioning and migration system** to evolve its data model as features change.

#### 19.8.1 Automated Schema Evolution
- Each module declares its schema version (e.g., `core@1.3.0`).
- TALI tracks schema history in an internal registry table (`tali_schema_history`).
- Migrations are defined declaratively (YAML/SQL hybrid) and version-tagged per subsystem.
- During startup, the migration engine automatically:
  1. Detects outdated or missing structures.
  2. Generates diffs using AI-assisted suggestions (based on data access patterns).
  3. Applies migrations sequentially with rollback checkpoints.
  4. Logs schema versions to Grafana and internal events (`tali.analytics.schema.updated`).
- Neo4j schema updates are handled via GraphML deltas, while Redis and DuckDB schemas are ephemeral and regenerated.

**Impact:**
- Prevents schema drift between modules.
- Enables rapid iteration and experimentation without data corruption.
- Serves as groundwork for model evolution and traceability — every structural change is version-tagged, traceable, and reversible.

---

### 19.9 Security, Access Controls & Observability
- Database credentials stored in Docker secrets and rotated regularly.
- Role-based schema access — core modules read/write only their own domains.
- Sensitive data (voice embeddings, resident identifiers) encrypted at rest.
- External databases (Influx/Timescale) accessed read-only through secure HA integration.
- Grafana dashboards unify metrics across all stores (health, IO, latency, retention).

---

### 19.10 AI-Driven Anomaly Detection
TALI continuously monitors all database performance and health metrics to identify abnormal trends.

#### 19.10.1 Detection Pipeline
- **Data Source:** Prometheus exporters for each store feed telemetry (latency, query count, IO, memory usage).
- **Baseline Model:** Exponentially Weighted Moving Average (EWMA) with seasonal decomposition models baseline behaviour.
- **Anomaly Model:** Isolation Forest and rule-based classifiers identify outliers or performance degradation.
- **Alert Flow:** Detected anomalies generate `tali.analytics.anomaly.detected` events.

#### 19.10.2 Response & Actions
- TALI autonomously reacts to anomalies by:
  - Adjusting caching or retry strategies.
  - Isolating problematic queries and routing to fallback databases.
  - Notifying residents or developers via Messenger I/O with natural-language summaries.
  - Entering **Safe Mode** if data integrity is at risk, suspending AI retraining.

#### 19.10.3 Benefits
- Early detection of schema or workload instability.
- Self-healing through adaptive caching or rebalancing.
- Insightful analytics for long-term tuning of AI workloads.

---

### 19.11 Backup & Retention
- Scheduled backups to MinIO with integrity verification.
- Retention policies:
  - Core records: 12 months.
  - Persona / interaction logs: 6 months.
  - Analytical data: ephemeral (regenerated on demand).
  - Audio/media artifacts: transient and anonymized.
- Backup verification integrated into schema evolution cycle.

---

### 19.12 Optional Future Enhancements
- Temporal joins between graph edges and sensor data.
- Optional federation for multi-home environments.

---

*(Section 19 now integrates automated schema evolution and AI-driven anomaly detection as core systems — enabling TALI to self-manage, adapt, and monitor its data layer with minimal manual intervention.)*

