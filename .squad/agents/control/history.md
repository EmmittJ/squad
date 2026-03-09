# CONTROL — Project Knowledge

> Persistent learnings and context for CONTROL, the TypeScript Engineer.

---

## Learnings

### Skill Handler Type System — handler-types.ts (2026-03-09)

Created `packages/squad-sdk/src/skills/handler-types.ts` — the complete type foundation for the skill-script model.

**What:**
- 11 tool argument interfaces (CreateIssueArgs, UpdateIssueArgs, etc.) — all include `[key: string]: unknown` for skill-documented extensions
- Core handler types: SkillHandler<TArgs>, HandlerLifecycle (init/dispose hooks)
- 4 concern handler interfaces: TaskHandlers, DecisionHandlers, MemoryHandlers, LogHandlers
- Compile-time disjoint assertion for tool names across concerns (6 pair-wise checks)
- ConcernMap, LoadResult, BackendRef, SkillConfig, TrackingConfig
- defineHandler<TArgs>() identity function for type inference in skill author code

**Key Design Choices:**
- Disjoint assertion uses type-level proof: OwnKeys<T> strips lifecycle keys, AssertDisjoint<A,B> ensures no tool name collisions
- _disjointProof const is internal (not exported) — exists only for compile-time validation
- All types exported except the proof const — this is a pure type module
- defineHandler() is a zero-overhead identity function — runtime is just `return handler;`
- SkillConfig has `package?: never` to prevent future package key conflicts

**Why It Matters:**
- Provides type safety for skill authors writing TypeScript handlers
- Catches tool name collisions at compile time (before runtime resolveHandler() failures)
- Feeds EECOM's SkillScriptLoader implementation and FIDO's comprehensive test suite

📌 **Team update (2026-03-09T15:05:20Z):** M3-3 skill-script sprint complete. CONTROL designed handler type system with disjoint tool-name enforcement. EECOM implemented SkillScriptLoader + ToolRegistry.applySkillHandlers(). FIDO wrote 33-test suite, all passing. Decisions merged into .squad/decisions.md. PR #1 open on skill-script-loader branch. — Emmitt requested.
- Enables IDE autocomplete and type checking in skill scripts
- No runtime dependency — compiled JS handlers are plain functions
