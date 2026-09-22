# MindFlow — AutomationRule Entity Specification

**Trigger → Action mappings with human-control levels**

---

## Purpose

Defines the **automation logic**: when something happens (trigger), what the system does (action), and how much user consent is required (control level).

---

## Fields (Verified)

| Field | Type | Description |
|-------|------|-------------|
| `trigger` | Enum (10 values) | Event that fires the rule |
| `action` | Enum (11 values) | Operation to execute |
| `controlLevel` | Enum | `AUTO` / `SUGGEST` / `ASK_FIRST` |
| `enabled` | Boolean | Default: true |
| `conditions` | Object | Optional filter (e.g., `confidence > 0.8`) |
| `metadata` | Object | Action-specific parameters |

---

## 10 Triggers (Verified)

| Trigger ID | Fires When | Context |
|------------|------------|---------|
| `overdue_task` | Task.dueTime passed, status ≠ done | Task, DailyState |
| `postponed_task` | Task rescheduled (dueTime push or status) | Task, postponement count |
| `exceeded_focus_session` | FocusSession.actualDuration > planned × 1.5 | FocusSession, Task |
| `low_physical_state` | DailyState.physicalEnergy ≤ 3 | DailyState |
| `high_stress` | DailyState.stress ≥ 7 | DailyState |
| `voice_captured_appointment` | VoiceCapture extracts appointment | VoiceCapture, extracted data |
| `missed_completed_recurrence` | Recurring task missed or completed | Task, recurrenceTracking |
| `friction_detection` | FrictionPattern created (high confidence) | FrictionPattern, Task |
| `task_creation` | New Task created | Task, DailyState, UserPreference |
| `spiraling_task_overload` | ≥5 overdue/postponed tasks in 24h | Task aggregate, DailyState |

---

## 11 Actions (Verified)

| Action ID | Effect | Typical Control Level |
|-----------|--------|----------------------|
| `move_to_review` | Task.status → review | SUGGEST/ASK_FIRST |
| `friction_check` | Prompt structured diagnosis | SUGGEST |
| `hyperfocus_warning` | Notify: exceeded focus time | SUGGEST |
| `recovery_mode` | Activate recoveryMode | ASK_FIRST |
| `chaos_mode` | Activate chaosMode | ASK_FIRST |
| `minimum_survival_mode` | Filter to chaos/bedridden-compatible only | AUTO (safety) |
| `calendar_suggestion` | Propose calendar event | SUGGEST |
| `recurrence_scheduling` | Auto-schedule next recurrence | SUGGEST/ASK_FIRST |
| `friction_flag` | Add friction metadata to Task | AUTO (metadata) |
| `decomposition_suggestion` | Propose sub-task breakdown | SUGGEST |
| `reminder_noise_reduction` | Suppress low-priority reminders | AUTO (filtering) |

---

## Control Levels (Verified)

| Level | Behavior | User Consent |
|-------|----------|--------------|
| `AUTO` | Execute immediately | Pre-authorized via UserPreference |
| `SUGGEST` | Present recommendation; user accepts/declines | Implicit (user sees it) |
| `ASK_FIRST` | Request explicit confirmation | Explicit (user must confirm) |

---

## Trigger → Action Default Mapping

| Trigger | Primary Action(s) | Default Level |
|---------|------------------|---------------|
| `overdue_task` | `move_to_review`, `friction_flag` | SUGGEST / AUTO |
| `postponed_task` | `friction_check`, `friction_flag` | SUGGEST / AUTO |
| `exceeded_focus_session` | `hyperfocus_warning`, `minimum_survival_mode` | SUGGEST / AUTO |
| `low_physical_state` | `recovery_mode`, `minimum_survival_mode` | ASK_FIRST / AUTO |
| `high_stress` | `chaos_mode`, `reminder_noise_reduction` | ASK_FIRST / AUTO |
| `voice_captured_appointment` | `calendar_suggestion` | SUGGEST |
| `missed_completed_recurrence` | `recurrence_scheduling`, `friction_flag` | SUGGEST / AUTO |
| `friction_detection` | Per `FrictionPattern.reason` | Per reason |
| `task_creation` | Energy routing, `decomposition_suggestion` | SUGGEST |
| `spiraling_task_overload` | `chaos_mode`, `move_to_review` (bulk) | ASK_FIRST / SUGGEST |

---

## Conditions (Optional Filters)

```json
{
  "confidence": { "min": 0.8 },
  "timeOfDay": { "start": "09:00", "end": "17:00" },
  "energyThreshold": { "mental": 5 },
  "excludeDuring": ["recoveryMode", "chaosMode"]
}
```

---

## Metadata (Action-Specific)

| Action | Metadata Fields |
|--------|-----------------|
| `move_to_review` | `reason: "overdue" | "friction" | "manual"` |
| `friction_check` | `promptTemplate: "vague" | "large" | "blocked" | "dread"` |
| `calendar_suggestion` | `durationMinutes`, `attendees` |
| `decomposition_suggestion` | `maxSubtasks: 5`, `inheritEnergyTags: true` |
| `recurrence_scheduling` | `respectWeekends: true` |

---

## UserPreference Override

Each rule's effective control level = `UserPreference.automationPermissions[ruleId] || rule.controlLevel`

---

## Safety Invariants

| Invariant | Enforcement |
|-----------|-------------|
| No destructive AUTO | AUTO only: metadata, filtering, notifications |
| ASK_FIRST for state changes | recovery/chaos/minimum_survival require confirmation |
| Audit trail | Every execution → TaskEvent(type: automation_executed) |
| Rollback | All actions reversible via TaskEvent history |
| Disable respected | automationPermissions = DISABLED → never fires |

---

## Indexing

- (userId, enabled, trigger) — active rule evaluation

---

## RLS Policy (Expected)

- User owns their rules
- Read/write: userId = currentUser
- Admin bypass