# TALI Design Hub — Section 13: Security & Privacy Architecture

## 32. Security & Privacy Architecture
> Defines TALI’s approach to securing data, communications, and resident privacy — ensuring trust, transparency, and control across all operations.

---

### 32.1 Overview
TALI is designed with **privacy-by-default** and **security-in-depth** principles. Every action, message, and data flow within the ecosystem is governed by explicit access rules and encryption standards. The goal is not only to keep residents safe, but to make TALI *trustworthy* — both in practice and perception.

Security priorities:
1. Protect all personal and behavioural data from unauthorized access.
2. Maintain local-first operation with cloud fallback only under explicit consent.
3. Guarantee traceability of every automation or AI decision.
4. Enable resident control over what TALI stores, learns, and forgets.

---

### 32.2 Core Security Layers
| Layer | Scope | Example Mechanisms |
|--------|--------|--------------------|
| **Transport** | Encryption-in-transit | TLS 1.3, mTLS between internal services, secure MQTT connections |
| **Data-at-Rest** | Storage encryption | AES-256, encrypted volumes for Postgres and Redis persistence |
| **Access Control** | Identity & trust management | Role- and trust-level enforcement per resident or device |
| **Isolation** | Runtime compartmentalization | Docker namespaces, virtual threads, sandbox process limits |
| **Auditability** | Logging and event traceability | Structured JSON audit logs correlated with trace IDs |

---

### 32.3 Identity & Trust Management
TALI’s security model revolves around **resident identities** and **trust scores**.

| Concept | Description |
|----------|-------------|
| **Resident Identity** | Each person interacting with TALI has a unique identity, derived from linked accounts (e.g., Discord, HA presence, device metadata). |
| **Trust Score** | Numeric confidence metric (0.0–1.0) reflecting verification strength and behavioural reliability. |
| **Authentication Sources** | Discord OAuth, local presence detection, device fingerprints, voice identification. |
| **Authorization Enforcement** | Commands require minimum trust thresholds (e.g., 0.7 to execute device actions). |
| **Cross-Verification** | Multi-signal validation: presence + biometrics + linked account consistency. |

#### Example:
```
Adam (trust 0.93): may issue automation or learning commands.
Guest (trust 0.35): read-only access to room state and notifications.
```

---

### 32.4 Secure Communication
All inter-service and external communication channels are encrypted and verified.

| Channel | Protocol | Security Features |
|----------|-----------|-------------------|
| **MQTT / EMQX** | TLS 1.3, optional mTLS | Per-client certificates for each TALI service. |
| **Redis Streams / PubSub** | TLS tunnel | Used internally for low-latency, transient events. |
| **HTTP / REST APIs** | HTTPS + OAuth 2.1 | Used for dashboards and external integrations. |
| **WebSockets** | Encrypted channels | Authentication tokens scoped per session. |
| **Messenger I/O** | Discord OAuth2 + App tokens | Access restricted to whitelisted guilds and DMs. |

---

### 32.5 Data Governance
TALI’s approach to data is **transparent, user-owned, and auditable**.

#### 32.5.1 Data Classification
| Type | Examples | Retention | Storage |
|------|-----------|------------|----------|
| **Core Configuration** | Devices, rooms, HA mappings | Permanent | PostgreSQL |
| **Persona Data** | Tone weights, behaviour overlays | 6 months | PostgreSQL / Redis |
| **Learning Artifacts** | Sandbox results, reward matrices | Rotating snapshots | MinIO |
| **Telemetry & Metrics** | Sensor data, performance logs | 12 months | InfluxDB / TimescaleDB |
| **Voice & Text Logs** | Transcripts, commands | Ephemeral | Redis / MinIO (anonymized) |

#### 32.5.2 PII Handling
- Personally identifiable information is stored only locally.
- All external LLM or cloud integrations redact PII before transmission.
- TALI provides an API to **forget** residents or anonymize history.
- Logs referencing deleted residents are scrubbed through cascading retention policies.

---

### 32.6 Sandboxing & Execution Safety
TALI’s code execution and learning loops run inside **controlled sandboxes**:
- Strict CPU, memory, and execution-time limits.
- Filesystem and network isolation per sandbox.
- Read-only access to production data snapshots.
- Real-time monitoring via the Analytics module for abnormal resource use.

If a sandbox violates thresholds, it is suspended and its state quarantined for review.

---

### 32.7 LLM Privacy & Transparency
When cloud-based LLMs are used (e.g., for persona tone or schema diffs):
- All PII and identifiers are redacted.
- Queries and responses are logged for resident review.
- Residents are notified when requests leave the local environment.
- Local LLMs are always preferred where hardware allows.

---

### 32.8 Auditing & Compliance
- Every event or database mutation generates a **traceable audit record** linked to its originator.
- TALI Core maintains tamper-resistant logs (`tali_audit_log` in Postgres + Loki stream).
- Audit entries include: timestamp, initiator, affected subsystem, and confidence score.
- Compliance hooks support export to external SIEM or Splunk if desired.

---

### 32.9 Incident Response & Recovery
- The Security Kernel detects anomalies from the Analytics service.
- Critical triggers: repeated failed auth, unknown entity commands, abnormal query spikes.
- Automatic responses:
  - Throttle or suspend affected modules.
  - Notify resident(s) via Messenger I/O.
  - Generate signed forensic snapshot for review.
- Manual override accessible only to verified administrators.

---

### 32.10 Resident Control & Consent
TALI prioritizes resident empowerment:
- Each resident can view and modify what TALI knows about them.
- Explicit consent required for:
  - Cloud model usage.
  - Cross-resident behaviour correlation.
  - Long-term persona retention.
- Residents can issue:
  - `forget-me` (erase personal data)
  - `export-my-data` (download structured summary)
  - `pause-learning` (temporarily suspend learning loops)

All actions are confirmed by TALI and logged.

---

### 32.11 Future Enhancements
- Hardware-backed key store integration (e.g., TPM or Secure Enclave).
- Federated identity for multi-home environments.
- Differential privacy layer for analytics.
- AI-driven behavioural anomaly detection for security posture.

---

*(Section 32 defines TALI’s end-to-end security and privacy architecture — from identity and encryption to data governance, sandboxing, and resident control — ensuring trust and safety at every operational level.)*

