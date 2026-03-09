# FIDO

> Flight Dynamics Officer

## Learnings

### Name-Agnostic Testing Pattern
Tests reading live .squad/ files (team.md, routing.md) must assert structure/behavior, not specific agent names. Names change during team rebirths. Two classes of tests: (1) live-file tests — must survive rebirths, use property checks, (2) inline-fixture tests — self-contained, can hardcode names. See ralph-triage.test.ts for the canonical pattern.

### Dynamic Content Discovery
Blog tests in docs-build.test.ts use filesystem discovery (readdirSync) instead of hardcoded arrays. Adding/removing blog posts no longer requires updating the test. Pattern: discover from disk, sort, validate build output exists.

### Test Baseline (Post-v0.8.24)
3,931 tests passing, 46 todo, 5 skipped, 149 test files (~89s). Only failure: aspire-integration.test.ts (needs Docker daemon). Speed gates in speed-gates.test.ts enforce UX budgets (help output <100 lines, init <10s, etc.).

### Command Wiring Regression Test
cli-command-wiring.test.ts prevents the "unwired command" bug: verifies every .ts file in packages/squad-cli/src/cli/commands/ is imported in cli-entry.ts. Bidirectional validation — also checks that every import points to an existing file.

### CLI Packaging Smoke Test (v0.8.24 Release Assessment)
cli-packaging-smoke.test.ts validates the PACKAGED CLI artifact (npm pack → install → execute). Tests 27 commands + 3 aliases by invoking them through the installed tarball. Catches: missing imports (MODULE_NOT_FOUND), broken package.json exports, bin script misconfiguration, and ESM resolution failures. Gate runs before both SDK and CLI publish jobs. Complements cli-command-wiring.test.ts (source-level import verification) by testing the artifact users actually download. All 32 tests passing (37s runtime). Approved for v0.8.24 release gate.

📌 **Team update (2026-03-08T21:18:00Z):** FIDO + EECOM released unanimous GO verdict for v0.8.24. Smoke test approved as release gate. FIDO confirmed 32/32 pass + publish.yml wired correctly. EECOM confirmed 26/26 commands + packaging complete (minor gap: "streams" alias untested, non-blocking).

### Skill Script Loader Testing — Real Fixtures, No Mocks (2026-03-09)

skill-script-loader.test.ts validates SkillScriptLoader, ToolRegistry.applySkillHandlers(), and resolveSkillPath(). **33 tests, all passing (372ms).**

**Test Coverage:**
- Null return conditions (markdown fallback): 2 tests
- Successful load variations: 6 tests
- Lifecycle hooks (init/dispose): 6 tests
- Error cases (non-function exports, bad paths): 3 tests
- Schema lookup filtering: 3 tests
- ToolRegistry.applySkillHandlers (handler replacement): 5 tests
- resolveSkillPath (path resolution + containment): 8 tests

**Key Pattern: Real Temp Fixtures**
- Create unique temp directory per test (test-fixtures/skill-loader-{randomUUID}/)
- Write actual .js handler scripts to filesystem
- Load with SkillScriptLoader via dynamic import()
- Cleanup with fs.rmSync({recursive: true})

**Why Real Files:**
1. Import semantics — dynamic import() requires actual paths, no mock equivalent
2. Windows path validation — backslash vs forward slash bugs only surface with real file URLs
3. Module cache testing — script reloading behavior only testable with real import() calls

**Trade-offs:** Slower than in-memory mocks (~920ms for 33 tests vs ~50ms theoretical), but validates end-to-end reality. Pattern approved for other skill-related test files.

📌 **Team update (2026-03-09T15:05:20Z):** M3-3 skill-script sprint complete. FIDO wrote comprehensive test suite (33 tests, all passing). Real fixture pattern approved for reuse. Depends on CONTROL's handler-types.ts and EECOM's SkillScriptLoader. PR #1 open. — Emmitt requested.

