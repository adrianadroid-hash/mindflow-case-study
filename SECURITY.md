# MindFlow — Security Model & Source Availability Notice

**Last updated:** 2026-09-22  
**Repository type:** Documentation-only case study (no source code)

---

## Source Availability Status

| Aspect | Status | Detail |
|--------|--------|--------|
| **Source code in this repository** | **NONE** | This repository contains only documentation |
| **Canonical source location** | Base44 (App ID: `69ffc127443f6c1b56bb1ceb`) | Platform-hosted; not locally exportable without Builder plan |
| **Source export capability** | **BLOCKED** | Requires Base44 Builder plan subscription |
| **Claimed source ownership** | **NO CLAIM MADE** | This repo does not assert ownership of exportable source |
| **Future source integration** | **PLANNED** | If/when export available, will be added under `src/` after audit |

> **IMPORTANT:** Do not interpret this repository as an open-source release of the MindFlow application. It is a documented case study of a product whose source is currently platform-locked.

---

## Security Architecture (Verified from Base44 Inspection)

| Layer | Mechanism | Status |
|-------|-----------|--------|
| **Platform authentication** | Base44-managed | ✅ VERIFIED (platform default) |
| **Data storage** | Base44-managed backend | ✅ VERIFIED (platform default) |
| **Row-level access controls** | Not verified for MindFlow | ❓ NOT AUDITED (NexusTask has RLS; MindFlow unverified) |
| **Environment variables / secrets** | Base44-managed | ❓ NOT AUDITED (names only visible in export) |
| **External API integrations** | 81 connector types available; 0 connected | ✅ VERIFIED (zero active integrations) |
| **AI provider integrations** | Not verified | ❓ NOT AUDITED (requires export) |
| **Data retention / deletion** | Not verified | ❓ NOT AUDITED (requires export) |
| **Backup / export controls** | Base44 platform | ❓ NOT AUDITED (requires export) |

---

## Data Privacy Considerations

| Data Category | Present in MindFlow? | Sensitivity | Notes |
|---------------|---------------------|-------------|-------|
| **Personal daily state** (energy, mood, stress, notes) | YES (DailyState) | **HIGH** | Intimate self-tracking data |
| **Task content** (titles, descriptions, dread flags) | YES (Task) | **MEDIUM-HIGH** | May reveal personal/work concerns |
| **Voice transcripts** (temporary) | YES (VoiceCapture) | **HIGH** | Auto-deletion specified; verify implementation |
| **Behavioral patterns** (friction, postponement, hyperfocus) | YES (FrictionPattern, TaskEvent) | **MEDIUM-HIGH** | Reveals cognitive/emotional patterns |
| **Location/context** | YES (DailyState) | **MEDIUM** | Contextual but not precise GPS |
| **User preferences** (sleep, work hours, automation perms) | YES (UserPreference) | **LOW-MEDIUM** | Configurational |
| **Project/goals** | YES (Project) | **LOW-MEDIUM** | Goal content may be sensitive |

**Key privacy principles for any future source integration:**
- No production user data in repository
- Synthetic fixtures only for demonstrations
- Schema/migrations without data
- Configuration templates without secrets
- `.gitignore` must exclude: `.env*`, `*.key`, `*.pem`, `secrets/`, `data/`, `*.db`, `*.sqlite`

---

## Credential & Secret Handling

| Item | Current Status | Required for Source Integration |
|------|----------------|--------------------------------|
| Base44 API tokens | Not in this repo | Must be excluded via `.gitignore` |
| AI provider API keys | Not in this repo | Must be excluded via `.gitignore` |
| Database credentials | Not in this repo (Base44-managed) | N/A — platform managed |
| Third-party service credentials | Not in this repo (0 connectors active) | Must be excluded via `.gitignore` |
| `PROGRAMS/login-Nex.txt` | **Excluded by path** — unrelated to MindFlow | N/A |

**No secret values have been read, exposed, or committed in this repository's preparation.**

---

## Repository Security Checklist (This Repo)

| Check | Status | Method |
|-------|--------|--------|
| No `.env`, `.env.*`, `secrets/`, `*.key`, `*.pem` files | ✅ PASS | File enumeration |
| No credentials in documentation | ✅ PASS | Content review |
| No production/user data | ✅ PASS | No data files present |
| No private identifiers (emails, phones, IDs) | ✅ PASS | Content review |
| No Base44 project credentials | ✅ PASS | Not present |
| Clear source limitation notice | ✅ PASS | README + this file |
| License appropriate for documentation | ✅ PASS | CC BY 4.0 declared |

---

## Platform Dependency Risk (Base44)

| Risk | Assessment | Mitigation |
|------|------------|------------|
| **Source export never available** | Medium | Document architecture thoroughly now; treat as design portfolio piece |
| **Exported source tightly coupled to Base44 runtime** | Medium | Audit immediately post-export; document coupling; consider adapter layer |
| **Platform deprecation / pricing change** | Low | Base44 is active; monitor; architecture is portable conceptually |
| **AI behavior depends on Base44-managed models** | Medium | Document prompt/logic structure separately; plan for model portability |

---

## Compliance Notes

| Regulation | Applicability | Current State |
|------------|---------------|---------------|
| **GDPR** (personal data) | HIGH — DailyState, VoiceCapture, TaskEvent contain personal data | No production data in repo; schema-only if source exported |
| **AI Act** (AI system transparency) | MEDIUM — AI behavior in automation rules | Not yet documented; requires export |
| **License compliance** (third-party assets) | UNKNOWN | Requires asset audit post-export |

---

## Incident Response (If Source Becomes Available)

1. **Immediately audit** exported source for secrets, credentials, private data
2. **Apply `.gitignore`** before any `git add`: `.env*`, `*.key`, `*.pem`, `secrets/`, `data/`, `*.db`, `*.sqlite`, `node_modules/`, `dist/`, `build/`, `.vercel`, `.netlify`
3. **Redact** any hardcoded configuration values that reference production resources
4. **Verify** no user records, session data, or production database exports included
5. **Document** any Base44-specific runtime dependencies
6. **Update** this SECURITY.md with findings
7. **Only then** consider GitHub publication with appropriate visibility

---

## Contact for Security Questions

Adriana Andreeva — contact via the repository owner's GitHub profile  
(No security.txt published; this is a public case-study documentation repository)