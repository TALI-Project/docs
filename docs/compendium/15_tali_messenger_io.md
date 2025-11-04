# TALI Design Hub — Section 8: Messenger I/O (Text-Based Communication)

## 15. Messenger I/O (Non-Voice Interaction Layer)
> Defines how TALI communicates over text-based channels such as Discord, chat dashboards, or messaging integrations — supporting multimedia, contextual awareness, slash commands, and per-resident action attribution.

### 15.1 Overview
The **Messenger I/O module** enables TALI to interact naturally through text-based and mixed-media communication platforms. It mirrors the Voice & Audio Pipeline, applying the same Persona, Context, and LLM processing layers while routing output through textual, visual, or interactive channels.

Supported platforms and media types include:
- **Discord (core target)** — channels, threads, direct messages, reactions, embedded media, and slash commands.
- **Local web dashboard chat** — TALI’s built-in UI console with image and attachment support.
- **Extensible adapters** — future integrations with Telegram, Slack, Matrix, or SMS.
- **Media Formats:** Plain text, Markdown, embedded images, interactive buttons, structured JSON payloads, and slash command inputs.

---

### 15.2 Architecture Overview
```
[User Message / Media] → [Platform Adapter] → [Identity Resolver] → [Prompt Assembly Engine] → [LLM Response] → [Persona Formatter] → [Channel Dispatcher]
```
Each adapter conforms to the `tali.messenger.Adapter` interface, ensuring consistent structure and behaviour across messaging platforms.

---

### 15.3 Discord Integration
#### 15.3.1 Structure
| Channel Type | Description |
|---------------|-------------|
| **Guild Channel** | Shared household or group interaction; TALI attributes actions to specific users automatically. |
| **Direct Message (DM)** | Private, resident-specific context for personal routines or confidential communication. |
| **System Channel** | Logs, alerts, and system feedback from TALI Core. |

#### 15.3.2 Behaviour
- TALI respects channel scope — she never crosses context between guilds or DMs.
- In shared channels, TALI uniquely identifies which resident performed each action or issued each command.
- This identity mapping allows **per-resident persona updates**, ensuring learning and personality adjustments are personalized even in group spaces.
- Per-channel context maps to Home Assistant zones (e.g., living-room text channel ↔ room presence).
- Tone, humour, and phrasing dynamically align with the persona overlay of the identified user.
- Supports **multi-threaded conversations** with independent context retention.

#### 15.3.3 Example Exchange
```
Adam: “TALI, set the heating to 21.”
Maddie: “And make sure the kitchen lights are warm white.”
TALI: “Understood. Heating 21°C for Adam, kitchen lights updated for Maddie — excellent teamwork.”
```

---

### 15.4 Message Flow & Context
| Stage | Component | Role |
|--------|------------|------|
| 1 | **Adapter** | Converts platform events, messages, and media into normalized message objects. |
| 2 | **Identity Resolver** | Determines which resident performed the action using linked accounts, metadata, or context cues. |
| 3 | **Prompt Assembly Engine (PAE)** | Builds the layered prompt (System → Persona → Context → Memory → Input). |
| 4 | **LLM Processor** | Generates structured JSON response (IntentExtraction + ActionPlan). |
| 5 | **Persona Formatter** | Applies resident-specific overlay and tone. |
| 6 | **Dispatcher** | Sends response or action result (text, media, or control signal) to the originating channel. |

---

### 15.5 Resident Linking & Action Attribution
- Residents link messaging accounts (e.g., Discord) to their TALI identity.
- Each message or reaction event is attributed to its sender via **identity resolution** (account, device metadata, and contextual signals).
- Even in shared channels, TALI records **who did what** — updating:
  - **Persona models** (trust, humour tolerance, responsiveness).
  - **Learning weights** for behaviour adaptation.
  - **Interaction logs** for traceability.
- This allows TALI to maintain multi-resident awareness and evolving personalization simultaneously.

---

### 15.6 Message Intent, Media Handling & Action Routing
- Supports mixed input formats — text, images, emojis, structured commands, and attachments.
- Media (e.g., photos of devices or dashboards) can be analyzed locally or via LLM for context enhancement.
- Commands are parsed through the structured JSON LLM pass for intent and slot extraction.
- Validated **ActionPlans** are routed through the Core Integration Layer to MQTT, HA APIs, or system tasks.

**Example:**
```json
{
  "intent": "toggle_light",
  "slots": {"room": "living_room", "state": "off"},
  "confidence": 0.97,
  "initiator": "adam"
}
```
```
TALI: “Lights off — logged under Adam’s profile. Maddie, would you like similar ambient mode in the kitchen?”
```

---

### 15.6.1 Slash Commands
TALI registers a structured command tree in Discord for deterministic, low-latency interaction. These commands provide a schema-aligned alternative to free-form text and map directly to TALI’s intent/action framework.

| Command | Description | Example Invocation | Result |
|----------|--------------|--------------------|---------|
| `/status` | Query system or room status. | `/status room:kitchen` | “Kitchen lights off, window open.” |
| `/set` | Set a device state or parameter. | `/set entity:climate.living_room value:21` | “Heating set to 21°C.” |
| `/routine` | Trigger a learned automation. | `/routine morning` | Executes “morning routine”. |
| `/feedback` | Provide corrective feedback to TALI. | `/feedback that was too warm` | Adjusts comfort weighting for that resident. |

**Implementation Details:**
- Slash commands resolve via the **Messenger Adapter** → **Prompt Assembly Engine** → **Core Action Router**.
- Metadata (user ID, guild, channel) feeds directly into the **Identity Resolver**.
- Responses that contain personal or sensitive information are sent as **ephemeral messages** visible only to the initiator.
- Supports autocompletion and dynamic parameter loading (e.g., listing available rooms or devices).

**Security & Permissions:**
- Commands that modify device states require trust ≥ 0.7.
- Sensitive operations (e.g., `/shutdown`, `/reset`, `/model retrain`) require explicit confirmation or admin-level permission.

---

### 15.7 Context Persistence & Memory
- Each conversation thread holds ephemeral context, including recent intents, tone, and participants.
- Memory TTL: configurable (default 1h idle).
- Context tracks participants individually, so shared conversations update each persona independently.
- Thread memory links to HA events, presence data, and recent automations.

---

### 15.8 Security & Permissions
- Outbound messages use platform tokens stored securely in Docker secrets.
- Resident mapping prevents unauthorized control or impersonation.
- Device-affecting commands require trust ≥ 0.7.
- All messages and action logs include initiator attribution.
- Media uploads are sandboxed, scanned, and never forwarded externally without consent.

---

### 15.9 Developer Hooks
- Extend `tali.messenger.Adapter` for new messaging platforms or formats.
- Pre- and post-processing hooks for rich media handling (images, reactions, structured embeds, slash commands).
- Persona overlays can register stylistic filters for textual and visual output.
- Grafana Loki integration for cross-modal log tracing (voice ↔ text ↔ action).

---

### 15.10 Future Enhancements
- Real-time visual summarization (image + message context synthesis).
- Multi-platform conversation continuity (Discord ↔ Voice ↔ Dashboard).
- Media captioning and scene awareness using local LLMs.
- Federated learning of cross-user collaboration patterns in shared channels.

---

*(Section 15 now includes support for multimedia input, slash commands, and per-resident action attribution — ensuring TALI can interpret, respond, and learn from all forms of user interaction across text-based environments.)*

