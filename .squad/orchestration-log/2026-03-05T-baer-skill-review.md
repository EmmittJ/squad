# Orchestration Log: Baer — Skill System Security Review

**Date:** 2026-03-05
**Agent:** Baer (Security)
**Task:** Security review of `docs/proposals/skill-system-design.md`
**Verdict:** Proceed with mitigations (1 HIGH, 2 MEDIUM)

## Scope

Security posture of the skill system — hook pipeline integration, path containment, secret scoping, subprocess trust model, env var expansion, dispose timeout.

## Outcome

- 1 HIGH finding: HookPipeline not wired into ToolRegistry dispatch
- 2 MEDIUM findings: import path containment limits, init() secret scoping
- 4 accepted risks (dynamic import trust, module cache, env var expansion, dispose timeout)

## Findings

1. **HIGH — HookPipeline not wired.** `HookPipeline` exists but `ToolRegistry` has zero references to it. Skill handlers bypass all governance hooks (file-write guards, PII scrubbing, shell restrictions, reviewer lockout). Must wire pre/post hooks before or co-requisite to skill system.
2. **MEDIUM — scripts/lib/ imports not path-contained.** Path containment validates the loader, not the loaded code. Handler scripts can `import()` outside skill directory. Accept as documented risk.
3. **MEDIUM — init(config) secret scoping.** Multi-concern skill sees expanded secrets from all concerns across init() calls. Verify each concern's init() only receives its own config.

## Required Before Merge

1. Wire HookPipeline into ToolRegistry dispatch (pre-tool and post-tool hooks)
2. Marketplace security rule: audit scripts for external imports
3. Authoring guide: handlers MUST NOT spread args into CLI/SQL/APIs

## Decision Written

`decisions/inbox/baer-skill-system-review.md` → merged to `decisions.md`
