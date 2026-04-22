# Handoff — Kahneman AI Reasoning Framework v2.0

**Date:** 2026-04-22
**Version:** v2.0
**Status:** Framework complete, documentation thorough, runtime enforcement partial

## What This Is

Portable, operationally enforceable reasoning framework for AI agents. Based on Kahneman's *Thinking, Fast and Slow* and frontier LLM patterns (Claude Opus, o1/o3, Gemini 2.5 Pro).

**Core innovation: 3 tiers, not 2.**

| Tier | Name | Evidence Budget | When |
|------|------|----------------|------|
| 1 | FAST | 0 pulls | Pattern match, single-file edit |
| 2 | DELIBERATE | 1 pull | One unknown, quick check |
| 3 | SLOW | 3 pulls | Architecture, debugging, planning |

## What's Implemented

- Complete framework spec (SPEC.md)
- 3-tier reasoning contract (core/reasoning-contract.md)
- Mode state machine with transitions (core/mode-state-machine.md)
- Evidence budget rules (core/evidence-budget.md)
- Think tool JSON schema (core/think-tool-schema.md)
- Completion gate checklist (core/completion-gate.md)
- Pattern library (fast/deliberate/slow/pre-mortem/outside-view/WYSIATI)
- Integration guides (OpenCode, Claude Code, LangChain, AutoGen)
- Drop-in templates (agent footer, orchestrator routing, think tool prompt)
- Example configs (minimal, multi-agent, council)

## What's Partial

- Think tool is documented schema, not runtime-validated (enforcement is honor-system)
- Evidence counter is manual, not automatic
- Mode transitions rely on agent self-reporting
- Not yet integrated into active prompts in downstream repos

## Integration Status

| Repo | Integration | Status |
|---|---|---|
| 8-agent-team | _shared/cognitive-kernel.md | Active — 3-tier model in production |
| 8-agent-team | scripts/validate-think-tool.js | Added v1.8.0 — lint-time enforcement |
| persistent-brain | N/A | Framework-agnostic, no direct integration |
| Other projects | templates/ | Available, not yet deployed |

## Key Files

| File | Purpose |
|---|---|
| SPEC.md | Full technical specification |
| core/reasoning-contract.md | 3-tier mode contract |
| core/mode-state-machine.md | State transitions, signals, terminals |
| core/evidence-budget.md | Countable rules per mode |
| core/think-tool-schema.md | JSON schema for structured reasoning |
| core/completion-gate.md | Mode compliance check |
| patterns/*.md | Fast, deliberate, slow, pre-mortem, outside-view, WYSIATI |
| integrations/*.md | OpenCode, Claude Code, LangChain, AutoGen |
| templates/*.md | Drop-in agent footer, orchestrator routing, think tool prompt |
| examples/*.json | Minimal, multi-agent, council configs |

## Known Gotchas

- Framework is documentation-heavy — runtime enforcement requires downstream integration
- 8-agent-team has the most complete integration (cognitive-kernel.md + validator)
- Think tool schema v2.0 in this repo differs slightly from 8-agent-team's embedded version (6-phase vs 7-phase slow mode) — sync needed
- DELIBERATE mode is new — agents may need calibration to use it correctly

## Next Steps

1. **Runtime enforcement** — Build automatic evidence counter (not honor-system)
2. **Downstream sync** — Ensure 8-agent-team cognitive-kernel.md stays in sync with this repo
3. **More integrations** — Add Cursor, Gemini CLI, Cline adapters
4. **Calibration data** — Collect empirical data on which tasks need which mode
5. **Council integration** — Think tool in council debate for structured arbitration

## Repo

https://github.com/nhouseholder/ai-reasoning-kahneman