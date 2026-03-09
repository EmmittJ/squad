# Orchestration Log: Edie — Skill System Type Review

**Date:** 2026-03-05
**Agent:** Edie (TypeScript Engineer)
**Task:** Type system review of `docs/proposals/skill-system-design.md`
**Verdict:** Sound, 1 must-fix (disjoint key assertion)

## Scope

Type-level correctness of `ConcernMap`, `LoadResult<C>`, handler hierarchy, index signatures, `defineHandler` pattern, `HandlerRegistration` generic bounds.

## Outcome

- Type system design structurally sound
- `ConcernMap` indexed-access pattern and `LoadResult<C>` generic well-designed
- 1 real bug: `AllHandlers` intersection does NOT enforce disjoint key invariant
- `defineHandler<TArgs>()` pattern correct (same as Vite `defineConfig`, Vue `defineComponent`)

## Must-Fix

- `AllHandlers` intersection silently merges handler signatures if two concern interfaces share a tool name key. Ship with compile-time `AssertDisjoint<A, B>` assertion using `Extract<keyof A, keyof B> extends never`. Strip `keyof HandlerLifecycle` (shared intentionally). Zero runtime cost.

## Accepted

- `[key: string]: unknown` on `*Args` — acceptable for v1 (document destructure-not-iterate pattern)
- `SkillConfig` index signature with `package?: never` trick — sound, compiles correctly
- `HandlerRegistration<H>` generic bound — acceptable, `TrackingConfig` already binds correct type
- `defineHandler` return type — `SkillHandler<TArgs>` is already narrowest useful type

## Decision Written

`decisions/inbox/edie-skill-system-review.md` → merged to `decisions.md`
