# TALI Design Hub — Section 28: Accessibility & Internationalization

## 17. Accessibility & Internationalization
> Defines the design considerations that ensure TALI is usable and inclusive for a wide range of residents, while clarifying that English is the sole supported language for the foreseeable future.

---

### 17.1 Overview

TALI aims to be inclusive and accessible to all residents regardless of ability or interaction preference. Accessibility is implemented across **voice**, **text**, and **visual** interfaces, ensuring that everyone in a household can engage with TALI effectively.

At present, **TALI operates exclusively in English**, including all voice recognition, natural language understanding, and text responses. Multi-language expansion is not planned in early development phases to maintain quality, stability, and performance.

---

### 17.2 Accessibility Principles

| Principle | Description |
|------------|-------------|
| **Equitable Use** | All features are usable without requiring visual, auditory, or fine motor precision. |
| **Perceptible Feedback** | Every action produces audible, verbal, or visual confirmation. |
| **Simple Interaction Paths** | Commands and dialogues avoid nested or ambiguous phrasing. |
| **Error Tolerance** | Misheard or mistyped inputs trigger graceful clarifications rather than failure. |
| **Consistency** | Behavioural and vocal responses remain predictable across contexts. |

---

### 17.3 Supported Input & Output Modes

| Mode | Accessibility Features |
|-------|------------------------|
| **Voice Interaction** | Adjustable speech rate and clarity; wake word or push-to-talk activation; replay of responses upon request. |
| **Text Interface (Messenger)** | Clear language, high contrast message embeds, emoji indicators for quick comprehension. |
| **Audible Alerts** | Context-aware volume adjustment; different tones for urgency vs. information. |
| **Dashboard / Visual Display** | Font scaling, minimal colour dependence, and accessible contrast ratios (WCAG 2.2 AA). |

---

### 17.4 Audio & Speech Accessibility

- TALI’s Text-to-Speech supports adjustable **rate**, **pitch**, and **volume**, configured per resident.  
- Optional **speech repetition** mode for users with auditory processing difficulties.  
- Wake-word sensitivity calibration allows for reduced accidental activation in noisy environments.  
- Microphone gain auto-tuning maintains accuracy for quieter voices or distant speech.  

---

### 17.5 Visual & Textual Accessibility

- Dashboard interfaces use a clean, responsive layout with scalable typography.  
- Status messages avoid colour-only signalling — icons and text provide redundant cues.  
- Messenger messages are short, descriptive, and use clear sentence structure.  
- Optional “concise mode” limits verbose explanations for users who prefer minimal responses.  

---

### 17.6 Cognitive & Interaction Accessibility

- Persona phrasing designed for clarity and predictability — no slang or idiomatic expressions.  
- Optional **simplified mode** for reduced conversational complexity (ideal for neurodiverse users).  
- Error prompts framed helpfully: “Did you mean…” instead of generic failure messages.  
- Configurable reminder frequency to support memory-related needs.  

---

### 17.7 Internationalization Statement

Although TALI’s codebase supports UTF‑8 and international data schemas, **no localization or multilingual voice models** are planned.  
All interface elements, training data, and LLM prompts are optimized for English to preserve coherence, low latency, and predictable behaviour.

Future consideration of localization may occur after full core stability, but only if language models and voice synthesis can meet equivalent ethical and performance standards.

---

### 17.8 Testing for Accessibility

Accessibility is validated via:
- Automated checks against WCAG 2.2 AA for dashboard and message UIs.  
- Resident testing with assistive devices (screen readers, hearing aids).  
- Persona dialogue audits for clarity and tone appropriateness.  
- Voice interaction latency tests to ensure near‑real‑time response consistency.  

---

*(Section 17 defines TALI’s accessibility standards and explicitly confirms that English is the only supported language — focusing on inclusivity, clarity, and consistency for all users within English-speaking households.)*

