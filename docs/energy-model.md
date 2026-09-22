# MindFlow — Energy Model Design

**Multi-axis energy system specification**

---

## Design Principle

Energy is not a single scalar. Humans operate on **multiple simultaneous energy dimensions** that deplete and recover at different rates. A productivity system must model this dimensionality to make accurate capacity predictions.

---

## Five Energy Dimensions (DailyState)

| Dimension | Range | Semantic Meaning | Recovery Characteristics |
|-----------|-------|------------------|-------------------------|
| **Mental Energy** | 0–10 | Cognitive capacity: focus, analysis, decision-making, working memory | Slow recovery; sleep-dependent; depleted by deep work |
| **Physical Energy** | 0–10 | Bodily capacity: movement, stamina, sensory tolerance | Medium recovery; movement/nutrition dependent |
| **Social Capacity** | 0–10 | Interpersonal bandwidth: communication, empathy, meeting tolerance | Variable recovery; alone-time dependent |
| **Motivation** | 0–10 | Drive/initiative: task initiation, persistence, goal-directedness | Fluctuates rapidly; context-sensitive |
| **Stress** | 0–10 | Pressure/arousal: anxiety, urgency, overwhelm (inverted — lower is better) | Recovery requires active down-regulation |

---

## Capacity Derivation

| Input | Role |
|-------|------|
| **Available Minutes** | Explicit time budget for the day (from calendar, commitments) |
| **Location/Context** | Environmental modifier (home/office/travel/noisy/quiet) |
| **Energy Dimensions** | Multiplicative filters on task eligibility |
| **Recovery/Chaos Mode** | Boolean overrides that change all rules |

**Capacity Formula (Conceptual):**
```
EffectiveCapacity = AvailableMinutes × EnergyMatch(task) × ContextModifier × ModeModifier
```

Where `EnergyMatch(task)` computes compatibility between task's energy tags and current energy dimensions.

---

## Energy Tags (Task-Side)

Tasks declare their energy requirements via **tags** (9 taxonomy values):

| Tag | Energy Dimension(s) Affected | Typical Tasks |
|-----|------------------------------|---------------|
| `low` | All dimensions minimal | Admin filing, reading, organizing |
| `medium` | Moderate across dimensions | Email, planning, light coding |
| `high` | High mental + motivation | Deep work, complex problem-solving |
| `social` | High social capacity | Meetings, calls, collaboration |
| `admin` | Low mental, low social | Expenses, scheduling, paperwork |
| `chaos-compatible` | Works in chaos mode | Quick captures, voice notes, triage |
| `bedridden-compatible` | Works at near-zero physical | Phone tasks, mental planning, voice |
| `outdoor` | Requires physical movement + context | Errands, site visits, walks |
| `creative` | High mental + motivation + low stress | Design, writing, ideation |

---

## Energy Matching Algorithm (Conceptual)

```
function energyMatch(task, dailyState):
  score = 0
  for each tag in task.energyTags:
    score += dimensionCompatibility(tag, dailyState)
  return normalize(score)

function dimensionCompatibility(tag, dailyState):
  // Map tag → required dimensions → compare to dailyState values
  // e.g., 'high' requires mentalEnergy ≥ 7, motivation ≥ 6
  // 'chaos-compatible' returns 1.0 if chaosMode=true, else 0.3
  // 'bedridden-compatible' returns 1.0 if physicalEnergy ≤ 3, else 0.5
```

---

## Mode Overrides

| Mode | Trigger | Effect on Energy Matching | Effect on Automation |
|------|---------|---------------------------|---------------------|
| **Recovery Mode** | Manual or automation (low physical, high stress) | All tags score lower; only `chaos-compatible`/`bedridden-compatible` viable | Minimum survival mode activated; non-critical automations paused |
| **Chaos Mode** | Manual or automation (spiraling overload) | Only `chaos-compatible` tasks shown | Reminder-noise reduction; hyperfocus warnings aggressive |
| **Minimum Survival** | Automation (exceeded focus, low physical) | Only `bedridden-compatible` + `chaos-compatible` | All SUGGEST→ASK_FIRST; AUTO paused except safety |

---

## UserPreference Energy Defaults

| Preference | Purpose |
|------------|---------|
| `energyDefaults` | Baseline assumptions when DailyState not logged |
| `workingHours` | Constrains AvailableMinutes calculation |
| `focusDuration` | Informs FocusSession planning |
| `planningStyle` (flexible/structured/minimal) | Changes recommendation density |

---

## Design Rationale

1. **Multi-dimensional > single score** — A user can have high mental but low physical; single "energy" score loses this
2. **Tags > estimates** — Energy tags are more honest than time estimates (which users consistently misjudge)
3. **Explicit modes > implicit failure** — Recovery/chaos modes make state visible and actionable
4. **Daily snapshot > continuous tracking** — DailyState is a deliberate checkpoint, not surveillance
5. **Tags on tasks, not users** — Tasks declare what they need; system matches to daily reality

---

## Privacy Note

DailyState contains **intimate self-tracking data** (mood, stress, energy, notes). Any implementation must:
- Never expose to other users (RLS required)
- Support full deletion/export
- Minimize retention (DailyState is daily; old entries lose relevance)
- VoiceCapture has explicit auto-deletion metadata