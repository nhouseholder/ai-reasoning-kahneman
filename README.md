# AI Reasoning — Kahneman Dual Mode Framework

A dual-mode reasoning framework for AI coding agents, based on Daniel Kahneman's System 1 / System 2 theory from *Thinking, Fast and Slow*.

## Core Principle

**Default to Fast. Escalate to Slow.**

The brain uses two systems for thinking. AI agents should too:

| | **System 1 (Fast)** | **System 2 (Slow)** |
|---|---|---|
| **Mode** | Automatic, pattern-matching | Deliberate, sequential |
| **Cost** | Low tokens, single-shot | High tokens, multi-step |
| **Tasks** | Edits, renames, commands, CRUD, cosmetics, lookups | Architecture, debugging, planning, security, refactoring |
| **Failure** | WYSIATI — ignores missing context | Analysis paralysis — token exhaustion |

## System 1: Fast Mode

**The default.** Route directly, execute, verify, done.

### When to Use
- Single-file edits with clear scope
- Renames, formatting, boilerplate
- Running known commands
- CRUD operations
- Cosmetic changes
- Trivial lookups (`grep`, `ls`, `git status`)
- Executing an existing plan

### Processing Flow
```
Pattern match → Route to agent → Execute → Quick verify (LSP + scope) → Done
```

### Failure Mode: WYSIATI
System 1 jumps to conclusions based on available data, ignoring what's missing. It solves the *easier* version of the problem (attribute substitution). Guard: if anything feels off → escalate to System 2.

## System 2: Slow Mode

**Triggered, not default.** Research, plan, execute, verify, self-correct.

### Handoff Triggers (System 1 → System 2)

Based on Kahneman's 5 signals for System 2 activation:

| Trigger | Signal | Example |
|---|---|---|
| **Difficulty** | No stored pattern for this task | "Build a real-time collaboration engine" |
| **Surprise** | Tool failure, unexpected output, test breakage | Edit produces different result than expected |
| **Error** | LSP errors, low confidence, user correction | Fix attempt doesn't resolve the issue |
| **Strain** | Ambiguous scope, 2+ valid approaches, high-stakes domain | "Add auth" — JWT vs sessions vs OAuth |
| **Explicit** | User says "plan this", "think through", "should we" | Any request for deliberation |

### Processing Flow
```
Research (specs, memory, related files)
  → Plan (2-3 approaches, trade-offs)
  → Execute (sequential, progressive disclosure)
  → Verify (tests, LSP, constraint check)
  → WYSIATI guard ("What have I not examined?")
  → Self-correct if needed
```

### WYSIATI Guard (MANDATORY before committing)
Before System 2 claims completion:
1. What files, dependencies, or constraints have I not yet examined?
2. Does my solution actually satisfy all original constraints?
3. What edge cases am I blind to because I haven't seen them?

## Cognitive Load Management

- **Token budgets per phase** — Don't dump entire codebase into one prompt
- **Session limits** — Long System 2 sessions degrade in quality → handoff to fresh instance
- **Progressive disclosure** — Read only what's needed for the current step
- **Context compaction** — At 60% context, write handoff and start fresh session

## Skill Compilation

When System 2 successfully solves a novel complex problem, cache the pattern as a reusable skill. Next time, System 1 can execute it directly. This is how agents develop expertise.

## Anti-Patterns

| Anti-Pattern | Cause | Fix |
|---|---|---|
| **Overthinking** | System 2 activated for System 1 tasks | Trust the triggers — if none fire, stay fast |
| **Underthinking** | System 1 handles System 2 tasks | WYSIATI guard — "have I seen this before?" |
| **Analysis paralysis** | System 2 runs too many reasoning cycles | Single-pass: think once, challenge once, act |
| **Context exhaustion** | System 2 session runs too long | Handoff at 60% context, fresh session |
| **Attribute substitution** | Solving easier proxy problem | Re-read original request before claiming done |

## Implementation

This framework is implemented in the 10-agent orchestration system:
- `agents/orchestrator.md` — System 1 routing + System 2 triggers
- `agents/generalist.md` — Mid-task escalation protocol
- `agents/_shared/memory-systems.md` — Confidence gate (signal-based verification)

## License

MIT
