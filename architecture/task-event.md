# MindFlow — TaskEvent Entity Specification

**Immutable event log for full behavioral history**

---

## Purpose

Append-only audit trail of **every significant task interaction**. Enables friction detection, behavioral analytics, automation audit trail, and user-facing history.

---

## Fields (Verified)

| Field | Type | Description |
|-------|------|-------------|
| `eventType` | Enum (12 values) | Type of event |
| `taskRelationship` | Reference | → Task (required) |
| `timestamp` | DateTime | Required, immutable |
| `metadata` | Object | Event-specific data |

---

## 12 Event Types (Verified)

| Event Type | Trigger | Metadata |
|------------|---------|----------|
| `created` | Task created | `{source: "manual" | "voice" | "template", initialEnergyTags: [...]}` |
| `started` | Task.status → in_progress | `{previousStatus: "todo"}` |
| `paused` | Task.status → paused | `{durationInProgress: minutes}` |
| `rescheduled` | Task.dueTime changed | `{oldDueTime, newDueTime, reason}` |
| `postponed` | Task pushed to later | `{postponementCount, frictionPatternId?}` |
| `completed` | Task.status → done | `{actualDuration, focusSessions: [...]}` |
| `abandoned` | Task.status → abandoned | `{reason, timeInvested}` |
| `blocked` | Task blocked on dependency | `{blockingTaskId, blockingReason}` |
| `focus_session` | FocusSession linked | `{focusSessionId, plannedDuration, actualDuration}` |
| `decomposition` | Sub-task created/linked | `{parentTaskId, childTaskIds: [...]}` |
| `merge` | Tasks merged | `{sourceTaskIds, targetTaskId}` |
| `friction_detection` | FrictionPattern created | `{frictionPatternId, pattern, reason, confidence}` |

---

## Append-Only Guarantee

- **Never modified** after creation
- **Never deleted** (configurable retention only)
- Sequential ID or timestamp ordering

---

## Friction Detection Pipeline Input

`friction_detection` events feed the **pattern detector**:

```
TaskEvent stream (friction_detection)
         │
         ▼
┌──────────────────┐
│ Pattern Detector │  → FrictionPattern (if high confidence)
│  (7 patterns)    │
└──────────────────┘
```

---

## Automation Audit Trail

Every automation execution creates a TaskEvent:
```
automation_executed: {
  ruleId, trigger, action, controlLevel,
  result: "executed" | "dismissed" | "failed",
  timestamp
}
```

---

## Analytics Queries Enabled

| Query | Purpose |
|-------|---------|
| `postponementCount(taskId)` | Repeated postponement pattern |
| `timeToFirstStart(taskId)` | Never started pattern |
| `focusEfficiency(taskId)` | Exceeds estimate pattern |
| `viewCount(taskId)` | Repeatedly opened pattern |
| `dreadFlagChanges(taskId)` | Dread pattern |
| `energyMismatchDays(taskId)` | Energy mismatch pattern |
| `blockedDuration(taskId)` | Blocked dependency pattern |

---

## Retention

| Policy | Default | User Control |
|--------|---------|--------------|
| `maxAgeDays` | 730 (2 years) | Configurable |
| `minEventsPerTask` | 5 | Preserve minimum |
| `exportOnDelete` | true | GDPR compliance |

---

## Indexing

- (userId, timestamp) — chronological timeline
- (taskId, timestamp) — per-task history
- (userId, eventType) — filtered event streams
- (userId, timestamp, eventType) — compound analytics

---

## RLS Policy (Expected)

- User owns their events
- Read: userId = currentUser
- Write: system-only (append-only)
- Admin bypass