# MindFlow — Friction Taxonomy

**Friction patterns and reasons specification**

---

## Design Principle

Friction is not laziness. It is **structural resistance** between a task and the human's current capacity/context. Making friction explicit, detectable, and actionable transforms it from a source of shame into a source of system intelligence.

---

## 7 Friction Patterns (Detected from TaskEvent Stream)

| Pattern | Detection Logic | Signal |
|---------|----------------|--------|
| **Repeated Postponement** | Task rescheduled ≥ 3 times without progress | "This keeps getting moved" |
| **Never Started** | Task created > 7 days ago, status = todo, no TaskEvent.started | "I never even began" |
| **Exceeds Estimate** | Actual focus time > 2× estimated duration (from FocusSession) | "This took way longer than I thought" |
| **Repeatedly Opened** | Task viewed/opened ≥ 5 times without status change | "I keep looking at it but don't do it" |
| **Dread** | Task.dreadFlag = true + postponement/avoidance patterns | "I actively don't want to do this" |
| **Energy Mismatch** | Task.energyTags incompatible with recent DailyState (consistently) | "Wrong energy for this" |
| **Blocked Dependency** | Task has incomplete blocking task/reference; no progress path | "I'm waiting on something" |

---

## 9 Friction Reasons (Root Cause Classification)

When a FrictionPattern is detected, the system classifies the **reason**:

| Reason | Description | Typical Automation Response |
|--------|-------------|----------------------------|
| `too_vague` | Task lacks concrete next action; "do project X" vs "draft outline for X" | **Decomposition suggestion** (break down) |
| `too_large` | Task scope exceeds single-session capacity | **Decomposition suggestion** (break down) |
| `missing_info` | Required information not available (contacts, specs, decisions) | **Friction check** → identify missing piece |
| `emotionally_unpleasant` | Task triggers avoidance (difficult conversation, boring, scary) | **ASK_FIRST** automation; dread flag support |
| `requires_another_person` | Blocked on external dependency | **Calendar suggestion** (schedule follow-up) |
| `wrong_timing` | Task energy tags mismatch current capacity pattern | **Reschedule suggestion** (energy-aware) |
| `excessive_energy` | Task demands more energy than typically available | **Decomposition** + **energy tag review** |
| `not_important` | Task priority low relative to active goals | **Move to review** (archive/snooze) |
| `unknown` | Pattern detected but reason unclear | **Friction flag** for human review |

---

## Detection → Response Pipeline

```
TaskEvent Stream
       │
       ▼
┌──────────────────┐
│ Pattern Detector │  (runs on each TaskEvent + periodic batch)
│  (7 patterns)    │
└────────┬─────────┘
         │ detected
         ▼
┌──────────────────┐
│ Reason Classifier│  (infers 1 of 9 reasons from context)
│  (9 reasons)     │
└────────┬─────────┘
         │ classified
         ▼
┌──────────────────┐
│ FrictionPattern  │  (record created: pattern, reason, confidence, task ref)
│   Record         │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ AutomationRule   │  (trigger: friction_detection)
│   Evaluation     │
└────────┬─────────┘
         │ matched
         ▼
   ┌─────┴─────┐
   │           │
AUTO        SUGGEST/ASK_FIRST
   │           │
   ▼           ▼
Execute      Present to user
action       with context
```

---

## Automation Responses by Reason

| Reason | AUTO Response | SUGGEST Response | ASK_FIRST Response |
|--------|---------------|------------------|-------------------|
| `too_vague` | — | "Break this into 3 concrete steps?" | "Want me to create sub-tasks?" |
| `too_large` | — | "Split into sessions under 90 min?" | "Create decomposition now?" |
| `missing_info` | — | "Missing: [contact/spec/decision]" | "Add missing info field?" |
| `emotionally_unpleasant` | — | — | "This has dread flag. Defer? Decompose? Talk through?" |
| `requires_another_person` | — | "Schedule follow-up for [date]?" | "Create calendar event?" |
| `wrong_timing` | Reschedule to better energy window | "Better at [time]?" | "Move to [time]?" |
| `excessive_energy` | — | "Lower energy version available?" | "Create low-energy variant?" |
| `not_important` | Move to review (archive) | "Archive this?" | "Delete or keep?" |
| `unknown` | — | "Friction detected. Review?" | "What's blocking you?" |

---

## Confidence Scoring

Each FrictionPattern record includes a **confidence score** (0.0–1.0):

- **High (≥0.8):** Clear pattern (e.g., 5+ postponements, explicit dread flag)
- **Medium (0.5–0.79):** Probable pattern (e.g., 3 postponements, energy mismatch over 3 days)
- **Low (<0.5):** Weak signal (e.g., 2 postponements, single energy mismatch)

**Automation only triggers on High confidence** for AUTO level. SUGGEST/ASK_FIRST can trigger on Medium+.

---

## User Control

| Control | Mechanism |
|---------|-----------|
| **Disable pattern detection** | Per-pattern toggle in UserPreference |
| **Adjust sensitivity** | Confidence threshold per pattern |
| **Override reason** | User can reclassify FrictionPattern.reason |
| **Dismiss pattern** | Mark as "not friction" — trains detector |
| **Automation permissions** | Per-rule AUTO/SUGGEST/ASK_FIRST (UserPreference.automationPermissions) |

---

## Privacy Note

FrictionPattern and TaskEvent reveal **behavioral and emotional patterns** (dread, avoidance, procrastination). This is sensitive personal data requiring:
- RLS enforcement (user-only access)
- Full export/deletion rights
- No cross-user analytics without explicit consent
- Auto-deletion policies for TaskEvent (configurable retention)