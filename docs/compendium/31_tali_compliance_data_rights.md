# TALI Design Hub — Section 30: Compliance & Data Rights

## 31. Compliance & Data Rights
> Defines TALI’s privacy framework, data retention rules, and compliance model — including the Personally Identifiable Information (PII) Redaction Pipeline that ensures ethical and lawful handling of resident data.

---

### 31.1 Overview

TALI processes sensitive personal and behavioural data as part of its contextual learning system. This section establishes how TALI upholds data protection laws (such as UK GDPR), enforces user rights, and ensures no personal data is retained or transmitted unnecessarily.

TALI’s compliance design emphasizes **transparency, minimalism, and control** — giving residents full visibility into what is collected, how it is used, and when it is destroyed.

---

### 31.2 Data Classification Framework

| Category | Description | Examples |
|-----------|-------------|-----------|
| **Operational Data** | Required for automation and analytics. | Temperature readings, light states, MQTT events. |
| **Behavioural Data** | Used for learning and pattern recognition. | Room occupancy, activity sequences, automation triggers. |
| **PII (Personally Identifiable Information)** | Can identify a resident directly or indirectly. | Names, voice embeddings, device ownership metadata. |
| **Derived Data** | Generated from internal analysis. | Predictive schedules, comfort profiles, model weights. |

Each category has its own storage policy, encryption requirements, and retention limits.

---

### 31.3 The PII Redaction Pipeline

The **PII Redaction Pipeline** ensures that any personal identifiers are anonymized, pseudonymized, or removed before data is used in learning, analytics, or optional cloud operations.

#### 31.3.1 Pipeline Stages

1. **Capture Phase:** Raw data enters the system tagged with its classification.  
2. **Redaction Stage:** The PII pipeline automatically detects and masks personal data (e.g., replacing `Adam` with `Resident_A01`).  
3. **Normalization Stage:** Data is stripped of unique tokens (MACs, serials) and replaced with UUIDs.  
4. **Storage Phase:** Redacted and normalized data is committed to analytical stores (DuckDB, Neo4j, InfluxDB).  
5. **Review & Logging:** All redactions are logged in the internal audit ledger for traceability.

#### 31.3.2 Supported Redaction Domains
| Data Type | Redaction Method |
|------------|------------------|
| **Text Logs** | Named entity recognition removes names, addresses, phone numbers. |
| **Voice Data** | Speaker embeddings stored separately from transcripts. No raw voice retained. |
| **MQTT Payloads** | Device IDs and MACs hashed with a rotating salt. |
| **HA Service Calls** | Context fields containing user IDs replaced with system tokens. |
| **LLM Prompts** | Context scrubbed before transmission; cloud calls receive only redacted JSON. |

#### 31.3.3 Local-First Enforcement

All redaction occurs **locally within TALI Core** before any optional cloud interaction.  
No unredacted personal or behavioural data is sent to external endpoints.  
Even when cloud LLMs are used, all PII is replaced with neutral descriptors or unique placeholders.

---

### 31.4 Resident Rights & Data Control

Residents can inspect, export, or delete their personal data at any time through TALI’s data management interface.

| Right | Implementation |
|--------|----------------|
| **Access** | REST or Messenger command: `export my data`. Produces a redacted JSON package. |
| **Rectification** | Residents may correct mislabelled or inaccurate data via persona dialogue. |
| **Erasure (Right to be Forgotten)** | Executes full purge of resident-linked data, embeddings, and logs. |
| **Restriction** | Temporarily halts learning for a specific resident. |
| **Objection** | Residents can opt out of analytics or reinforcement training. |
| **Portability** | Exports data in standard JSON or CSV for external review. |

All actions are logged and require confirmation via admin or authenticated resident identity.

---

### 31.5 Retention & Expiry Policies

| Data Type | Retention Period | Expiry Action |
|------------|------------------|---------------|
| **Operational Data** | 90 days | Aggregated into anonymized summaries. |
| **Behavioural Data** | 180 days | Rolled into long-term trend models, raw data purged. |
| **Voice Data** | 7 days (embeddings excluded) | Raw audio deleted after transcription validation. |
| **Audit Logs** | 1 year | Archived to MinIO, encrypted, immutable. |
| **LLM Interactions** | 30 days | Sanitized context retained for debugging only. |

---

### 31.6 Encryption & Storage Controls

- All databases use AES-256 at-rest encryption.  
- Communication between modules secured with TLS 1.3 and mutual authentication.  
- Redis in-memory encryption enabled for sensitive keys.  
- Neo4j relationship-level access control used for privacy zones.  
- MinIO object store uses per-tenant encryption keys rotated every 90 days.  

---

### 31.7 Compliance Governance

| Policy Area | Enforcement Mechanism |
|--------------|----------------------|
| **Data Minimization** | Collect only what is necessary for function. |
| **Purpose Limitation** | Learning models cannot repurpose personal data for unrelated tasks. |
| **Transparency** | Residents notified when data is used in analytics or learning. |
| **Integrity & Confidentiality** | Enforced via security sandbox and role-based access. |
| **Accountability** | All compliance actions recorded in immutable audit log. |

---

### 31.8 Audit & Verification

- Quarterly internal privacy audits validate adherence to retention and redaction policies.  
- Logs compared against PII detection coverage metrics.  
- Optional third-party compliance check compatible with UK GDPR, ISO/IEC 27001, and OWASP IoT Top 10.  

---

### 31.9 Future Compliance Enhancements

- Expand redaction models to support context-based anonymization.  
- Resident dashboard for real-time privacy status visualization.  
- Federated privacy ledger for multi-home installations.  
- Automated compliance verification during CI/CD pipelines.  

---

*(Section 31 defines TALI’s complete compliance and data protection framework — detailing the PII Redaction Pipeline, resident rights, and enforcement policies to ensure lawful, transparent, and ethical handling of all household data.)*