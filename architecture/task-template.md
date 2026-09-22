# MindFlow — TaskTemplate Entity Specification

**Reusable task/habit definitions with energy and recurrence metadata**

---

## Purpose

Eliminates repetitive task creation for recurring work, habits, and standard procedures. Templates carry full task configuration including energy tags and recurrence rules.

---

## Fields (Verified)

| Field | Type | Description |
|-------|------|-------------|
| `name` | String | Required; template identifier |
| `description` | String | Optional; usage guidance |
| `taskTitle` | String | Task title template (can include placeholders) |
| `taskDescription` | String | Task description template |
| `energyTags` | Array<String> | From taxonomy (9 values) |
| `priority` | Enum/Number | Default priority |
| `type` | Enum | `task` / `appointment` / `reminder` / `idea` |
| `estimatedDuration` | Number | Minutes |
| `dreadFlag` | Boolean | Default: false |
| `recurring` | Boolean | Default: false |
| `recurrencePattern` | Object | If recurring: cron/natural language |
| `projectRelationship` | Reference | → Project, optional |
| `category` | String | Template category (habit, admin, creative, etc.) |
| `usageCount` | Number | Auto-incremented on use |
| `lastUsedAt` | DateTime | Timestamp of last instantiation |

---

## Instantiation

When user creates task from template:

```
Task created with:
- title ← template.taskTitle
- description ← template.taskDescription
- energyTags ← template.energyTags
- priority ← template.priority
- type ← template.type
- estimatedDuration ← template.estimatedDuration
- dreadFlag ← template.dreadFlag
- recurring ← template.recurring
- recurrenceTracking ← template.recurrencePattern
- projectRelationship ← template.projectRelationship
- templateSource ← template.id (reference)
```

Template.usageCount++ and lastUsedAt updated.

---

## Categories (Suggested)

| Category | Typical Templates |
|----------|-------------------|
| `habit` | Daily meditation, weekly review, exercise |
| `admin` | Expense filing, email cleanup, calendar review |
| `creative` | Writing session, design sprint, brainstorming |
| `communication` | Weekly sync, 1:1 prep, newsletter draft |
| `maintenance` | System updates, backup check, cleanup |
| `learning` | Reading time, course module, practice |

---

## Habit-Specific Features

| Feature | Implementation |
|---------|----------------|
| **Streak tracking** | Via TaskEvent.completed on recurring tasks |
| **Flexible scheduling** | RecurrencePattern + energy-aware rescheduling |
| **Minimum viable** | `bedridden-compatible` tag for low-capacity days |
| **Progressive overload** | Duration/intensity increase over time (manual) |

---

## Integration Points

| Integration | Purpose |
|-------------|---------|
| Task creation UI | "From template" picker |
| AutomationRule (task_creation) | Suggest templates for vague tasks |
| VoiceCapture | "Create [template name]" → instantiate |
| Insights | Template usage analytics |

---

## Indexing

- (userId, category) — category browsing
- (userId, usageCount) — most-used templates
- (userId, lastUsedAt) — recent templates

---

## Retention

- Indefinite (user configuration)
- GDPR: included in user data export

---

## RLS Policy (Expected)

- User owns their templates
- Read/write: userId = currentUser
- Admin bypass