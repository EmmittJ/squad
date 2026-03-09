# Decisions

> Team decisions that all agents must respect. Managed by Scribe.

---

## Foundational Directives (carried from beta, updated for Mission Control)

### Type safety — strict mode non-negotiable
**By:** CONTROL (formerly Edie)
**What:** `strict: true`, `noUncheckedIndexedAccess: true`, no `@ts-ignore` allowed.
**Why:** Types are contracts. If it compiles, it works.

### Hook-based governance over prompt instructions
**By:** RETRO (formerly Baer)
**What:** Security, PII, and file-write guards are implemented via the hooks module, NOT prompt instructions.
**Why:** Prompts can be ignored. Hooks are code — they execute deterministically.

### Node.js >=20, ESM-only, streaming-first
**By:** GNC (formerly Fortier)
**What:** Runtime target is Node.js 20+. ESM-only. Async iterators over buffers.
**Why:** Modern Node.js features enable cleaner async patterns.

### Casting — Apollo 13, mission identity
**By:** Squad Coordinator
**What:** Team names drawn from Apollo 13 / NASA Mission Control. Scribe is always Scribe. Ralph is always Ralph. Previous universe (The Usual Suspects) retired to alumni.
**Why:** The team outgrew its original universe. Apollo 13 captures collaborative pressure, technical precision, and mission-critical coordination — perfect for an AI agent framework.

### Proposal-first workflow
**By:** Flight (formerly Keaton)
**What:** Meaningful changes require a proposal in `docs/proposals/` before execution.
**Why:** Proposals create alignment before code is written.

### Tone ceiling — always enforced
**By:** PAO (formerly McManus)
**What:** No hype, no hand-waving, no claims without citations.
**Why:** Trust is earned through accuracy, not enthusiasm.

### Zero-dependency scaffolding preserved
**By:** Network (formerly Rabin)
**What:** CLI remains thin. Zero runtime dependencies for the CLI scaffolding path.
**Why:** Users should be able to run `npx` without downloading a dependency tree.

### Merge driver for append-only files
**By:** Squad Coordinator
**What:** `.gitattributes` uses `merge=union` for `.squad/decisions.md`, `agents/*/history.md`, `log/**`, `orchestration-log/**`.
**Why:** Enables conflict-free merging of team state across branches.

### Interactive Shell as Primary UX
**By:** Brady
**What:** Squad becomes its own interactive CLI shell. `squad` with no args enters a REPL.
**Why:** Squad needs to own the full interactive experience.

### No temp/memory files in repo root
**By:** Brady
**What:** No plan files, memory files, or tracking artifacts in the repository root.
**Why:** Keep the repo clean.

---

## Sprint Directives

### Secret handling — agents must never persist secrets
**By:** RETRO (formerly Baer), v0.8.24
**What:** Agents must NEVER write secrets, API keys, tokens, or credentials into conversational history, commit messages, logs, or any persisted file. Acknowledge receipt without echoing values.
**Why:** Secrets in logs or history are a security incident waiting to happen.

### Test assertion discipline — mandatory
**By:** FIDO (formerly Hockney), v0.8.24
**What:** All code agents must update tests when changing APIs. FIDO has PR blocking authority on quality grounds.
**Why:** APIs changed without test updates caused CI failures and blocked external contributors.

### Docs-test sync — mandatory
**By:** PAO (formerly McManus), v0.8.24
**What:** New docs pages require corresponding test assertion updates in the same commit.
**Why:** Stale test assertions block CI and frustrate contributors.

### Contributor recognition — every release
**By:** PAO, v0.8.24
**What:** Each release includes an update to the Contributors Guide page.
**Why:** No contribution goes unappreciated.

### API-test sync cross-check
**By:** FIDO + Booster, v0.8.24
**What:** Booster adds CI check for stale test assertions. FIDO enforces via PR review.
**Why:** Prevents the pattern of APIs changing without test updates.

### Doc-impact review — every PR
**By:** PAO, v0.8.25
**What:** Every PR must be evaluated for documentation impact. PAO reviews PRs for missing or outdated docs.
**Why:** Code changes without doc updates lead to stale guides and confused users.

---

## Release v0.8.24

### CLI Packaging Smoke Test: Release Gate Decision
**By:** FIDO, v0.8.24  
**Date:** 2026-03-08

The CLI packaging smoke test is APPROVED as the quality gate for npm releases.

**What:**
- npm pack → creates tarball of both squad-sdk and squad-cli
- npm install → installs in clean temp directory (simulates user install)
- node {cli-entry.js} → invokes 27 commands + 3 aliases through installed package
- Coverage: All 26 primary commands + 3 of 4 aliases (watch, workstreams, remote-control)

**Why:** Catches broken package.json exports, MODULE_NOT_FOUND errors, ESM resolution failures, command routing regressions — the exact failure modes we've shipped before.

**Gaps (acceptable):**
- Semantic validation not covered (only routing tested)
- Cross-platform gaps (test runs on ubuntu-latest only)
- Optional dependencies allowed to fail (node-pty)

**Result:** ✅ GO — v0.8.24 release approved. 32/32 tests pass.

---

### CLI Release Readiness Audit — v0.8.24
**By:** EECOM  
**Date:** 2026-03-08

Definitive CLI completeness audit confirms all commands work post-publish.

**What:**
- 26 primary commands routed, all tested ✅
- 4 aliases routed (watch, workstreams, remote-control, streams) — 3 tested, 1 untested
- Tarball: 318 files, bin entry correct, postinstall script functional
- ESM runtime patch verified for Node 24+ compatibility
- All tests pass: 32/32 (36s runtime)

**Gaps (non-blocking):**
- `streams` alias routed but not smoke-tested (same code path as tested `subsquads` — low risk)

**Result:** ✅ SHIP IT — 95% confidence. CLI production-ready for v0.8.24.

---

## Sprint: M3-3 Skill Script Handlers

### Handler Type System Design — Disjoint Tool Name Enforcement
**By:** CONTROL  
**Date:** 2026-03-09

The skill handler type system (`packages/squad-sdk/src/skills/handler-types.ts`) uses compile-time type assertions to enforce disjoint tool names across concern handler interfaces.

**What:**
- `OwnKeys<T>` strips HandlerLifecycle keys, keeps only tool-name keys
- `AssertDisjoint<A, B>` returns `true | never` — true if A and B share no tool names, never otherwise
- All 6 pairs explicitly checked: TaskHandlers, DecisionHandlers, MemoryHandlers, LogHandlers
- Compile fails if any concern pair shares a tool name

**Why:** Without this check, adding duplicate tool names (e.g., `squad_list` in both TaskHandlers and DecisionHandlers) would silently compile but break `resolveHandler()` at runtime. The type system catches this class of error during development with zero runtime overhead.

**Impact:** Prevents runtime handler resolution bugs at compile time. Self-documenting — assertion types live alongside handler interfaces they protect.

---

### Skill Script Loader Implementation
**By:** EECOM  
**Date:** 2026-03-09

Implemented SkillScriptLoader class in `packages/squad-sdk/src/skills/skill-script-loader.ts` for dynamic loading of executable skill handlers from backend skill directories (`.squad/skills/{name}/scripts/`). Added `ToolRegistry.applySkillHandlers()` method to replace built-in tool handlers with skill-backed versions.

**What:**
1. **Tool Name → Script Name Mapping:** Tool names follow `squad_{operation}` convention. Script filenames strip the `squad_` prefix: `squad_create_issue` → `create_issue.js`.
2. **Windows Path Normalization:** Dynamic imports use `pathToFileURL()` but require path separator normalization BEFORE conversion. Backslashes normalized to forward slashes.
3. **Path Containment Enforcement:** `resolveSkillPath()` ensures resolved paths stay within projectRoot or teamRoot. Throws Error on escape attempts.
4. **Partial Implementation Support:** Missing handler scripts silently skipped. Invalid exports (module.default not a function) fatal. No scripts/ directory → returns null.
5. **Handler Signature Bridge:** `wrapSkillHandler()` bridges SkillHandler signature `(args, config)` to SquadToolHandler signature `(args, invocation)`.
6. **Lifecycle Hooks:** Optional `scripts/lifecycle.js` can export `init(config)` and `dispose()` functions.
7. **ToolRegistry Integration:** `applySkillHandlers()` replaces tool handlers in registry's internal Map. Only affects pre-existing tools — unknown names silently ignored.

**Why:** Core of the skill-script model — backend skills can replace built-in tool implementations with custom logic. Design prioritizes security (path containment), partial implementations, and Windows compatibility. ToolRegistry remains single source of truth.

---

### Skill Script Loader Test Design
**By:** FIDO  
**Date:** 2026-03-09

Test fixture pattern for SkillScriptLoader validation: use real temporary script files with dynamic import() rather than mocks or stubs.

**What:**
- Create unique temp directory per test
- Write valid handler scripts to temp filesystem
- Load with SkillScriptLoader
- Cleanup with fs.rmSync

**Why:** 
1. **Import Semantics:** dynamic import() requires actual filesystem paths. Cannot mock module resolution without significant overhead.
2. **Windows Path Validation:** Path normalization bugs (backslash vs forward slash) only surface with real file URLs. pathToFileURL() behavior differs across separators.
3. **Module Cache Testing:** Script reloading behavior can only be validated with real import() calls.

**Pattern Approved:**
- 33 tests, all passing (372ms)
- Validates end-to-end script loading, catches path bugs, no mocking complexity
- Trade-off: slower (~920ms for 33 tests) but realistic
- Approved for other skill-related test files

---

*Fresh start — Mission Control rebirth, 2026-03-08. Previous decisions archived.*
