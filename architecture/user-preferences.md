# MindFlow — UserPreference Entity Specification

**Complete user configuration and automation permissions**

---

## Purpose

Centralizes all user-configurable behavior: working patterns, automation control, energy defaults, notification preferences. This entity is the **control plane** for personalization.

---

## Fields (Verified)

| Field | Type | Description |
|-------|------|-------------|
| `workingHours` | Object | `{start: "09:00", end: "17:00", days: [1,2,3,4,5]}` |
| `sleepTime` | Time | Typical sleep onset |
| `wakeTime` | Time | Typical wake time |
| `focusDuration` | Number | Default focus session length (minutes) |
| `recommendationLimits` | Object | `{maxPerSession: 5, maxPerDay: 20}` |
| `reminderTolerance` | Object | `{minIntervalMinutes: 30, suppressDuringFocus: true}` |
| `energyDefaults` | Object | Baseline when DailyState missing: `{mental: 5, physical: 5, social: 5, motivation: 5, stress: 3}` |
| `planningStyle` | Enum | `flexible` / `structured` / `minimal` |
| `weekendBehavior` | Object | `{enabled: true, reducedCapacity: 0.6, differentHours: {...}}` |
| `notificationPreferences` | Object | Per-channel: push, email, in-app |
| `automationPermissions` | Object | **Per-rule control level override** |
| `primaryFriction` | String | Self-identified main friction type |
| `onboardingState` | Object | `{completed: boolean, currentStep: number, seenFeatures: [...]}` |

---

## automationPermissions Structure (Critical)

```json
{
  "overdue_task": "SUGGEST",
  "postponed_task": "SUGGEST",
  "exceeded_focus_session": "AUTO",
  "low_physical_state": "ASK_FIRST",
  "high_stress": "ASK_FIRST",
  "voice_captured_appointment": "SUGGEST",
  "missed_completed_recurrence": "SUGGEST",
  "friction_detection": "SUGGEST",
  "task_creation": "SUGGEST",
  "spiraling_task_overload": "ASK_FIRST"
}
```

**Values per rule:** `AUTO` | `SUGGEST` | `ASK_FIRST` | `DISABLED`

User can override any rule's default control level.

---

## planningStyle Behavior

| Style | Recommendation Density | Automation Tendency |
|-------|------------------------|---------------------|
| `flexible` | Low (3-5/day) | More SUGGEST, fewer AUTO |
| `structured` | High (10-15/day) | More AUTO, scheduled blocks |
| `minimal` | Very low (1-3/day) | Mostly ASK_FIRST, manual |

---

## weekendBehavior Detail

```json
{
  "enabled": true,
  "reducedCapacity": 0.6,
  "differentHours": {
    "start": "10:00",
    "end": "16:00",
    "days": [0, 6]
  },
  "suppressNonCritical": true
}
```

---

## onboardingState Flow

```
step 0: welcome → energy defaults setup
step 1: working hours → sleep/wake
step 2: planning style → focus duration
step 3: automation permissions (guided)
step 4: first DailyState → first Task
step 5: first FocusSession → first Automation trigger
completed: true
```

---

## Indexing

- userId (1:1 with User) — primary lookup

---

## RLS Policy (Expected)

- User owns their preferences
- Read/write: userId = currentUser
- Admin bypass
- **Critical:** automationPermissions never exposed to other users