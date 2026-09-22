# MindFlow — Architecture Overview

**High-level system architecture**

---

## System Context

```
┌─────────────────────────────────────────────────────────────┐
│                        Base44 Platform                       │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                  MindFlow Application                │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐  │    │
│  │  │   Data   │ │  Logic   │ │   UI     │ │  Auth  │  │    │
│  │  │  Layer   │ │  Layer   │ │  Layer   │ │        │  │    │
│  │  └──────────┘ └──────────┘ └──────────┘ └────────┘  │    │
│  └─────────────────────────────────────────────────────┘    │
│         │              │              │              │        │
│         ▼              ▼              ▼              ▼        │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              Base44 Services                         │    │
│  │  (Database, Auth, Connectors, Deploy, Runtime)       │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

---

## Data Layer (12 Entities)

### Core Domain Entities
| Entity | Responsibility | Key Relationships |
|--------|---------------|-------------------|
| **DailyState** | Daily energy/capacity snapshot | 1:1 per User per date |
| **Task** | Core work unit | N:1 Project, N:1 User, Self-ref (decomposition), N:1 Routine |
| **Project** | Goal container | 1:N Tasks, 1:1 User |
| **UserPreference** | Configuration | 1:1 User |

### Automation & Intelligence Entities
| Entity | Responsibility | Key Relationships |
|--------|---------------|-------------------|
| **AutomationRule** | Trigger→Action mappings | N:1 User, references Task/DailyState/FrictionPattern |
| **FrictionPattern** | Detected behavioral patterns | 1:1 Task (pattern detection), N:1 User |
| **Routine** | Ordered trigger-based sequences | 1:N Tasks, 1:1 User |

### Observability & History Entities
| Entity | Responsibility | Key Relationships |
|--------|---------------|-------------------|
| **TaskEvent** | Immutable event log | N:1 Task, N:1 User |
| **FocusSession** | Focus tracking | N:1 Task, N:1 User |
| **VoiceCapture** | Temporary voice processing | N:1 User, auto-deletion |

### Reusability Entities
| Entity | Responsibility | Key Relationships |
|--------|---------------|-------------------|
| **TaskTemplate** | Reusable task definitions | N:1 User, referenced by Task creation |

### Foundation Entity
| Entity | Responsibility |
|--------|---------------|
| **User** | Account, ownership, RLS context |

---

## Logic Layer

### Automation Engine
```
Trigger Detection → Rule Matching → Control Level Check → Action Execution
     │                    │                  │                    │
     ▼                    ▼                  ▼                    ▼
  Event-based         Pattern-based       AUTO/SUGGEST/      Move to review,
  (overdue,           (friction,          ASK_FIRST          friction check,
   voice, etc.)       hyperfocus,         decision           hyperfocus warning,
                      overload)                                recovery mode, etc.
```

### Energy-Aware Planning
```
DailyState (energy + capacity) + UserPreference (defaults) + Task (energy tags)
         │
         ▼
  Filter/rank tasks by: energy match, capacity fit, priority, friction risk
         │
         ▼
  Present "Do Now" / "Schedule" / "Defer" / "Decompose" recommendations
```

### Friction Detection Pipeline
```
TaskEvent stream → Pattern detection (7 patterns) → FrictionPattern record
                                                    │
                                                    ▼
                                           AutomationRule trigger
                                                    │
                                                    ▼
                                    SUGGEST/ASK_FIRST: decomposition,
                                    vague-task clarification, etc.
```

---

## UI Layer (Not Verified — Requires Base44 Access)

### Anticipated Screens (from architecture)
| Screen | Primary Entities | Purpose |
|--------|-----------------|---------|
| **Daily Dashboard** | DailyState, Task, FocusSession | Energy-aware daily view |
| **Task Manager** | Task, Project, FrictionPattern | Friction-aware task operations |
| **Project View** | Project, Task (decomposition) | Goal tracking + breakdown |
| **Automation Center** | AutomationRule, TaskEvent | Rule configuration + history |
| **Insights** | TaskEvent, FrictionPattern, DailyState | Behavioral patterns, energy trends |
| **Voice Capture** | VoiceCapture | Quick capture → task extraction |
| **Settings** | UserPreference, User | Preferences, automation permissions |

---

## Integration Layer

| Integration Type | Status | Details |
|-----------------|--------|---------|
| **Base44 Connectors** | 81 available, 0 connected | Extensible for calendar, email, storage, etc. |
| **AI Providers** | Not verified | Prompts/models/tools require export audit |
| **Voice Stack** | Designed (VoiceCapture) | STT/TTS/wake word — implementation unverified |
| **Auth** | Base44-managed | Platform default |

---

## Deployment & Runtime

| Aspect | Status |
|--------|--------|
| **Platform** | Base44 (hosted) |
| **Export capability** | Requires Builder plan |
| **Build process** | Base44-managed |
| **Runtime** | Base44 runtime (not verified) |
| **Scaling** | Base44-managed |
| **Monitoring** | Base44-managed |

---

## Security Boundaries

| Boundary | Mechanism |
|----------|-----------|
| **User isolation** | Base44 platform (verified for NexusTask RLS; MindFlow RLS not verified) |
| **Data ownership** | User entity ownership context |
| **Secrets** | Base44 environment management |
| **API keys** | Base44 connector configuration |
| **Voice data** | Auto-deletion metadata specified in VoiceCapture |

---

## Scalability Considerations (Architectural)

| Concern | Architectural Approach |
|---------|------------------------|
| **Event volume** | TaskEvent is append-only; consider partitioning by User/date |
| **Automation evaluation** | Trigger-based (not polling); scales with event rate |
| **Friction detection** | Pattern matching on TaskEvent stream; incremental |
| **DailyState** | One per user per day; trivial scale |
| **Connector fan-out** | 81 connector types; lazy initialization |

---

## Technical Debt / Known Gaps (Architecture Level)

| Gap | Impact | Resolution Path |
|-----|--------|-----------------|
| **RLS not verified for MindFlow** | Multi-tenant safety unknown | Verify during Base44 export audit |
| **AI behavior not documented** | Transparency gap | Export prompts/models/tools |
| **Voice auto-deletion not verified** | Privacy risk | Verify implementation |
| **No local export** | Portability blocked | Builder plan required |
| **Connector config not audited** | Integration surface unknown | Export + review |