# Orchestration Log: Kujan — Skill System SDK Surface Review

**Date:** 2026-03-05
**Agent:** Kujan (SDK Expert)
**Task:** SDK surface and platform review of `docs/proposals/skill-system-design.md`
**Verdict:** Sound, 1 fix required (new subpath export)

## Scope

SDK export surface, platform compatibility (§11), adapter/session interaction, `defineHandler` pattern, subpath exports infrastructure, runtime vs. authoring dependency claim.

## Outcome

- SDK design sound — `SkillHandler`/`SquadToolHandler` split clean, adapter boundary correct
- 1 required fix: new `./skills/backend` subpath export for 25+ backend authoring types
- Platform availability table honest, 1 correction needed (@copilot agent → "best-effort")
- `defineHandler` pattern correct (same as Vite/Vue/Nuxt)
- `wrapSkillHandler` bridging correct — skill handlers don't need session IDs

## Required Fix

Add `./skills/backend` subpath export:

- `./skills` — existing prompt-only skill system (unchanged)
- `./skills/backend` — new: `defineHandler`, `validateSkill`, all `*Args`, all `*Handlers`, `SkillHandler`, `HandlerLifecycle`, config types
- `.` root — re-export only `defineHandler` and `TrackingConfig` (what `squad.config.ts` needs)

## Platform Correction

@copilot coding agent: Change from "✅ Terminal execution" to "best-effort instruction following" — agent reads SKILL.md and runs scripts via terminal, not ToolRegistry integration.

## Decision Written

`decisions/inbox/kujan-skill-system-review.md` → merged to `decisions.md`
