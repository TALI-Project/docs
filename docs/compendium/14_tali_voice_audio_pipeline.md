# TALI Design Hub — Section 7: Voice & Audio Pipeline

## 14. Voice & Audio Pipeline

> Defines how TALI processes, interprets, and produces voice-based interactions — from microphone input to spoken output.

### 14.1 Overview

The **Voice & Audio Pipeline** enables natural, real-time verbal interaction with TALI. It supports both passive wake-word detection and active push-to-talk modes, ensuring reliable voice capture, recognition, and playback across multiple rooms.

TALI’s audio system operates through two primary services:

- `` — Handles wake-word detection, voice activity detection (VAD), automatic speech recognition (ASR), and speaker identification.
- `` — Interprets speech transcripts, resolves identity, and routes responses through the Persona Engine and TTS output layer.

---

### 14.2 Audio Data Flow

```
[Microphone Input] → [VAD & Wake Word] → [ASR Engine] → [Speaker-ID] → [Intent Extraction (LLM)] → [Persona Engine] → [TTS Output]
```

Each stage communicates via MQTT topics and JSON payloads, maintaining low-latency asynchronous flow.

---

### 14.3 Input Processing

#### 14.3.1 Wake Word & VAD

- **Wake Word Models:** Porcupine, OpenWakeWord, or custom Whisper-based trigger models.
- **Modes:**
  - *Passive*: Always listening for the configured wake phrase.
  - *Active*: Push-to-talk (PTT) for privacy-sensitive environments.
- **VAD:** WebRTC-based detection suppresses noise and filters silence.

#### 14.3.2 ASR (Automatic Speech Recognition)

| Mode                 | Engine                                                                 | Description                                         |
| -------------------- | ---------------------------------------------------------------------- | --------------------------------------------------- |
| **Local (Default)**  | Whisper.cpp / Vosk                                                     | Fully offline transcription, low-latency streaming. |
| **Cloud (Optional)** | Google STT / Deepgram                                                  | Used only when local models fail or are disabled.   |
| **Output Format**    | `{text, confidence, language}` via MQTT topic `tali/voice/<room>/stt`. |                                                     |

#### 14.3.3 Speaker Identification

- Uses **voice embeddings** to recognize registered residents.
- Supports per-room adaptation for different acoustic profiles.
- Output includes `{person, confidence}` via `tali/voice/<room>/speaker`.
- Combines with device metadata and presence sensors for identity resolution.

---

### 14.4 Processing Pipeline Details

| Stage | Component               | Role                                                            |
| ----- | ----------------------- | --------------------------------------------------------------- |
| 1     | **Audio Input Capture** | Collects raw PCM stream from mic array.                         |
| 2     | **Preprocessing**       | Noise reduction, echo cancellation, normalization.              |
| 3     | **Wake Word Detector**  | Triggers TALI’s attention pipeline.                             |
| 4     | **ASR Engine**          | Converts speech to text for downstream LLM interpretation.      |
| 5     | **Speaker-ID**          | Matches voice embeddings to known profiles.                     |
| 6     | **Core Router**         | Sends recognized text to Persona Engine with resolved identity. |
| 7     | **LLM Processing**      | Derives intent and response.                                    |
| 8     | **TTS Synthesis**       | Converts text output to natural audio stream.                   |
| 9     | **Playback Control**    | Routes output to the nearest or most relevant speaker device.   |

---

### 14.5 Output Generation

#### 14.5.1 Text-to-Speech (TTS)

| Mode                 | Engine                                                              | Description                                            |
| -------------------- | ------------------------------------------------------------------- | ------------------------------------------------------ |
| **Local (Default)**  | Piper / Silero / Coqui-TTS                                          | On-device speech synthesis using TALI’s voice profile. |
| **Cloud (Optional)** | ElevenLabs / Azure Neural                                           | Optional fallback for high-fidelity synthesis.         |
| **Output Format**    | PCM or WAV audio stream sent to MQTT topic `tali/voice/<room>/tts`. |                                                        |

#### 14.5.2 Voice Profiles

- Derived from **Voice Character Profile** in Section 5.
- Parameters include pitch, tempo, tone, and accent.
- Each profile is stored as a **JSON voice manifest**:

```json
{
  "name": "tali_default",
  "pitch": 190,
  "tempo": 0.85,
  "accent": "en-GB-RP-light",
  "texture": "light_husk",
  "tts_model": "piper:en_GB_female_mature"
}
```

- Profiles are hot-swappable and reloadable without downtime.

---

### 14.6 Multi-Room Coordination

- Each room microphone registers under `audio.rooms` configuration.
- Presence data from Bermuda/mmWave determines preferred speaker for output.
- Audio routing prioritizes room proximity, occupant identity, and current activity.
- If multiple occupants are present, TALI adapts tone to the highest-confidence identity or defaults to neutral.

**Example Configuration:**

```yaml
audio:
  rooms:
    living_room:
      mic: usb:LivingRoomArray
      wake_word: enabled
      vad: webrtc
      echo_cancel: true
    kitchen:
      mic: usb:KitchenArray
      wake_word: ptt_only
  speakers:
    enrollments:
      adam: {embeddings:5, min_conf:0.75}
      maddie: {embeddings:5, min_conf:0.75}
```

---

### 14.7 Latency & Quality Targets

| Stage                             | Target Latency | Notes                                     |
| --------------------------------- | -------------- | ----------------------------------------- |
| **Wake Word → VAD**               | <100ms         | Real-time response detection.             |
| **ASR Transcription**             | <400ms         | Streaming mode for quick turnaround.      |
| **Speaker-ID Resolution**         | <250ms         | Includes matching and confidence scoring. |
| **TTS Synthesis**                 | <300ms         | Depends on model complexity.              |
| **End-to-End (Query → Response)** | <1.2s          | Goal for natural conversation cadence.    |

---

### 14.8 Security & Privacy

- Raw audio is **never stored**; only transcripts and anonymized embeddings are logged.
- Wake-word segments are ephemeral and discarded after activation.
- Speaker embeddings are encrypted and locally stored.
- Cloud-based services are disabled by default and require explicit consent.
- Audio data pipelines use secure, in-network MQTT with TLS.

---

### 14.9 Future Enhancements

- Integration of **emotion detection** (tone, stress, sentiment) for richer interactions.
- Support for **multi-language TTS/ASR models**.
- Dynamic speech rate and tone adaptation based on resident profile.
- Optional **LLM-driven rephrasing layer** to vary prosody and linguistic expression.

---

*(Section 14 defines the full voice and audio lifecycle for TALI — from wake word to multi-room speech output, optimized for low latency, privacy, and contextual intelligence.)*

