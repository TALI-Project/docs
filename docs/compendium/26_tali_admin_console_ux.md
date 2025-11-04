# TALI Design Hub — Section 32: Admin Console & UX

## 26. Admin Console & UX
> Defines the administrative and user experience philosophy for TALI — emphasizing that all interaction occurs via **voice and text interfaces**, with no dedicated graphical UI beyond existing integrated tool dashboards.

---

### 26.1 Overview

Unlike traditional automation systems, TALI is designed to be **interface-minimal**. There are **no plans for TALI to have a bespoke web, desktop, or mobile UI**. All resident and administrative interaction occurs through:
- **Voice** — natural dialogue with the Persona Engine.  
- **Text** — via integrated messaging platforms (e.g., Discord channels, direct messages).  

Any visual interfaces arise only from **tools that TALI integrates with**, such as Grafana for metrics, MinIO console for backups, and Home Assistant’s existing dashboards for device visualization.

---

### 26.2 Interaction Philosophy

TALI’s UX design follows the concept of an **ambient intelligence** — an assistant that exists within the environment, not behind a screen. This ensures:
- Reduced cognitive load for residents.  
- Seamless blending with existing smart home ecosystems.  
- Natural interaction via conversation rather than configuration panels.  

In TALI’s ecosystem, **the conversation is the interface.**

---

### 26.3 Available Interfaces

| Channel | Purpose | Example |
|----------|----------|----------|
| **Voice (ASR/TTS)** | Direct verbal interaction and feedback. | “TALI, what’s the temperature upstairs?” |
| **Messenger (Discord)** | Text-based commands, logs, and notifications. | `/status` → returns system uptime and load. |
| **Grafana Dashboard** | Observability metrics and analytics. | CPU, memory, and sandbox performance panels. |
| **Home Assistant UI** | Device state visualization (read-only). | Lights, sensors, and environmental metrics. |
| **Prometheus / Loki** | Developer-level debugging and performance logs. | Accessible via browser dashboards. |

---

### 26.4 Admin Access & Controls

Administrative functions are available exclusively through **authenticated text or voice commands** — no GUI configuration pages.  
Admins interact using structured commands:
```text
/tali admin backup create
/tali config reload
/tali service restart core
```

Voice equivalents are supported for key maintenance actions:
> “TALI, restart your learning service.”  
> “TALI, reload configuration.”  

All admin actions require **role-based authentication** and are logged in the audit trail.

---

### 26.5 Developer Interfaces

For advanced users or maintainers:
- **Grafana:** Performance, anomaly, and service dashboards.  
- **Loki:** Log search for behavioural debugging.  
- **Prometheus:** Metric scraping for real-time monitoring.  
- **API Endpoints:** Optional REST access for configuration or diagnostic queries.  

These are not designed as resident-facing UIs but as operational tools.

---

### 26.6 Accessibility in Voice/Text UX

Because TALI relies on voice and text as its only interaction layers, accessibility is built directly into those channels:
- Voice rate, pitch, and verbosity adjustable per resident.  
- Text messages formatted for clarity and contrast (per Section 28).  
- Optional auditory cues for confirmation or alerts.  
- Text summaries available for every voice dialogue.  

---

### 26.7 Guiding Design Principles

| Principle | Description |
|------------|-------------|
| **Minimal Surface Area** | No redundant interfaces — leverages existing observability tools. |
| **Conversational Transparency** | Residents can always ask TALI what she’s doing or why. |
| **Security by Simplicity** | Fewer UIs mean fewer attack surfaces. |
| **Unified Experience** | All interfaces (voice/text/tools) reflect the same context and data. |

---

### 26.8 Future Considerations

While no standalone UI is planned, possible future augmentations include:
- **Voice-Driven Grafana Integration:** Query metrics conversationally.  
- **Messenger Dashboards:** Graphical embeds (status cards, charts) within Discord.  
- **CLI Tooling (Developer Use):** Optional command-line companion for deep system introspection.  

These would not replace TALI’s voice-first model but rather extend existing channels for convenience.

---

*(Section 26 defines TALI’s Admin Console and UX philosophy — affirming that she will have **no dedicated user interface**, instead relying entirely on natural voice and text interaction, supported by the existing ecosystem of observability and management tools.)*