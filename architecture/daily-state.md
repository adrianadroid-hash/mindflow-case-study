# MindFlow — DailyState Entity Specification

**Daily energy and capacity snapshot**

---

## Purpose

Capture the user's **daily operating capacity** across multiple energy dimensions, plus contextual modifiers. This is the primary input for energy-aware planning, friction detection, and automation triggers.

---

## Fields (Verified)

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `date` | Date | Required, unique per user | Calendar date of snapshot |
| `mentalEnergy` | Number | 0–10, required | Cognitive capacity: focus, analysis, working memory |
| `physicalEnergy` | Number | 0–10, required | Bodily capacity: movement, stamina, sensory tolerance |
| `socialCapacity` | Number | 0–10, required | Interpersonal bandwidth: communication, meetings |
| `motivation` | Number | 0–10, required | Drive/initiative: task initiation, persistence |
| `stress` | Number | 0–10, required | Pressure/arousal (inverted: lower = better) |
| `availableMinutes` | Number | ≥0, required | Explicit time budget for productive work |
| `locationContext` | String | Optional | Environment: home/office/travel/noisy/quiet |
| `notes` | String | Optional | Free-text context (mood, events, observations) |
| `recoveryMode` | Boolean | Default: false | Activated by user or automation |
| `chaosMode` | Boolean | Default: false | Activated by user or automation |

---

## Derived/Computed Fields (Not Stored)

| Computed | Formula | Use |
|----------|---------|-----|
| `overallEnergy` | mean(mental, physical, social, motivation) - stress | Quick health indicator |
| `capacityScore` | availableMinutes × energyMatchFactor × modeModifier | Planning budget |
| `isLowCapacity` | overallEnergy < 4 OR recoveryMode OR chaosMode | Automation trigger |

---

## Energy Match Factor (Conceptual)

```
energyMatchFactor = 
  (mentalEnergy/10 × mentalWeight) +
  (physicalEnergy/10 × physicalWeight) +
  (socialCapacity/10 × socialWeight) +
  (motivation/10 × motivationWeight) -
  (stress/10 × stressWeight)
```
Weights configurable via UserPreference.energyDefaults

---

## Mode Modifiers

| Mode | Modifier | Effect |
|------|----------|--------|
| Normal | 1.0 | Standard planning |
| Recovery Mode | 0.3 | Drastically reduced capacity |
| Chaos Mode | 0.2 | Minimal capacity; only chaos-compatible tasks |
| Minimum Survival | 0.1 | Bedridden-compatible only |

---

## Automation Triggers Referencing DailyState

| Trigger | Condition | Action |
|---------|-----------|--------|
| `low_physical_state` | physicalEnergy ≤ 3 | `recovery_mode`, `minimum_survival_mode` |
| `high_stress` | stress ≥ 7 | `chaos_mode`, `reminder_noise_reduction` |
| `task_creation` | (on create) | Route task by energy match |
| `friction_detection` | (via FrictionPattern) | Energy mismatch detection |

---

## Indexing

- **Primary:** (userId, date) — unique daily snapshot
- **Query:** userId + date range (weekly/monthly trends)

---

## Retention

- **Indefinite** — daily snapshots enable long-term trend analysis
- User can delete individual entries
- GDPR: included in user data export

---

## Privacy

- Contains intimate self-tracking data (mood, stress, energy, notes)
- **RLS required:** userId = currentUser only
- No cross-user analytics without explicit consent
- Export/delete on request