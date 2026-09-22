# MindFlow — VoiceCapture Entity Specification

**Transient voice input → structured task extraction**

---

## Purpose

Lowest-friction capture method. Voice notes are transcribed, summarized, and converted to task proposals — then **auto-deleted** per privacy policy.

---

## Fields (Verified)

| Field | Type | Description |
|-------|------|-------------|
| `transcript` | String | Raw STT output (temporary) |
| `summary` | String | LLM-generated intent summary |
| `extractedTasks` | Array<TaskDraft> | Structured task proposals |
| `autoDeletionMetadata` | Object | Retention policy, trigger, timestamp |
| `createdAt` | DateTime | Capture timestamp |
| `source` | Enum | `button` / `wake_word` / `shortcut` |
| `durationSeconds` | Number | Audio length |

---

## TaskDraft Structure (In extractedTasks)

| Field | Source | Notes |
|-------|--------|-------|
| `title` | LLM extraction | Required |
| `description` | LLM extraction | Optional |
| `energyTags` | Inferred from language | Defaults to UserPreference.energyDefaults |
| `dreadFlag` | Inferred (negative language) | Optional |
| `dueTime` | Extracted temporal refs | Optional |
| `projectRelationship` | Inferred from keywords | Optional |

---

## Auto-Deletion Policy (Verified Fields)

| Metadata Field | Purpose |
|----------------|---------|
| `maxAgeHours` | Default: 24; configurable |
| `deleteOnTaskCreate` | Default: true; delete after task creation |
| `deleteOnDismiss` | Default: true; delete if user dismisses |
| `retainSummaryOnly` | Default: false; if true, keep summary only |
| `deletionTriggeredAt` | Timestamp when deletion initiated |
| `deletedAt` | Timestamp when deletion completed |

---

## Workflow (Verified Architecture)

```
Voice capture activated
         │
         ▼
Speech-to-Text (STT) → transcript
         │
         ▼
LLM Intent Extraction → summary + extractedTasks[]
         │
         ▼
┌─────────┴─────────┐
▼                   ▼
Review UI         Auto-create (if
(SUGGEST/ASK_FIRST)  high confidence)
         │                   │
         └─────────┬─────────┘
                   ▼
Task(s) created with:
- source: "voice"
- VoiceCapture.ref linked
- energyTags inferred
- dreadFlag inferred
                   │
                   ▼
Auto-deletion timer starts
         │
         ▼
Deletion per autoDeletionMetadata
```

---

## Privacy Architecture

| Layer | Protection |
|-------|------------|
| **Transience** | Transcript is temporary; auto-deletion mandatory |
| **No persistent audio** | Only transcript/summary retained (configurable) |
| **User-controlled retention** | autoDeletionMetadata fully configurable |
| **No cross-user training** | Voice data never used for model training without consent |
| **Microphone permission** | Explicit per-session or persistent |

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

## Unverified (Requires Base44 Export)

- STT provider and model
- LLM provider, model, and extraction prompt
- Wake word implementation
- Audio buffer handling and encryption
- Exact auto-deletion implementation
- VoiceCapture → TaskEvent linkage
- Offline capability