# TALI Design Hub — Section 33: Integration & Interoperability

## 25. Integration & Interoperability
> Defines how TALI connects, extends, and interoperates with external ecosystems while maintaining strong security boundaries and local-first control.

---

### 25.1 Overview

TALI is built to **interoperate rather than replace** existing smart home and data ecosystems. Her design embraces open standards (MQTT, REST, WebSocket, Protobuf) and modular adapters that enable deep integration with third-party platforms — while always keeping **TALI Core as the source of truth and local command authority**.

All integrations follow the same guiding rules:
- **Local-first** whenever possible.  
- **Permission-based** for all cross-boundary interactions.  
- **Format-agnostic**, using adapters for each protocol or API.  

---

### 25.2 Integration Tiers

| Tier | Description | Examples |
|------|--------------|-----------|
| **Tier 1: Native Integrations** | Built directly into TALI Core, always local. | Home Assistant, MQTT, Redis, Neo4j. |
| **Tier 2: Extended Connectors** | Enabled via configuration, may reach LAN/cloud APIs. | Discord, Prometheus, Grafana, EMQX, MinIO. |
| **Tier 3: Optional Cloud Offload** | Only used with explicit consent; all data PII-redacted. | Cloud LLMs (OpenAI, Anthropic), speech services. |
| **Tier 4: Research & Experimental** | Optional and sandboxed integrations under evaluation. | HuggingFace local pipelines, custom agent APIs. |

---

### 25.3 Core Integrations

#### 25.3.1 Home Assistant (Primary Host)
- TALI interfaces directly with Home Assistant through its **WebSocket API** and **MQTT network**.  
- All entity states, device metadata, and service calls are synchronized.  
- TALI learns through observation of HA events and may replicate actions through HA’s service calls for consistency.  

#### 25.3.2 MQTT Layer
- MQTT is the real-time message backbone between TALI, sensors, and ESP devices.  
- Supports both Mosquitto and EMQX brokers with TLS mutual authentication.  
- TALI Core maintains topic-level access control for all internal and external publishers.  

#### 25.3.3 Grafana / Prometheus / Loki
- Observability stack integration for logs, metrics, and analytics.  
- Prometheus scrapes internal TALI metrics via `/actuator/prometheus`.  
- Grafana used purely for visualization — no write access back to Core.  

#### 25.3.4 Discord Messenger Integration
- Acts as the primary **text I/O interface** for residents.  
- Supports both private messages and shared channels.  
- Uses slash commands for admin control and conversational context.  
- Identity and message attribution ensure each interaction is logged against the correct resident.  

#### 25.3.5 MinIO Object Store
- Stores all large artifacts — model snapshots, logs, and backups.  
- Serves as the single artifact registry shared across Docker services.  
- Optional replication to secondary storage for fault tolerance.  

---

### 25.4 Optional Cloud Services

When local inference or processing is unavailable, TALI can — with **explicit consent** — temporarily offload tasks to external services.  
Examples include:
- **Cloud LLMs** for complex natural language inference or weight recalibration.  
- **Cloud TTS/ASR** fallback if local models fail or lack coverage.  

Every outbound request passes through the **PII Redaction Pipeline** (see Section 30).  
Residents are notified when cloud resources are used:  
> “This request required external processing — all identifiers were removed.”

---

### 25.5 External Data & API Adapters

Each external integration implements a **typed adapter interface**:
```java
interface IntegrationAdapter<T> {
    void connect();
    boolean isConnected();
    T read();
    void write(T payload);
    void disconnect();
}
```
Adapters run in sandboxed containers or microthreads to prevent fault propagation.  

Adapters may be dynamically loaded via the Plugin API (see Section 15) and registered through configuration files under `/config/integrations/`.

---

### 25.6 Interoperability & Data Exchange

All cross-system communication uses standardized, versioned schemas:
- **JSON / Protobuf:** for structured data.  
- **NDJSON:** for log streaming.  
- **WebSocket / MQTT:** for event transmission.  

When receiving data from non-native systems, TALI automatically normalizes payloads into internal event objects (e.g. `TaliEvent`, `TaliEntityState`, `TaliServiceCall`).

---

### 25.7 Network Security Boundaries

- Integrations are grouped into **trust zones** (see Section 24).  
- All Tier 2+ connectors require signed configuration tokens.  
- Cloud-bound connectors must pass the Redaction Pipeline.  
- Periodic integrity checks ensure no external service maintains persistent access beyond its session.

---

### 25.8 Developer Integration Guidelines

For developers extending TALI:
- Use official APIs and adapters.  
- Avoid direct database or MQTT writes — go through TALI’s integration service.  
- Declare required scopes (read, write, event, admin).  
- Provide test stubs to validate integration safety before deployment.  

Future plans include a **Developer SDK** (Section 34) providing boilerplate for integration adapters, schema validation, and sandboxed plugin testing.

---

*(Section 25 defines how TALI interoperates with local ecosystems, observability tools, and optional cloud services — ensuring modular extensibility, strict data boundaries, and transparent, consent-based cross-system communication.)*