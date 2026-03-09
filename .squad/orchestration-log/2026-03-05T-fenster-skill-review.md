# Orchestration Log: Fenster — Skill System Runtime Review

**Date:** 2026-03-05
**Agent:** Fenster (Core Dev)
**Task:** Runtime feasibility review of `docs/proposals/skill-system-design.md`
**Verdict:** Needs work then ship (4 blockers)

## Scope

Implementation feasibility from runtime perspective — ToolRegistry integration, async loading, config types, signal handlers, module cache behavior.

## Outcome

- Design fundamentally sound — right abstraction level, compatible with existing ToolRegistry
- 4 blockers identified, all with straightforward fixes
- 2 should-fix items (handler signature mismatch, module cache isolation)

## Blockers

1. Two `SquadConfig` types — `config/schema.ts` vs `runtime/config.ts` both export `SquadConfig`. `tracking` goes on runtime SquadConfig.
2. ToolRegistry constructor is sync — async `import()` for skills requires separate `async applySkillHandlers()` method.
3. Async signal handler gap — existing handlers are sync, `disposeAll()` is async. Need `shuttingDown` flag pattern.
4. No handler-per-tool override — ToolRegistry bundles schema+handler. Need to either restructure or create new SquadTool entries with swapped handler.

## Should-Fix

5. `SkillHandler` vs `SquadToolHandler` signature mismatch — wrapper location TBD
6. Module cache isolation — Windows junction points resolve differently than symlinks

## Decision Written

`decisions/inbox/fenster-skill-system-review.md` → merged to `decisions.md`
