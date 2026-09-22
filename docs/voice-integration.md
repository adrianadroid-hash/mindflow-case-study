# MindFlow — Voice Integration Design

**Voice capture → task extraction workflow**

---

## Design Principle

Voice is the **lowest-friction capture method** — especially when energy is low, hands are busy, or typing creates friction. Voice capture must be transient, privacy-respecting, and seamlessly convertible to structured tasks.

---

## VoiceCapture Entity (Verified)

| Field | Type | Purpose |
|-------|------|---------|
| `transcript` | String | Raw STT output (temporary) |
| `summary` | String | LLM-generated summary of intent |
| `extractedTasks` | Array<TaskDraft> | Structured task proposals |
| `autoDeletionMetadata` | Object | Retention policy, deletion trigger, timestamp |

---

## Workflow

```
User activates voice capture (button/shortcut/wake word)
         │
         ▼
┌─────────────────────────────────────┐
│         Speech-to-Text (STT)        │  → transcript
│   (provider: not verified;          │
│    Base44-managed or BYO)           │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│      Intent Extraction (LLM)        │  → summary + extractedTasks
│  (model: not verified;              │
│   prompt: structured extraction)    │
└──────────────────┬──────────────────┘
                   │
                   ▼
         ┌─────────┴─────────┐
         ▼                   ▼
   Review UI           Auto-create (if
   (SUGGEST/ASK_FIRST)   high confidence)
         │                   │
         └─────────┬─────────┘
                   ▼
         Task(s) created with:
         - source: "voice"
         - energy tags: inferred or default
         - dread flag: inferred from language
         - VoiceCapture.ref linked
                   │
                   ▼
         Auto-deletion timer starts
         (per autoDeletionMetadata)
```

---

## Extracted Task Structure

| Field | Source | Notes |
|-------|--------|-------|
| `title` | LLM extraction | Required |
| `description` | LLM extraction | Optional |
| `energyTags` | Inferred from language + context | Defaults to UserPreference.energyDefaults |
| `dreadFlag` | Inferred (negative language, hesitation) | Optional |
| `dueTime` | Extracted temporal references ("tomorrow", "Friday") | Optional |
| `projectRelationship` | Inferred from context/keywords | Optional |

---

## Privacy Architecture

| Layer | Protection |
|-------|------------|
| **Transience** | `transcript` is temporary; auto-deletion mandatory |
| **On-device STT option** | Not verified; would eliminate cloud audio transmission |
| **No persistent audio storage** | Only transcript/summary retained (configurable) |
| **User-controlled retention** | `autoDeletionMetadata` specifies: max age, trigger conditions |
| **No cross-user training** | Voice data never used for model training without explicit consent |
| **Microphone permission** | Explicit per-session or persistent (user choice) |

---

## Auto-Deletion Policy (Configurable)

| Parameter | Default | User Control |
|-----------|---------|--------------|
| `maxAgeHours` | 24 | Configurable in UserPreference |
| `deleteOnTaskCreate` | true | Toggle |
| `deleteOnDismiss` | true | Toggle |
| `retainSummaryOnly` | false | Toggle (retain summary, delete transcript) |

---

## Automation Integration

| Trigger | Action | Control Level |
|---------|--------|---------------|
| `voice_captured_appointment` | `calendar_suggestion` | SUGGEST |
| `voice_captured_task` (high confidence) | Auto-create task | SUGGEST (configurable) |
| `voice_captured_task` (low confidence) | Present for review | ASK_FIRST |

---

## Provider Independence (Design Goal)

| Component | Current (Base44) | Portable Design |
|-----------|------------------|-----------------|
| STT | Base44-managed | Pluggable: Whisper, Web Speech API, cloud APIs |
| LLM Extraction | Base44-managed | Pluggable: local LLM, API, structured prompts |
| Wake Word | Not verified | Pluggable: Porcupine, custom |

---

## Accessibility

| Feature | Status |
|---------|--------|
| **Voice-only operation** | Designed for (full task creation via voice) |
| **Wake word** | Not verified |
| **Offline STT** | Not verified (would require local model) |
| **Multi-language** | Not verified (depends on STT/LLM providers) |

---

## Unverified (Requires Base44 Export)

- STT provider and model
- LLM provider, model, and extraction prompt
- Wake word implementation
- Audio buffer handling and encryption
- Exact auto-deletion implementation
- VoiceCapture → TaskEvent linkage
- Offline capability