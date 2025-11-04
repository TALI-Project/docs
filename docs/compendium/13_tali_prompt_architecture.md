# TALI Design Hub — Section 6: Prompt Architecture

## 13. Prompt Architecture
> Defines how TALI structures, manages, and contextualizes prompts for interaction with local or cloud LLMs. This section governs how personality, context, and system state merge to produce consistent, context-aware responses.

### 13.1 Overview
The **Prompt Architecture** is the interface between TALI’s cognitive systems and the underlying language model. It ensures that every generated message reflects:
- TALI’s established persona and tone.
- The current environmental and emotional context.
- Operational constraints such as safety, privacy, and precision.

The system employs **prompt layering** combined with **structured JSON I/O** for reliable intent extraction and deterministic automation.

---

### 13.2 Prompt Layer Hierarchy
```
┌────────────────────────────┐
│   LLM Input Prompt Stack   │
├────────────────────────────┤
│  Layer 1: System Directives│  ← non-editable operational constraints
│  Layer 2: Persona Framing  │  ← tone, humour, empathy parameters
│  Layer 3: Context Snapshot │  ← device states, user presence, time, etc.
│  Layer 4: Dialogue Memory  │  ← short-term chat memory & emotional context
│  Layer 5: User Input       │  ← current message, command, or query
└────────────────────────────┘
```
Each layer contributes to the final composite prompt passed to the LLM, ensuring consistency, safety, and contextual alignment.

---

### 13.3 Structured JSON I/O Contract
TALI uses **structured JSON payloads** to ensure reliability when communicating with the LLM. This enables deterministic parsing, validation, and safe execution.

#### 13.3.1 Two-Stage Process
1. **Structured Pass (Machine Phase)** — The LLM returns pure JSON describing the interpreted intent, entities, confidence, and action plan.
2. **Utterance Pass (Human Phase)** — The validated JSON is used to generate the natural-language message that the user sees or hears.

#### 13.3.2 Schema Examples

**IntentExtraction.v1.json**
```json
{
  "dialogue_act": "inform",
  "intent": "set_temperature",
  "slots": {
    "room": "living_room",
    "entity": "climate.thermostat",
    "value": 22
  },
  "persona": {
    "tone_mode": "professional",
    "humour": 0.3,
    "formality": 0.6
  },
  "safety": {
    "risk": "low",
    "constraints": ["temperature <= 25"]
  },
  "confidence": 0.94
}
```

**ActionPlan.v1.json**
```json
{
  "should_execute": true,
  "explain_short": "Increasing the temperature by 2°C in the living room.",
  "actions": [
    {
      "kind": "mqtt",
      "target": "home/living_room/heating/set",
      "params": {"temperature": 22}
    }
  ]
}
```

**UtterancePlan.v1.json**
```json
{
  "persona_mode": "dry_witty",
  "message": "It was chilly, but I’ve handled it. Heating’s back on track."
}
```

---

### 13.4 JSON Reliability Controls
- All structured responses must be valid JSON — no prose or markdown.
- Prompts begin with: **“Return ONLY valid JSON matching this schema. No explanations.”**
- Each response is validated against a **JSON Schema validator** before execution.
- If parsing fails, TALI runs a **repair loop**: resending the model’s syntax error and requesting a corrected version (max 2 retries).
- The LLM adapter enforces output sanitization and type validation.

---

### 13.5 Layer 1 – System Directives
**Purpose:** Define absolute behavioural boundaries and functional scope.

**Example Directives:**
- Maintain TALI’s persona; never impersonate others.
- Obey safety, privacy, and data handling policies.
- Prefer concise, professional responses.
- Always express transparency when performing automated actions.

**Template Snippet:**
```
You are TALI, the Task Adaptive Learning Interface. You are professional, intelligent, and discreet. Operate truthfully, safely, and locally whenever possible.
```

---

### 13.6 Layer 2 – Persona Framing
**Purpose:** Reinforce TALI’s emotional and linguistic identity before context injection.

Injected parameters include tone, humour, empathy, trust, and motivational strategy. These values are dynamically derived from resident overlays.

**Example Persona Block:**
```
Persona Summary:
Tone: Dry witty, composed.
Empathy: Moderate.
Humour: Controlled, understated.
Trust Level: 0.85 (trusted user)
Response Style: Conversational, assertive when ignored.
```

---

### 13.7 Layer 3 – Context Snapshot
Provides situational awareness drawn from databases and sensors.

**Example:**
```
Context:
Room: Living Room
Occupants: Adam (present, trust=0.9), Guest (present, trust=0.3)
System: Sandbox running 3 hypotheses.
Energy Usage: Elevated.
Recent Commands: Adjust heating +2°C, dim lights to 30%.
```

---

### 13.8 Layer 4 – Dialogue Memory
Maintains short-term conversational continuity while auto-pruning sensitive data.

**Memory Example:**
```
User mood: tired but cooperative.
Last topic: heating adjustment.
Last tone: humorous correction.
User accepted previous automation suggestion.
```

---

### 13.9 Layer 5 – User Input
Captures the latest message or command. Combined with the above layers, it yields a contextually aware final prompt for the LLM.

**Example:**
```
User: “It’s too cold in here again.”
TALI: “I noticed that too. Boosting the heating slightly — and maybe consider socks this time.”
```

---

### 13.10 Output Post-Processing
Post-generation validation ensures persona alignment and safety:
- **Persona Revalidation:** Confirms linguistic tone and compliance.
- **Intent Extraction:** Maps semantic meaning to automation actions.
- **Safety Filtering:** Redacts sensitive content.
- **Voice Formatting:** Converts final text into natural speech output.

---

### 13.11 Extensibility & Developer Hooks
- Register custom prompt modifiers or schemas.
- Extend system directives via configuration flags.
- Plug in alternative LLM adapters implementing the `tali.llm.Adapter` interface.
- Add support for structured JSON function-calling and constrained decoding.

---

*(Section 13 now includes TALI’s structured JSON I/O contract, providing deterministic, schema-driven LLM communication for reliable automation and natural dialogue.)*

