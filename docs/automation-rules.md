# MindFlow — Automation Rules Specification

**Triggers, actions, and control levels**

---

## Design Principle

Automation must **earn trust**. Every automated action is classified by **control level** (AUTO/SUGGEST/ASK_FIRST) so users retain agency while reducing decision fatigue for routine operations.

---

## Control Levels

| Level | Behavior | Use Case | User Consent |
|-------|----------|----------|--------------|
| **AUTO** | Execute immediately without prompt | Safe, reversible, high-confidence | Pre-authorized via UserPreference |
| **SUGGEST** | Present recommendation; user accepts/declines | Beneficial but context-dependent | Implicit (user sees suggestion) |
| **ASK_FIRST** | Request explicit confirmation before action | High-stakes, irreversible, preference-sensitive | Explicit (user must confirm) |

**Default mapping (configurable per rule):**
- Safety-critical → ASK_FIRST
- Routine maintenance → AUTO
- Intelligence/insight → SUGGEST

---

## 10 Verified Triggers

| # | Trigger ID | Fires When | Context Available |
|---|------------|------------|-------------------|
| 1 | `overdue_task` | Task.dueTime passed, status ≠ done | Task, DailyState, UserPreference |
| 2 | `postponed_task` | Task rescheduled (status change or dueTime push) | Task, postponement count, FrictionPattern |
| 3 | `exceeded_focus_session` | FocusSession.actualDuration > plannedDuration × 1.5 | FocusSession, Task, DailyState |
| 4 | `low_physical_state` | DailyState.physicalEnergy ≤ 3 | DailyState, Task (energy tags) |
| 5 | `high_stress` | DailyState.stress ≥ 7 | DailyState, Task, AutomationRule config |
| 6 | `voice_captured_appointment` | VoiceCapture creates extractable appointment | VoiceCapture, extracted task data |
| 7 | `missed_completed_recurrence` | Recurring task missed or completed (pattern break) | Task, recurrenceTracking |
| 8 | `friction_detection` | FrictionPattern created (high confidence) | FrictionPattern, Task, DailyState |
| 9 | `task_creation` | New Task created (initial routing) | Task, DailyState, UserPreference |
| 10 | `spiraling_task_overload` | ≥ 5 tasks with `overdue_task` or `postponed_task` in 24h | Task aggregate, DailyState |

---

## 11 Verified Actions

| # | Action ID | Effect | Typical Control Level | Safety Notes |
|---|-----------|--------|----------------------|--------------|
| 1 | `move_to_review` | Task.status → review; removed from active views | SUGGEST/ASK_FIRST | Reversible; preserves all data |
| 2 | `friction_check` | Prompt structured friction diagnosis (vague? large? blocked?) | SUGGEST | Non-destructive; user-guided |
| 3 | `hyperfocus_warning` | Notify: "You've been focused for X min (planned Y)" | SUGGEST | Informational only |
| 4 | `recovery_mode` | Activate UserPreference.recoveryMode = true | ASK_FIRST | Changes system behavior broadly |
| 5 | `chaos_mode` | Activate UserPreference.chaosMode = true | ASK_FIRST | Changes system behavior broadly |
| 6 | `minimum_survival_mode` | Filter to only chaos-compatible/bedridden-compatible tasks | AUTO (safety) | User can override; preserves all tasks |
| 7 | `calendar_suggestion` | Propose calendar event for appointment/follow-up | SUGGEST | Requires calendar connector |
| 8 | `recurrence_scheduling` | Auto-schedule next recurrence instance | SUGGEST/ASK_FIRST | Respects recurrenceTracking |
| 9 | `friction_flag` | Add friction metadata to Task; surface in Insights | AUTO (metadata only) | Non-user-facing; analytical |
| 10 | `decomposition_suggestion` | Propose sub-task breakdown for vague/large task | SUGGEST | User accepts/rejects/modifies |
| 11 | `reminder_noise_reduction` | Suppress low-priority reminders during chaos/recovery | AUTO (filtering) | User can disable in UserPreference |

---

## Trigger → Action Mapping (Default)

| Trigger | Primary Action(s) | Control Level | Rationale |
|---------|------------------|---------------|-----------|
| `overdue_task` | `move_to_review`, `friction_flag` | SUGGEST / AUTO | Overdue = review needed; flag for analytics |
| `postponed_task` | `friction_check`, `friction_flag` | SUGGEST / AUTO | Postponement = friction signal |
| `exceeded_focus_session` | `hyperfocus_warning`, `minimum_survival_mode` | SUGGEST / AUTO | Hyperfocus = safety risk |
| `low_physical_state` | `recovery_mode`, `minimum_survival_mode` | ASK_FIRST / AUTO | Physical low = state change |
| `high_stress` | `chaos_mode`, `reminder_noise_reduction` | ASK_FIRST / AUTO | Stress = state change |
| `voice_captured_appointment` | `calendar_suggestion` | SUGGEST | Voice = intent capture |
| `missed_completed_recurrence` | `recurrence_scheduling`, `friction_flag` | SUGGEST / AUTO | Recurrence = pattern maintenance |
| `friction_detection` | Per `FrictionPattern.reason` (see friction-taxonomy.md) | Per reason | Context-specific |
| `task_creation` | Initial energy routing, `decomposition_suggestion` if vague/large | SUGGEST | First-touch intelligence |
| `spiraling_task_overload` | `chaos_mode`, `move_to_review` (bulk) | ASK_FIRST / SUGGEST | Overload = system intervention |

---

## UserPreference Configuration

| Preference Field | Controls |
|------------------|----------|
| `automationPermissions` | Per-rule: AUTO/SUGGEST/ASK_FIRST/DISABLED |
| `reminderTolerance` | Threshold for `reminder_noise_reduction` |
| `recommendationLimits` | Max suggestions per session |
| `focusDuration` | Baseline for `exceeded_focus_session` |
| `primaryFriction` | User's self-identified main friction type |

---

## Safety Invariants

| Invariant | Enforcement |
|-----------|-------------|
| **No destructive AUTO actions** | All AUTO actions are metadata-only or filtering (reversible) |
| **ASK_FIRST for state changes** | recovery_mode, chaos_mode, minimum_survival_mode require confirmation |
| **User can disable any rule** | automationPermissions = DISABLED |
| **Audit trail** | Every automation execution → TaskEvent (automation_executed) |
| **Rollback capability** | All actions reversible via TaskEvent history |

---

## Extensibility

New triggers/actions can be added via Base44 configuration without code changes:
- Trigger: event type + filter conditions
- Action: operation type + parameters
- Control level: AUTO/SUGGEST/ASK_FIRST
- Permissions: linked to UserPreference.automationPermissions