# Session Log: Platform Adapter Retro
**Date:** 2026-03-09  
**Duration:** Parallel retro (3 agents)  
**Topic:** Platform adapter architecture retrospective  

## Participants
- **Flight** (lead) — facilitation, architectural verdict
- **EECOM** (core dev) — runtime analysis, pattern evaluation
- **CAPCOM** (SDK expert) — MCP boundary, capability discovery

## Verdict
**DEPRECATE platform adapters** — zero runtime callers, MCP provides superior abstraction.

## Key Decisions
1. **Adapters are unused** — no production code calls adapter methods (coordinator, agents, Ralph all use CLI/MCP directly)
2. **MCP is the answer** — async transport, remote capability, ecosystem reuse (adapters duplicate this at lower quality)
3. **Keep lightweight platform code** — `detectPlatform()`, `getRalphScanCommands()`, `PlatformType` survive
4. **Capability discovery needed** — agents need to declare required MCP tools; Squad needs `getAvailableTools()` API
5. **Execution plan** — deprecate v0.9.0, remove v1.0.0, enhance MCP integration

## Outcome
- 3 comprehensive decision documents written to `.squad/decisions/inbox/`
- 3 orchestration logs created for archive
- All agents in agreement: adapters were the wrong abstraction for an AI-native system
- Next phase: implement MCP capability discovery, deprecate adapters, update agent charters and documentation

## Root Cause
Timeline mismatch between SDK design (traditional programmatic API) and actual deployment (AI agents with tool-based access). Early design assumed SDK consumers would call `adapter.listWorkItems()` — that pattern never materialized because Copilot sessions give agents direct tool access.

---

**Filed by:** Scribe  
**Timestamp:** 2026-03-09T00-00-36Z
