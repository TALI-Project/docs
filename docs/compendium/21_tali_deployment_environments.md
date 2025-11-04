# TALI Design Hub — Section 12: Deployment & Runtime Environments

## 21. Deployment & Runtime Environments
> Defines how TALI is deployed, configured, and executed across supported environments — including containerized, hybrid, and standalone setups.

---

### 21.1 Overview
TALI supports multiple deployment modes to suit hardware capabilities and privacy preferences:

| Mode | Description | Typical Use Case |
|------|--------------|------------------|
| **Docker Compose (Full Stack)** | All TALI services plus database and observability stack run in containers. | Servers, NAS devices, HA hosts, or Linux workstations. |
| **Hybrid (Native Core + Docker Services)** | Core intelligence runs natively; databases and observability stack remain containerized. | macOS systems (especially M-series) leveraging local LLM acceleration. |
| **Standalone (Headless)** | Single native binary with minimal local dependencies; connects to remote databases if available. | Embedded or lightweight edge devices. |

---

### 21.2 Architecture Summary by Mode
```
Containerized: [Core + Modules + DBs + Grafana + MQTT]
Hybrid:        [Core(native) ↔ Docker(DBs + Analytics)]
Standalone:    [Core + Persona + Learning] (remote DB endpoints optional)
```

---

### 21.3 Deployment Components
Each mode includes varying subsets of the following services:

| Component | Required (Full) | Required (Hybrid) | Optional (Standalone) |
|------------|------------------|------------------|------------------------|
| tali-core | ✅ | ✅ | ✅ |
| tali-learning | ✅ | ✅ | ✅ |
| tali-persona | ✅ | ✅ | ✅ |
| tali-voice | ✅ | ✅ | ⚪ |
| tali-messenger | ✅ | ✅ | ⚪ |
| tali-dataservice | ✅ | ✅ | ⚪ (connects remotely) |
| tali-analytics | ✅ | ✅ | ⚪ |
| PostgreSQL | ✅ | ✅ | ⚪ (external) |
| Redis | ✅ | ✅ | ⚪ |
| Neo4j | ✅ | ✅ | ⚪ |
| DuckDB | ✅ | ✅ | ✅ |
| MinIO | ✅ | ✅ | ⚪ |
| Prometheus + Grafana + Loki | ✅ | ✅ | ⚪ |

---

### 21.4 Docker Compose Layout
A standard `docker-compose.yml` includes:
- All **core TALI services** (core, persona, learning, messenger, voice, analytics, dataservice).
- Supporting containers:
  - PostgreSQL, Redis, Neo4j, MinIO.
  - Grafana, Prometheus, Loki (observability stack).
  - EMQX or Mosquitto (MQTT broker).
- Optional services:
  - Kafka for extended event replay.
  - InfluxDB / TimescaleDB as Home Assistant integrations.

**File structure example:**
```
/tali/
 ├── docker-compose.yml
 ├── env/
 │   ├── core.env
 │   ├── db.env
 │   └── grafana.env
 ├── volumes/
 │   ├── postgres/
 │   ├── redis/
 │   ├── neo4j/
 │   └── minio/
 └── logs/
```

---

### 21.5 Standalone Runtime
In **standalone mode**, TALI is packaged as a single native binary built via **GraalVM Native Image**. This configuration:
- Runs locally without Docker overhead.
- Automatically discovers database endpoints if the full stack is reachable.
- Prefers **local LLM inference** (Ollama, LM Studio, GPT4All, etc.) for privacy and latency.
- Still connects to the same containerized observability and data services.

**Launch Example:**
```
$ tali-core --mode=standalone --env=prod
```

When running standalone, TALI exposes an internal web dashboard for configuration, monitoring, and messenger interaction.

---

### 21.6 Configuration & Environment Management
Configuration values are loaded from multiple layers in order of precedence:
1. **Command-line flags** (`--mode`, `--llm.provider=local`, `--db.url=...`)
2. **Environment variables** (via `.env` files in `/env/`)
3. **Config Server** (Section 11)
4. **Home Assistant secrets.yaml** integration (optional)

**Example:**
```yaml
tali:
  mode: hybrid
  llm:
    provider: local
    model: mistral:7b-instruct
  database:
    postgres: jdbc:postgresql://tali-db:5432/tali
  analytics:
    grafana_url: http://grafana:3000
```

---

### 21.7 Observability Stack
Regardless of mode, TALI always deploys or connects to its observability services:
- **Prometheus** for metrics scraping.
- **Grafana** for dashboards.
- **Loki** for log aggregation.
- **Tempo / OpenTelemetry (planned)** for distributed tracing.

All containers emit structured logs (`jsonl`) tagged with service, version, and trace ID. These can be viewed via Grafana dashboards for health and anomaly detection.

---

### 21.8 Scaling & Performance
- Modules can scale horizontally via multiple replicas (e.g., `tali-learning` for sandbox parallelism).
- Redis and Neo4j support cluster mode for larger installations.
- Grafana and Prometheus scale vertically with minimal tuning.
- Kafka (if enabled) supports distributed replay for ML event analytics.

**Performance Considerations:**
- M-series chips perform best in hybrid mode (native core + containerized data stack).
- For x86 servers, full Docker mode provides simpler scaling and orchestration.

---

### 21.9 Health, Updates & Lifecycle
- All services expose `/actuator/health` endpoints.
- Updates are versioned semantically (e.g., `v0.5.2` → minor feature → automatic migration).
- **Rolling update strategy:** containers are replaced one at a time, preserving event state.
- **Schema migration coordination:** handled by `tali-dataservice` (see Section 10.8).

TALI can detect when a deployment is misconfigured or degraded, and propose corrections via the Messenger I/O interface.

---

### 21.10 Security & Access Control
- All containers run under non-root users.
- Environment secrets mounted as Docker secrets.
- TLS enforced for internal MQTT and API calls.
- Optional mTLS for inter-service authentication.
- Admin access requires both resident verification and system key.

---

### 21.11 Future Enhancements
- Kubernetes Helm chart for large-scale deployments.
- Automated hardware profiling (AI-assisted optimal mode suggestion).
- Snapshot migration between Docker and standalone environments.

---

*(Section 21 defines how TALI deploys across containerized, hybrid, and standalone environments — balancing performance, privacy, and observability through flexible configuration and self-healing runtime management.)*

