# Session Log: 5-Reviewer Skill System Design Review

**Date:** 2026-03-05
**Document:** docs/proposals/skill-system-design.md (Issue #162)
**Type:** Architecture review (design-phase, pre-implementation)

## Who Worked

| Reviewer | Role                | Scope                    |
| -------- | ------------------- | ------------------------ |
| Keaton   | Lead                | Full architecture review |
| Fenster  | Core Dev            | Runtime feasibility      |
| Edie     | TypeScript Engineer | Type system review       |
| Baer     | Security            | Security review          |
| Kujan    | SDK Expert          | SDK surface review       |

## Verdicts

- **Keaton:** Approve with changes (5 required items)
- **Fenster:** Needs work then ship (4 blockers)
- **Edie:** Sound, 1 must-fix (disjoint key assertion)
- **Baer:** Proceed with mitigations (1 HIGH: HookPipeline not wired, 2 MEDIUM)
- **Kujan:** Sound, 1 fix required (new `./skills/backend` subpath export)

## Key Findings

### Must-Fix Before Implementation

1. **HookPipeline not wired into ToolRegistry** (Baer, HIGH) — skill handlers bypass all governance hooks
2. **Disjoint key invariant not enforced** (Edie) — `AllHandlers` intersection silently merges on key collision
3. **Dual SquadConfig types** (Fenster) — `config/schema.ts` vs `runtime/config.ts` — must resolve before adding `tracking`
4. **ToolRegistry constructor is sync** (Fenster) — skill loading requires async `import()`, need `async applySkillHandlers()`
5. **No handler-per-tool override mechanism** (Fenster) — ToolRegistry bundles schema+handler in SquadTool objects
6. **Missing runtime result validator** (Keaton) — handler returning wrong shape silently corrupts agent response
7. **Config validation punted to v2** (Keaton) — should be v1, validated in `init()`

### Required Design Changes

- Remove `[key: string]: unknown` index signatures from `*Args` interfaces (Keaton)
- Lazy loading as default, not benchmark-gated (Keaton)
- Deprecation timeline for `squad_memory` → `squad_create_memory` rename (Keaton)
- Async signal handler refactor: `shuttingDown` flag + `disposeAll().then(exit)` (Fenster)
- New `./skills/backend` subpath export for authoring types (Kujan)
- @copilot platform table: "best-effort instruction following" not "✅ supported" (Kujan)

### Accepted Risks

- `scripts/lib/` imports not path-contained (inherent to Node.js `import()`) — document in authoring guide
- Dynamic `import()` trust model same as Husky hooks / GH Actions (accepted)
- ESM module cache immutability (script changes require restart) — document prominently

## Decisions Made

5 decisions merged into `.squad/decisions.md` under 2026-03-05 section. No duplicates with existing decisions. No overlapping decisions requiring consolidation.

## Sequencing Recommendation (Keaton)

1. Phase 1: Config schema + SkillScriptLoader + path containment
2. Phase 2: ToolRegistry handler wiring + wrapSkillHandler
3. Phase 3: Lifecycle management (init/dispose, rollback, signal handlers)
4. Phase 4: squad doctor extensions + env var expansion
5. Phase 5: First backend skill (GitHub Issues) as proof of concept
