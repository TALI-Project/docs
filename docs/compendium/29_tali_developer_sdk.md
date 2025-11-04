# TALI Design Hub — Section 15: Java AppDaemon Layer (Developer Apps)

## 29. Java AppDaemon Layer (Developer Apps)
> A lightweight, **optional** Java-based app layer for power users to add bespoke home features. TALI remains **self-learning and self-running**; apps are for edge cases or advanced behaviours that TALI hasn’t learned yet.

---

### 29.1 Purpose & Positioning
- **Augment, don’t replace.** Apps complement TALI’s autonomous intelligence; they cannot disable or override core learning loops.
- **Local-first & private.** All code runs locally under strict isolation and least-privilege.
- **Simple mental model.** Think “small Java classes that subscribe to events and occasionally call actions,” not a plugin marketplace.

---

### 29.2 Architecture (Minimal, Sandboxed)
```
[App .jar] → [App Loader (ClassLoader)] → [App API Facade]
                                  │
                                  ├── read: Context/Graph (Neo4j), State (Postgres → cache)
                                  ├── subscribe: Events (Section 9)
                                  └── act: HA/MQTT services via Actuator Gate
```
- **Isolation:** Each app is loaded in its own **classloader** with CPU & memory quotas. No direct filesystem or network by default.
- **Capabilities:** Fine-grained tokens allow **read** (context/state), **subscribe** (events), and **act** (limited actions).
- **Mediation:** All actions are routed through the **Actuator Gate** which applies safety policies, rate limits, and attribution.

---

### 29.3 Developer App Model
- **Packaging:** A single JAR with a tiny manifest and one entrypoint class.
- **Lifecycle:** `init(ctx) → start() → heartbeat() → stop(reason)`
- **Registration:** Apps declare what they **observe** and what they may **act** on.
- **Conflict-free:** If an app collides with TALI’s learned automation, the core wins; the app receives a callback explaining the suppression.

**Manifest (YAML) Example**
```yaml
app:
  id: com.example.CurtainSaver
  name: Curtain Saver
  version: 0.1.0
  entrypoint: com.example.CurtainSaver
  observe:
    events:
      - tali.messenger.message.received
      - ha.entity.changed: cover.living_room_*
  act:
    services:
      - ha.service: cover.close_cover
      - mqtt.topic: zigbee2mqtt/living_room/cover/set
  limits:
    cpu_ms_per_min: 250
    heap_mb: 64
```

**Entrypoint Skeleton**
```java
public final class CurtainSaver implements TaliApp {
  private AppContext ctx;
  @Override public void init(AppContext ctx) { this.ctx = ctx; }
  @Override public void start() {
    ctx.events().subscribe("ha.entity.changed", ev -> onEntity(ev));
  }
  private void onEntity(EntityChanged ev) {
    if (ev.entityId().startsWith("cover.living_room") && ev.newState().is("open") && ctx.time().isAfter("22:00")) {
      if (ctx.policy().canAct("ha.service:cover.close_cover")) {
        ctx.actions().ha().call("cover.close_cover", Map.of("entity_id", ev.entityId()));
        ctx.log().info("Closed {} after 22:00 per app rule", ev.entityId());
      }
    }
  }
  @Override public void stop(String reason) { ctx.log().info("Stopped: {}", reason); }
}
```

---

### 29.4 App API Surface (Narrow & Stable)
| Area | API Facade | Notes |
|------|------------|-------|
| **Events** | `ctx.events().subscribe(ns, cls, handler)` | Namespaces per Section 9 (e.g., `ha.entity.changed`, `tali.learning.*`). |
| **State/Context** | `ctx.state().entity(id)`, `ctx.graph().neighbors(node)` | Read-only convenience over Postgres/Redis/Neo4j snapshots. |
| **Actions** | `ctx.actions().ha().call(service, params)` / `ctx.actions().mqtt().publish(topic, payload)` | Routed through Actuator Gate with safety checks + attribution. |
| **Time & Schedules** | `ctx.time().cron(expr, runnable)` | Lightweight scheduler with jitter controls. |
| **Logging & Metrics** | `ctx.log().info(..)`, `ctx.metrics().counter("app.xyz").inc()` | Emitted under `tali.analytics.plugin.*` with app ID tags. |

> No direct DB connections, sockets, or filesystem paths — all via facades.

---

### 29.5 Safety & Governance
- **Least privilege:** Capabilities are explicit; apps can’t escalate.
- **Quotas & circuit breakers:** CPU/heap/time limits, action rate caps, automatic half-open probes after failures.
- **Policy Engine:** Enforces domain rules (e.g., “climate setpoint max 24°C after 22:00”).
- **Attribution:** Every action is logged with `initiator=app:<id>` and visible in Grafana & audit logs.
- **Kill switch:** Residents can disable any app instantly; TALI may quarantine misbehaving apps.

---

### 29.6 Installation & Management
- **Apps Folder:** Drop-in JARs under `/apps/` (Compose volume or local folder).
- **Hot Reload:** App Loader detects changes and restarts apps safely.
- **Status & Logs:** `tali-core` exposes an **Apps** page: state, health, quotas, recent actions.
- **Updates:** Managed like any other artifact (Section 14 snapshots & rollbacks).

---

### 29.7 Observability
- Per-app metrics: CPU ms, heap, event throughput, action success/error.
- Structured logs with app ID and correlation IDs.
- Optional debug trace mode per app with time-limited sampling.

---

### 29.8 Design Principles (to keep autonomy intact)
1. **Core precedence:** Learned automations take priority; apps can propose, not dictate.
2. **Explainability:** If an app acts, the reason and rule are displayed on request (Messenger/Voice).
3. **Non-interference:** Apps cannot inspect or modify internal learning weights.
4. **Transparency:** Residents can see which apps are installed, what they observe, and what they can do.

---

### 29.9 Examples of Good App Use-Cases
- Bridge niche devices not supported by HA yet.
- Temporary household rules (e.g., holiday lighting windows).
- Experimental routines you want to try before TALI learns them.
- Specialized context joins (e.g., graph-powered reminders tied to visitors).

---

### 29.10 Non-Goals
- No external marketplace or cloud distribution model.
- No unrestricted scripting runtimes.
- No override of TALI’s core autonomy or safety systems.

---

*(Section 29 now specifies the **Java AppDaemon layer**: a minimal, sandboxed way to add small, local Java apps — fully subordinate to TALI’s autonomous intelligence and safety model.)*

