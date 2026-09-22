# MindFlow — Project Entity Specification

**Goal-oriented containers for task organization**

---

## Purpose

Projects provide **goal context** for tasks, enabling progress tracking, deadline management, and strategic alignment. They are the "why" behind task collections.

---

## Fields (Verified)

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `goal` | String | Required | Outcome statement |
| `status` | Enum | `active` / `on_hold` / `completed` / `archived` | Project lifecycle |
| `priority` | Enum/Number | Required | Relative importance |
| `deadline` | Date | Optional | Target completion |
| `progress` | Number | 0–100, computed or manual | Completion percentage |
| `category` | String | Optional | Domain tag (work, personal, health, etc.) |
| `notes` | String | Optional | Context, constraints, decisions |
| `nextAction` | String | Optional | GTD-style immediate next step |

---

## Relationships

| Relationship | Cardinality | Notes |
|--------------|-------------|-------|
| Project → Task | 1 : N | Via Task.projectRelationship |
| Project → User | N : 1 | Owner |

---

## Progress Computation (Options)

| Method | Formula |
|--------|---------|
| **Task-count** | completedTasks / totalTasks × 100 |
| **Weighted** | Σ(task.weight × done) / Σ(task.weight) × 100 |
| **Manual** | User-set (overrides computed) |

Default: task-count; configurable per project.

---

## Automation Integration

| Trigger | Action |
|---------|--------|
| `task_creation` (with project) | Update project.progress |
| `task_completion` | Update project.progress; check deadline |
| `deadline_approaching` | Notify; suggest nextAction |

---

## Indexing

- (userId, status) — active project list
- (userId, deadline) — deadline sorting
- (userId, category) — category filtering

---

## RLS Policy (Expected)

- User owns all their projects
- Read/write: userId = currentUser
- Admin bypass