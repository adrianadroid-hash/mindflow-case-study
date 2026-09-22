# MindFlow — FrictionPattern Entity Specification

**Detected behavioral friction patterns with root-cause classification**

---

## Purpose

Transforms raw TaskEvent streams into **structured friction intelligence**: what pattern, why it's happening, and what automation should do about it.

---

## Fields (Verified)

| Field | Type | Description |
|-------|------|-------------|
| `taskRelationship` | Reference | → Task (required, 1:1) |
| `pattern` | Enum (7 values) | Detected friction pattern |
| `reason` | Enum (9 values) | Classified root cause |
| `confidence` | Number | 0.0–1.0 (detection confidence) |
| `detectedAt` | DateTime | When pattern was identified |
| `evidence` | Object | Supporting TaskEvent references |
| `resolved` | Boolean | Default: false |
| `resolvedAt` | DateTime | Optional |
| `resolution` | Enum | `decomposed` / `rescheduled` / `clarified` / `deferred` / `archived` / `dismissed` |

---

## 7 Patterns (Verified)

| Pattern | Description | Detection Signal |
|---------|-------------|------------------|
| `repeated_postponement` | Task rescheduled ≥3× without progress | postponementTracking.count ≥ 3 |
| `never_started` | Created >7 days ago, no start event | TaskEvent.started absent, age > 7d |
| `exceeds_estimate` | Actual focus > 2× estimated | FocusSession.actual > estimated × 2 |
| `repeatedly_opened` | Task viewed ≥5× without status change | View/open events (if tracked) ≥ 5 |
| `dread` | Dread flag + avoidance behavior | Task.dreadFlag + postponement/never_started |
| `energy_mismatch` | Energy tags consistently incompatible | DailyState vs energyTags over ≥3 days |
| `blocked_dependency` | Blocked on incomplete dependency | TaskEvent.blocked + no resolution path |

---

## 9 Reasons (Verified)

| Reason | Description | Typical Automation |
|--------|-------------|-------------------|
| `too_vague` | No concrete next action | `decomposition_suggestion` |
| `too_large` | Scope exceeds session capacity | `decomposition_suggestion` |
| `missing_info` | Required info unavailable | `friction_check` → identify missing piece |
| `emotionally_unpleasant` | Avoidance trigger (scary, boring, difficult) | ASK_FIRST dread support |
| `requires_another_person` | External dependency | `calendar_suggestion` |
| `wrong_timing` | Energy tags mismatch capacity | Reschedule to better window |
| `excessive_energy` | Demands more than typically available | Decompose + energy tag review |
| `not_important` | Low priority vs active goals | `move_to_review` |
| `unknown` | Pattern detected, reason unclear | `friction_flag` for human review |

---

## Confidence Scoring

| Range | Label | Automation Eligibility |
|-------|-------|------------------------|
| ≥ 0.8 | High | AUTO actions allowed |
| 0.5–0.79 | Medium | SUGGEST/ASK_FIRST only |
| < 0.5 | Low | Flag only; no automation |

**Only High confidence triggers AUTO-level automations.**

---

## Detection Pipeline

```
TaskEvent stream
       │
       ▼
┌──────────────────┐
│ Pattern Matcher  │  (7 pattern detectors, each with logic)
│  (runs per-event │
│   + batch)       │
└────────┬─────────┘
         │ candidate
         ▼
┌──────────────────┐
│ Reason Classifier│  (infers 1 of 9 from context)
│  (heuristics +   │
│   ML optional)   │
└────────┬─────────┘
         │ classified
         ▼
┌──────────────────┐
│ FrictionPattern  │  (record created if confidence ≥ 0.3)
│   Record         │
└──────────────────┘
```

---

## Resolution Flow

```
FrictionPattern created (confidence ≥ 0.3)
         │
         ▼
AutomationRule (trigger: friction_detection) evaluates
         │
         ├── AUTO: execute action (metadata, filtering)
         ├── SUGGEST: present to user
         └── ASK_FIRST: request confirmation
         │
         ▼
User responds / automation executes
         │
         ▼
TaskEvent: automation_executed / friction_resolved
         │
         ▼
FrictionPattern.resolved = true
FrictionPattern.resolution = [decomposed|rescheduled|clarified|deferred|archived|dismissed]
```

---

## Evidence Object

```json
{
  "supportingEvents": ["taskEventId1", "taskEventId2"],
  "postponementCount": 4,
  "daysSinceCreated": 12,
  "averageDailyState": { "mental": 4, "physical": 3 },
  "energyTagMismatchDays": 5
}
```

---

## Indexing

- (userId, taskId) — task friction lookup
- (userId, pattern, confidence) — analytics
- (userId, resolved, detectedAt) — unresolved friction queue

---

## Retention

- Linked to Task lifecycle (resolved when task resolved)
- Or configurable max age (default: 1 year)
- GDPR: included in user data export

---

## RLS Policy (Expected)

- User owns their friction patterns
- Read/write: userId = currentUser
- Admin bypass
- **Sensitive:** Reveals emotional/behavioral patterns — extra care