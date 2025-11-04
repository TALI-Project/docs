# TALI Design Hub — Section 18: Integration Layer & Ecosystem Bridges

## 9. Integration Layer & Ecosystem Bridges

> Defines how TALI observes, understands, and interacts with external ecosystems (Home Assistant, MQTT, Zigbee2MQTT, Matter, HTTP APIs) — including **packet-level command mirroring**, **HA service-call mimicry**, and **auto-tool synthesis** from observation.

---

### 9.1 Overview

TALI’s integration layer is designed for **local-first interoperability** with minimal assumptions about external stacks. Rather than hardcoding thousands of device drivers, TALI:

1. **Observes** traffic and behaviour, 2) **Infers** intent → action mappings, 3) **Validates** in sandbox, and 4) **Acts** via safe emitters or service calls. When patterns are stable, she can **synthesize her own tools** for reuse.

This learning is **per-home**: TALI adapts to the exact entities, topics, and service signatures present in each environment.

---

### 9.2 Bridge Adapters (at a glance)

| Adapter                       | Direction | Purpose                                                                                     |
| ----------------------------- | --------- | ------------------------------------------------------------------------------------------- |
| **HA Bridge**                 | In/Out    | Maps Home Assistant states, services, and entities into TALI events and actions.            |
| **MQTT Bridge**               | In/Out    | Observes and emits MQTT topics (EMQX/Mosquitto). Supports structured payload normalization. |
| **Zigbee2MQTT**               | In/Out    | Specialized observer for Z2M topic conventions; packet mirroring and discovery.             |
| **Matter / Thread (planned)** | In/Out    | Discovery and action via standard clusters; mirrors known attribute writes.                 |
| **HTTP / REST Adapters**      | In/Out    | Declarative HTTP integrations for local appliances and services.                            |

---

### 9.3 Observational Command Mirroring (Protocol Mimicry)

TALI can reproduce **exact control packets** or **service calls** she has previously observed producing a specific device outcome.

**Observation Phase**

- **MQTT:** Records `{topic, payload, qos, retain, headers, correlation_id}` for candidate control messages.
- **HA Service Calls:** Listens to HA’s `service_called` and related events/websocket frames, recording `{domain, service, data, target, context}` plus the resulting `state_changed` sequence.
- Associates messages/calls with **device/entity** and **intent** (e.g., `light.living_room: turn_on`).
- Builds a **causal map**: message/call → state change within Δt.

**Validation Phase (Sandbox)**

- Replays candidate MQTT packets against a **shadow bus** (loopback/test topic) and checks simulated outcomes.
- Replays candidate **HA service calls** against a **stubbed HA endpoint** or dry-run mode to verify parameters and side-effects.
- Adds guardrails: idempotence check, dedupe keys, loop prevention (no echo replays), and HA context scoping.

**Actuation Phase**

- Emits via the **Actuator Gate** only when safety policy passes (rate limits, time-of-day rules, trust ≥ threshold).
- **MQTT:** packets are tagged with `x-tali-origin=mirror` and correlation IDs for attribution.
- **HA:** service calls carry `context.user_id=tali` (or equivalent) for clear attribution in HA logs.

**Benefits**

- Works even if one path is degraded (choose MQTT direct or HA service path at runtime).
- Zero-config support for many devices; **per-house learning** of service signatures and entity quirks.
- Latency and reliability improvements while preserving full auditability in HA.

---

### 9.4 Auto‑Tool Synthesis from Observation ("Self‑Made Tools")

When TALI consistently maps an observed control pattern to a stable effect, she can **compile** that pattern into a reusable, parameterized **Tool**.

**18.4.1 Tool Definition**

```yaml
tool:
  id: tool.light.toggle
  version: 0.2.0
  capability: "lighting.toggle"
  transport: mqtt
  emit:
    topic: zigbee2mqtt/living_room_light/set
    payload_template: {"state": "${state}"}  # params → payload
  params:
    state: { enum: ["ON","OFF"], required: true }
  safety:
    rate_limit_per_min: 10
    allowed_hours: "06:00-23:30"
    requires_trust: 0.7
  attribution:
    origin: "tali"
```

**HA Service Tool Example**

```yaml
tool:
  id: tool.climate.set_temperature
  version: 0.1.0
  capability: "climate.setpoint"
  transport: ha_service
  emit:
    domain: climate
    service: set_temperature
    data_template:
      entity_id: ${entity_id}
      temperature: ${temperature}
  params:
    entity_id: { type: string, required: true }
    temperature: { type: number, min: 5, max: 30, required: true }
  safety:
    requires_trust: 0.8
    rate_limit_per_min: 6
    policy: ["max_setpoint_by_time"]
  attribution:
    origin: "tali"
```

- **Transport binding** (`mqtt`/`ha_service`/`http`) and **payload/service template** come from learned observations.
- **Params** are inferred from varying fields in observed payloads or HA service `data`.
- **Safety** and **trust gates** are attached automatically from policy.

**18.4.2 Synthesis Pipeline**

```
Observe packets & service calls → Cluster similar intents → Infer params & template → Generate Tool → Sandbox tests → Register → Use
```

- Clusters by topic/domain+service + effect (e.g., light state changed within 500ms).
- Deduces which fields are variable (e.g., `state`, `brightness`, `temperature`).
- Generates a minimal, declarative tool spec and tests it in isolation.
- On success, publishes to the **Capability Registry**.

**18.4.3 Capability Registry**

- Stores **tool specs** with versions, provenance, and test proofs.
- Exposes a stable **capability interface** to TALI Core and Persona (e.g., `lighting.set(level)`), decoupled from protocol details.
- Supports **deprecation** and **rollback** if a tool starts failing.

**18.4.4 Governance & Safety**

- Tools are **read-only to residents** by default and created **automatically by TALI**.
- Residents can view and optionally **approve** tools that control sensitive devices.
- Failing tools are quarantined; Core falls back to HA services.

---

### 9.5 Dual Path Control: HA Services vs Direct Packets

TALI can choose control path per context:

- **Primary:** Home Assistant service call (audited, high-level).
- **Fast Path:** Direct packet mirroring/tool emission when latency matters or HA is down. Selection is mediated by **confidence**, **policy**, and **health** signals.

---

### 9.6 Topic, Service & Payload Normalization

- Maintains dictionaries of known **MQTT topic shapes** (Z2M, Tasmota, Shelly, ESPHome).
- Maintains dictionaries of **HA service signatures** per domain (e.g., `light.turn_on`, `climate.set_temperature`).
- Auto-converts common fields (e.g., `on`→`ON`, `brightness_pct`→`brightness`).
- Applies **payload signing** or device-key headers when supported by the bridge.

---

### 9.7 Safety, Attribution & Loop Prevention

- All outbound packets and service calls are attributed (`x-tali-origin`, HA `context.user_id=tali`) and traced.
- **Echo suppression:** ignores messages that originate from TALI to avoid re-trigger.
- **Rate limits + circuit breakers** per topic/service/device.
- **Policy Engine** enforces domain rules (e.g., max setpoint, quiet hours, occupant presence requirements).

---

### 9.8 Examples

**Example A — Light Toggle (Z2M)**

- Observed: `zigbee2mqtt/living_room_light/set {"state":"ON"}` → entity turns on.
- Tool: `tool.light.toggle(state)` emits to same topic with templated payload.

**Example B — Thermostat Setpoint (HA Service)**

- Observed: `climate.set_temperature {entity_id: climate.lounge, temperature: 21}` via HA.
- Tool: `tool.climate.set(temperature)` chooses HA path; if HA down, fallback to known device payload.

**Example C — Scene Activation (HA Service)**

- Observed: `scene.turn_on {entity_id: scene.movie_night}`.
- Tool: `tool.scene.activate(id)` emits the matching HA service call with attribution.

---

### 9.9 Future Enhancements

- Learned **bi-directional** tools (emit + confirm via state ack patterns).
- Matter cluster auto‑discovery → tool synthesis via standard attributes.
- Confidence‑weighted path selection (HA vs direct) with online learning.

---

*(Section 9 now explicitly covers **HA service-call mimicry** alongside MQTT packet mirroring, and formalizes how TALI synthesizes reusable tools from both paths — all under strict safety, attribution, and per‑home learning.)*

