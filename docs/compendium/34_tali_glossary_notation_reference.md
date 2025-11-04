# TALI Design Hub — Section 22: Glossary & Notation Reference (Updated)

## 36. Glossary & Notation Reference
> Defines all major terms, acronyms, and system references used throughout the TALI Design Hub. Updated to include all features and subsystems introduced up to Section 35.

---

### 36.1 Core Concepts

| Term | Definition |
|------|-------------|
| **TALI (Task Adaptive Learning Interface)** | The core AI system designed to observe, learn, and automate home environments through natural interaction and autonomous reasoning. |
| **TALI Core** | The central runtime of TALI responsible for orchestration, reasoning, learning, and integration with Home Assistant and databases. |
| **Sandbox** | A controlled environment where TALI tests automation variants, reinforcement loops, and evolutionary simulations safely. |
| **Learning Loop** | Continuous process of observation → hypothesis → simulation → validation → reinforcement. |
| **Persona Engine** | Module defining TALI’s behaviour, tone, and personality, adapting per resident while maintaining consistent ethics. |
| **Alignment Engine** | Ethical safeguard ensuring all proposed actions conform to household policies, resident consent, and moral guidelines. |
| **Policy Engine** | Framework enforcing operational rules, permissions, and ethical policies during runtime. |
| **Core Advisory State** | Restricted mode entered when ethical or behavioural misalignments are detected. |

---

### 36.2 Data & Intelligence

| Term | Definition |
|------|-------------|
| **PII Redaction Pipeline** | Multi-stage system that strips or anonymizes personal data before analytics or cloud communication. |
| **Reinforcement Learning** | Feedback-based optimization where actions are rewarded or penalized to refine automation logic. |
| **Evolutionary Learning** | Darwinian optimization using mutation and recombination to evolve superior logic branches. |
| **Reward System** | Evaluation mechanism scoring simulated or real-world outcomes against resident comfort and safety. |
| **Meta-Learning** | Higher-order framework where TALI learns *how to learn*, tuning its reinforcement parameters dynamically. |
| **Dreaming Mode** | Offline simulation mode for predictive or experimental automation training. |
| **Context Graph (Neo4j)** | Graph database mapping relationships between residents, rooms, devices, and contexts. |
| **DuckDB** | Embedded analytics engine for structured data queries and AI feature extraction. |
| **PostgreSQL (Source of Truth)** | Primary relational database storing persistent states, configurations, and audit logs. |
| **Redis (Cache)** | High-speed memory store used for transient states and real-time voting/learning data. |
| **MinIO** | Object storage for model snapshots, logs, and backups. |

---

### 36.3 Communication & Integration

| Term | Definition |
|------|-------------|
| **MQTT** | Lightweight messaging protocol used for device communication and real-time events. |
| **Home Assistant Integration** | Primary bridge for device discovery, state monitoring, and control through HA APIs. |
| **Discord Messenger I/O** | Text-based communication channel allowing conversation, command input, and event feedback. |
| **Voice I/O Pipeline** | Subsystem managing microphones, wake-word detection, ASR (speech-to-text), and TTS (text-to-speech). |
| **ESP Room Microphone** | ESP32-based edge device capturing local audio and performing wake/VAD before streaming to TALI. |
| **Grafana / Prometheus / Loki Stack** | Observability suite for metrics, logging, and analytics visualization. |
| **MinIO Console** | Web interface for managing stored backups and model artifacts. |
| **Integration Adapter** | Modular connector class handling communication with an external service (local or cloud). |
| **Cloud Offload** | Optional, consent-based delegation of inference tasks to external LLMs or APIs with PII redaction. |

---

### 36.4 Behaviour & Persona

| Term | Definition |
|------|-------------|
| **Base Persona** | Default behavioural archetype defining TALI’s tone (professional, witty, calm). |
| **Resident Overlay** | Individual adaptation layer adjusting humour, empathy, and assertiveness per user. |
| **Motivation Strategy** | Behavioural pattern for prompting residents (e.g., consequence-based, supportive). |
| **Escalation Curve** | Defines intensity growth across repeated prompts (linear, logarithmic). |
| **Tone Spectrum** | Range of emotional delivery styles from formal to empathetic. |
| **Voice Profile** | Configurable audio characteristics: pitch, timbre, accent, and pacing. |
| **Empathy Bounds** | Ethical constraints limiting emotional or persuasive extremes. |

---

### 36.5 System & Deployment

| Term | Definition |
|------|-------------|
| **Standalone Mode** | TALI runs natively on the host machine (e.g., Mac M-series) using local LLMs. |
| **Containerized Mode** | All services operate via Docker Compose for portability. |
| **Observation Stack** | Combination of Grafana, Prometheus, and Loki for telemetry and tracing. |
| **Safe Mode** | Automatic fallback state triggered by integrity or ethical violations. |
| **Runbook** | Documented procedure for disaster recovery or manual restoration. |
| **Hardware Profile** | Defined minimum and recommended specs for host systems. |
| **Trust Zones** | Logical network segmentation defining device and service privilege boundaries. |

---

### 36.6 Compliance & Privacy

| Term | Definition |
|------|-------------|
| **Resident Data Rights** | GDPR-aligned framework granting access, correction, deletion, and export capabilities. |
| **Anonymization Ledger** | Audit log recording every redaction and anonymization event. |
| **Retention Policy** | Time-bound rules for purging old operational or behavioural data. |
| **Encryption At Rest** | AES-256 protection for stored databases and backups. |
| **TLS 1.3 Mutual Auth** | Secure handshake protocol used between services. |
| **Ethical Audit Trail** | Immutable log of actions and persona responses for accountability. |

---

### 36.7 Research & Experimental Terms

| Term | Definition |
|------|-------------|
| **Research Sandbox** | Isolated environment for testing experimental algorithms and models. |
| **Evolutionary Lineage Tree** | Historical record of learning generations and mutations. |
| **Cross-TALI Federation** | Future opt-in network for anonymized knowledge sharing between instances. |
| **Tool Synthesis** | AI-assisted generation of new functional modules. |
| **Dreaming Mode** | Offline predictive simulations for automation refinement. |
| **Emotional Inference** | Context-aware tone adjustment based on conversation sentiment. |
| **Ethical Reflection Log** | Dataset of self-review entries after resident corrections or conflicts. |

---

### 36.8 Administrative & Development

| Term | Definition |
|------|-------------|
| **Admin Command** | Privileged instruction issued via text or voice (e.g., restart, backup). |
| **Configuration Schema** | Declarative YAML/JSON structure defining environment parameters. |
| **Dynamic Reload** | Live configuration reapplication without service restart. |
| **Integration SDK** | Developer toolkit for writing and validating third-party adapters. |
| **Persona Reload Endpoint** | API command to refresh personality configuration live. |
| **HITL (Human in the Loop)** | Manual review process allowing humans to approve or correct learning changes. |
| **Plugin API** | Controlled mechanism for loading external Java modules into TALI Core. |

---

### 36.9 Notation Conventions

| Symbol | Meaning |
|---------|----------|
| `/entity/id` | Reference path for Home Assistant or MQTT topic. |
| `${VAR}` | Environment variable substitution. |
| `<room>` | Placeholder for physical room identifier. |
| `tali/voice/<room>` | MQTT topic structure for per-room audio messages. |
| `→` | Indicates data or control flow direction. |
| `::` | Denotes namespace or module separation. |
| `#` | Comment indicator within configuration examples. |

---

### 36.10 Acronyms

| Acronym | Meaning |
|----------|----------|
| **ASR** | Automatic Speech Recognition |
| **TTS** | Text-to-Speech |
| **LLM** | Large Language Model |
| **VAD** | Voice Activity Detection |
| **HA** | Home Assistant |
| **HSM** | Hardware Security Module |
| **RPO/RTO** | Recovery Point / Time Objective |
| **SDK** | Software Development Kit |
| **API** | Application Programming Interface |
| **PII** | Personally Identifiable Information |
| **WCAG** | Web Content Accessibility Guidelines |
| **QoS** | Quality of Service (MQTT) |

---

*(Section 36 provides a unified, up-to-date glossary and notation reference for the complete TALI Design Hub — encompassing all architectural, behavioural, and experimental terminology.)*