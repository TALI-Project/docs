# TALI Design Hub — Section 31: Configuration Schema Reference

## 28. Configuration Schema Reference
> Provides an overview of the evolving configuration structure used by TALI — detailing the conceptual layout of key YAML/JSON configuration files while noting that schema definitions remain provisional and subject to change.

---

### 28.1 Overview

TALI’s configuration layer governs system behaviour, data connections, and resident personalization. Config files are declarative (YAML or JSON), human-readable, and designed for live reloading where possible.

All schema examples in this section represent **conceptual drafts**, not final implementations. The configuration system will mature in parallel with development and may evolve into versioned schema files managed by the Configuration Service.

---

### 28.2 Core Configuration Domains

| Domain | Purpose |
|---------|----------|
| **core.yaml** | Defines runtime identity, environment variables, and service discovery. |
| **database.yaml** | Contains connection strings and migration settings for PostgreSQL, Redis, Neo4j, and MinIO. |
| **persona.yaml** | Establishes the base persona traits and resident overlays. |
| **learning.yaml** | Controls sandbox parameters, reinforcement logic, and mutation policies. |
| **voice.yaml** | Configures room microphones, TTS engines, and wake-word settings. |
| **mqtt.yaml** | Manages broker credentials, topics, and security settings. |
| **policy.yaml** | Stores ethics, alignment, and behavioural rules. |
| **facts.yaml** | Contains immutable resident-defined operational facts (e.g., heating constraints). |
| **dashboard.yaml** | Defines metrics visibility, Grafana endpoints, and alerting thresholds. |

---

### 28.3 Example: Core Configuration (Draft)

```yaml
core:
  instance_id: tali-001
  mode: standalone
  timezone: Europe/London
  log_level: INFO
  sandbox_threads: 8
  allow_cloud_offload: true
  telemetry:
    enabled: true
    prometheus_port: 9090
  storage:
    base_path: /data/tali
```

---

### 28.4 Example: Database Configuration (Draft)

```yaml
database:
  postgres:
    host: postgres
    port: 5432
    username: tali
    password: ${DB_PASSWORD}
    database: tali_core
  redis:
    host: redis
    port: 6379
    db: 0
  neo4j:
    uri: bolt://neo4j:7687
    username: neo4j
    password: ${NEO4J_PASSWORD}
  minio:
    endpoint: http://minio:9000
    access_key: tali
    secret_key: ${MINIO_SECRET}
    bucket: tali-artifacts
```

---

### 28.5 Example: Persona Configuration (Draft)

```yaml
persona:
  base_profile:
    tone: professional
    humour: 0.4
    empathy: 0.6
    assertiveness: 0.5
  residents:
    adam:
      humour: 0.5
      empathy: 0.4
      trust: 0.9
      escalation_curve: linear
    maddie:
      humour: 0.1
      empathy: 0.8
      trust: 0.7
      escalation_curve: logarithmic
```

---

### 28.6 Example: Learning Configuration (Draft)

```yaml
learning:
  sandbox:
    parallelism: 16
    reward_window: 10m
    mutation_rate: 0.05
    recombination_rate: 0.1
    convergence_threshold: 0.02
  storage:
    snapshots_enabled: true
    retention_days: 30
  policy:
    safe_mode_default: true
    exploration_factor: 0.2
```

---

### 28.7 Example: Voice Configuration (Draft)

```yaml
audio:
  rooms:
    living_room:
      mic_id: esp32_living_01
      wake_word: enabled
      vad: edge
      echo_cancel: true
    kitchen:
      mic_id: esp32_kitchen_02
      wake_word: enabled
      vad: edge
      echo_cancel: true
  speakers:
    living_room:
      device: usb:LivingRoomSpeaker
    kitchen:
      device: usb:KitchenSpeaker
```

---

### 28.8 Example: MQTT Configuration (Draft)

```yaml
mqtt:
  broker: mqtt.local
  port: 8883
  username: tali
  password: ${MQTT_PASSWORD}
  tls: true
  topics:
    base: tali/
    voice_in: tali/voice/input
    voice_out: tali/voice/output
    state_events: tali/state/#
  qos:
    default: 1
    critical: 2
```

---

### 28.9 Dynamic Reloading & Version Control

- Configuration changes applied dynamically where possible using Spring Cloud Config patterns.  
- Persistent schema versions maintained in PostgreSQL `config_registry` table.  
- All configuration edits logged with timestamps, author, and diff summary.  
- Invalid configs automatically rejected and reverted to last known good state.  

---

### 28.10 Future Schema Plans

- Move toward versioned schema definitions with JSON Schema validation.  
- Integrate schema documentation generation into Grafana dashboards.  
- Develop resident-facing configuration editor (web + Messenger) with guided validation.  
- Add cryptographic signing for critical configuration bundles.  

---

*(Section 28 provides draft examples of TALI’s configuration schemas and file structure — illustrating how environments, personas, and policies will be declared while acknowledging that the exact implementation remains under active design and refinement.)*

