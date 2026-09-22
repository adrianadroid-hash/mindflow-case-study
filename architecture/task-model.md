# MindFlow — Task Entity Specification

**Core work unit with energy-aware routing**

---

## Purpose

The central actionable entity. Tasks carry energy requirements, friction metadata, and decomposition relationships enabling adaptive planning.

---

## Fields (Verified)

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `title` | String | Required | Brief task name |
| `description` | String | Optional | Detailed context |
| `status` | Enum | `todo` / `in_progress` / `done` / `review` / `archived` | Workflow state |
| `priority` | Enum/Number | Required | Relative importance |
| `type` | Enum | `task` / `appointment` / `reminder` / `idea` | Semantic category |
| `energyTags` | Array<String> | From taxonomy (9 values) | Energy requirements |
| `dueTime` | DateTime | Optional | Deadline |
| `reminderTime` | DateTime | Optional | Notification trigger |
| `recurring` | Boolean | Default: false | Repeats per recurrenceTracking |
| `estimatedDuration` | Number | Minutes, optional | Planned effort |
| `dreadFlag` | Boolean | Default: false | Emotional avoidance signal |
| `archiveState` | Boolean | Default: false | Soft delete |
| `projectRelationship` | Reference | → Project, optional | Goal container |
| `decompositionRelationship` | Self-ref | → Task (parent), optional | Breakdown hierarchy |
| `postponementTracking` | Object | Auto-managed | Count, timestamps, patterns |
| `recurrenceTracking` | Object | Auto-managed | Pattern, next occurrence, history |

---

## Energy Tag Taxonomy (9 values)

| Tag | Energy Profile | Typical Use |
|-----|----------------|-------------|
| `low` | Minimal all dimensions | Filing, reading, organizing |
| `medium` | Moderate all | Email, planning, light coding |
| `high` | High mental + motivation | Deep work, complex problems |
| `social` | High social capacity | Meetings, calls, collaboration |
| `admin` | Low mental, low social | Expenses, scheduling, paperwork |
| `chaos-compatible` | Works in chaos mode | Quick capture, triage, voice |
| `bedridden-compatible` | Works at near-zero physical | Phone, mental planning, voice |
| `outdoor` | Requires movement + context | Errands, site visits, walks |
| `creative` | High mental + motivation, low stress | Design, writing, ideation |

---

## Status Flow

```
todo → in_progress → done
  │         │
  │         └→ abandoned
  │
  └→ review (from overdue/postponed/friction)
        │
        └→ todo / archived
```

---

## Energy-Aware Routing (Automation)

On `task_creation` trigger:
1. Match task.energyTags to current DailyState
2. If match ≥ threshold → suggest "Do Now"
3. If match < threshold but not dread → suggest "Schedule for [better time]"
4. If dreadFlag + low match → `friction_check` (ASK_FIRST)
5. If vague/large (heuristic) → `decomposition_suggestion` (SUGGEST)

---

## Friction Integration

| Task Field | Friction Role |
|------------|---------------|
| `dreadFlag` | Explicit dread signal; triggers `friction_detection` |
| `postponementTracking` | Counts, timestamps → `repeated_postponement` pattern |
| `estimatedDuration` vs FocusSession.actualDuration | `exceeds_estimate` pattern |
| `status` transitions | `never_started` (created > 7d, no start), `postponed_task` |
| `energyTags` vs DailyState | `energy_mismatch` pattern |

---

## Decomposition Relationship

- Parent task can have N child tasks (sub-tasks)
- Child tasks inherit projectRelationship from parent
- Child tasks can have independent energyTags
- Completion: parent.autoComplete = all children done (configurable)
- `TaskEvent.type = decomposition` on create/link

---

## Recurrence Tracking

| Field | Purpose |
|-------|---------|
| `pattern` | cron-like or natural language ("weekly", "Mon/Wed/Fri") |
| `nextOccurrence` | Computed next dueTime |
| `history` | Array of {date, status, notes} |
| `missedCount` | Consecutive missed → `missed_completed_recurrence` trigger |

---

## Indexing

- (userId, status) — active task queries
- (userId, projectId) — project task lists
- (userId, dueTime) — due date sorting
- (userId, energyTags) — energy-aware filtering (GIN/array index)
- (userId, dreadFlag) — dread-filtered views

---

## RLS Policy (Expected)

- User owns all their tasks
- Read/write: userId = currentUser
- Admin bypass