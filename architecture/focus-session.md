# MindFlow — FocusSession Entity Specification

**Planned vs. actual focus tracking with hyperfocus detection**

---

## Purpose

Captures **intentional focus periods** to enable: planned vs. actual comparison, hyperfocus alerts, energy-aware scheduling, and behavioral analytics.

---

## Fields (Verified)

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `plannedDuration` | Number | Minutes, required | Intended focus length |
| `actualDuration` | Number | Minutes, required | Actual elapsed time |
| `hyperfocusAlert` | Boolean | Default: false | Triggered when actual > planned × 1.5 |
| `taskRelationship` | Reference | → Task, optional | Associated task |
| `startTime` | DateTime | Required | Session start |
| `endTime` | DateTime | Optional | Session end (null if active) |
| `interrupted` | Boolean | Default: false | External interruption |

---

## Relationships

| Relationship | Cardinality | Notes |
|--------------|-------------|-------|
| FocusSession → Task | N : 1 | Optional; links focus to work |
| FocusSession → User | N : 1 | Owner |

---

## Hyperfocus Detection

**Trigger:** `exceeded_focus_session` fires when `actualDuration > plannedDuration × 1.5`

**Automation Response:**
- `hyperfocus_warning` (SUGGEST): "You've focused for 120 min (planned 60). Break?"
- `minimum_survival_mode` (AUTO): If physicalEnergy low, activate safety filter

---

## Integration Points

| Integration | Purpose |
|-------------|---------|
| Task.estimatedDuration | Baseline for plannedDuration |
| DailyState.availableMinutes | Capacity budget for planning |
| AutomationRule (exceeded_focus_session) | Hyperfocus safety |
| TaskEvent (focus_session) | History/audit trail |

---

## Metrics Derived

| Metric | Formula |
|--------|---------|
| `focusEfficiency` | actualDuration / plannedDuration |
| `dailyFocusTotal` | Σ(actualDuration) per date |
| `hyperfocusFrequency` | count(hyperfocusAlert=true) per week |
| `averageSessionLength` | mean(actualDuration) |

---

## Indexing

- (userId, startTime) — chronological queries
- (userId, date) — daily focus totals
- (taskId) — task-specific focus history

---

## RLS Policy (Expected)

- User owns their focus sessions
- Read/write: userId = currentUser
- Admin bypass