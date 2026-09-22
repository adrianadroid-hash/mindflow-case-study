# MindFlow — Entity Relationships

**Complete entity-relationship diagram (textual)**

---

## Overview

```
User (1) ─────< DailyState (1 per date)
User (1) ─────< Task (N)
User (1) ─────< Project (N)
User (1) ── UserPreference (1)
User (1) ─────< Routine (N)
User (1) ─────< AutomationRule (N)
User (1) ─────< FrictionPattern (N)
User (1) ─────< TaskEvent (N)
User (1) ─────< FocusSession (N)
User (1) ─────< VoiceCapture (N)
User (1) ─────< TaskTemplate (N)

Task (N) ──> Project (1) [projectRelationship]
Task (N) ──> Task (1) [decompositionRelationship, self-ref]
Task (N) ──> Routine (1) [routine assignment]
Task (N) ──> TaskTemplate (1) [template source]

AutomationRule (N) ──> Trigger types (10)
AutomationRule (N) ──> Action types (11)
AutomationRule (N) ──> ControlLevel (AUTO/SUGGEST/ASK_FIRST)

FrictionPattern (1) ──> Task (1) [pattern detected on]
FrictionPattern (N) ──> User (1)

TaskEvent (N) ──> Task (1)
TaskEvent (N) ──> User (1)
TaskEvent types: created, started, paused, rescheduled, postponed, completed, abandoned, blocked, focus_session, decomposition, merge, friction_detection

FocusSession (N) ──> Task (1)
FocusSession (N) ──> User (1)

VoiceCapture (N) ──> User (1)
VoiceCapture (N) ──> extractedTasks → Task (proposed)
```

---

## Relationship Cardinalities

| Relationship | Cardinality | Notes |
|--------------|-------------|-------|
| User → DailyState | 1 : (0..1 per date) | Unique constraint: (userId, date) |
| User → Task | 1 : N | Owner |
| User → Project | 1 : N | Owner |
| User → UserPreference | 1 : 1 | Created on signup |
| User → Routine | 1 : N | Owner |
| User → AutomationRule | 1 : N | Owner |
| User → FrictionPattern | 1 : N | Detected on user's tasks |
| User → TaskEvent | 1 : N | Actor |
| User → FocusSession | 1 : N | Owner |
| User → VoiceCapture | 1 : N | Owner |
| User → TaskTemplate | 1 : N | Owner |
| Task → Project | N : 1 | Optional (task can be project-less) |
| Task → Task (decomposition) | N : 1 | Self-referential; parent task |
| Task → Routine | N : 1 | Optional routine assignment |
| Task → TaskTemplate | N : 1 | Optional template source |
| AutomationRule → Trigger | N : 1 | 10 trigger types |
| AutomationRule → Action | N : 1 | 11 action types |
| FrictionPattern → Task | 1 : 1 | Pattern detected on specific task |

---

## Key Indices (Logical)

| Entity | Index | Purpose |
|--------|-------|---------|
| DailyState | (userId, date) unique | Daily snapshot lookup |
| Task | (userId, status) | Active task queries |
| Task | (userId, projectId) | Project task lists |
| Task | (userId, dueTime) | Due date sorting |
| Task | (userId, energyTags) | Energy-aware filtering |
| TaskEvent | (userId, timestamp) | Chronological history |
| TaskEvent | (taskId, timestamp) | Per-task history |
| FrictionPattern | (userId, taskId) | Task friction lookup |
| FrictionPattern | (userId, pattern, confidence) | Analytics |
| FocusSession | (userId, date) | Daily focus tracking |
| VoiceCapture | (userId, createdAt) | Cleanup queries |
| AutomationRule | (userId, enabled) | Active rule evaluation |

---

## RLS Policy (Verified for NexusTask; Not Verified for MindFlow)

**Expected MindFlow RLS (by analogy to NexusTask):**
- User owns all their entities
- User can only read/write: DailyState, Task, Project, UserPreference, Routine, AutomationRule, FrictionPattern, TaskEvent, FocusSession, VoiceCapture, TaskTemplate where userId = currentUser
- Admin role bypasses RLS
- No cross-user queries possible

---

## Cascade Behaviors

| Parent Deleted | Child Behavior |
|----------------|----------------|
| User | Cascade delete all owned entities (GDPR) |
| Project | Tasks: set projectRelationship = null (preserve tasks) |
| Task (parent) | Decomposition children: set decompositionRelationship = null |
| Routine | Tasks: set routine = null |
| TaskTemplate | Tasks: set template = null (preserve tasks) |
| AutomationRule | No cascade (rules reference tasks/events by ID) |

---

## Data Retention

| Entity | Retention Policy | Rationale |
|--------|------------------|-----------|
| DailyState | Indefinite (daily snapshots) | Long-term energy trend analysis |
| Task | Indefinite (user-controlled archive) | Complete history |
| TaskEvent | Configurable (default: 2 years) | Behavioral analytics; privacy |
| FocusSession | Configurable (default: 1 year) | Focus trend analysis |
| VoiceCapture | Auto-deletion (default: 24h) | Privacy; transient by design |
| FrictionPattern | Indefinite (linked to Task) | Pattern learning |
| AutomationRule | Indefinite (user configuration) | User preferences |