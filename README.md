# Kahneman AI Reasoning Framework v2.0

A **portable, operationally enforceable** reasoning framework for AI agents, based on Daniel Kahneman's *Thinking, Fast and Slow* and practical patterns from frontier LLMs (Claude Opus, OpenAI o1/o3, Gemini 2.5 Pro).

## Why This Exists

Most agent systems use binary fast/slow reasoning. This creates a false dichotomy:
- **FAST** is too shallow for tasks with one unknown
- **SLOW** is overkill for tasks needing a quick sanity check

**The solution: 3 tiers, not 2.**

| Tier | Name | Evidence Budget | When |
|------|------|----------------|------|
| 1 | **FAST** | 0 pulls | Pattern match, single-file edit, rename |
| 2 | **DELIBERATE** | 1 pull | One unknown, one ambiguity, quick check |
| 3 | **SLOW** | 3 pulls | Architecture, debugging, planning, security |

## Core Innovations

1. **Countable evidence budget** — Every mode has a hard limit on tool calls. Breach = mandatory escalation.
2. **Think tool schema** — Structured scratchpad (JSON) replaces free-text chain-of-thought. Validatable, not hallucinated.
3. **Meta-cognitive feedback** — Agents self-evaluate whether their mode choice was justified. Enables empirical calibration.
4. **Skill compilation** — Successful SLOW patterns graduate to FAST via memory caching. The system gets faster over time.
5. **Pre-mortem + outside view** — From Kahneman: imagine failure before acting; anchor on base rates before specifics.

## Quick Start

### For OpenCode Users
Copy `integrations/opencode.md` into your agent prompts. Add the think tool to your agent config.

### For Claude Code Users
Copy `integrations/claude-code.md` into your `.claude/CLAUDE.md`.

### For LangChain/AutoGen Users
See `integrations/langchain.md` and `integrations/autogen.md` for adapter patterns.

### Minimal Example
```json
{
  "agent": {
    "my-agent": {
      "mode": "all",
      "prompt_file": "templates/agent-prompt-footer.md"
    }
  }
}
```

## File Structure

```
├── README.md                    # This file
├── SPEC.md                      # Full technical specification
├── core/
│   ├── reasoning-contract.md    # The 3-tier mode contract
│   ├── mode-state-machine.md    # State transitions, signals, terminal states
│   ├── evidence-budget.md       # Countable rules per mode
│   ├── think-tool-schema.md     # JSON schema for structured reasoning
│   └── completion-gate.md       # Mode compliance check
├── patterns/
│   ├── fast-mode.md             # Pattern-matching, single-pass
│   ├── deliberate-mode.md       # Bounded check, 1-pull limit
│   ├── slow-mode.md             # 6-phase pipeline with hard stops
│   ├── pre-mortem.md            # Failure imagination protocol
│   ├── outside-view.md          # Base-rate-first estimation
│   └── wysiati-guard.md         # Anti-confirmation check
├── integrations/
│   ├── opencode.md              # OpenCode wiring
│   ├── claude-code.md           # Claude Code wiring
│   ├── langchain.md             # LangChain adapter
│   └── autogen.md               # AutoGen adapter
├── templates/
│   ├── agent-prompt-footer.md   # Drop-in footer for any agent prompt
│   ├── orchestrator-routing.md  # Mode classification + routing layer
│   └── think-tool-prompt.md     # Inject into tool definitions
└── examples/
    ├── minimal.json             # Single-agent config
    ├── multi-agent.json         # 3-agent team with mode negotiation
    └── council-debate.json      # Council using think tool
```

## How It Works

### 1. Mode Declaration
Every task starts with:
```
MODE: [fast|deliberate|slow]
JUSTIFICATION: [1 sentence — why this mode?]
```

### 2. Evidence Budget
- **FAST**: 0 additional evidence pulls (pattern only)
- **DELIBERATE**: 1 pull max (read one file, run one search)
- **SLOW**: 3 pulls max (anchor + 3, then terminal state)

One pull = one tool call returning new information.

### 3. Think Tool (DELIBERATE and SLOW modes)
```
THINK_TOOL:
  mode: [deliberate|slow]
  gist: [1-sentence decision]
  evidence_log:
    - pull_1: [tool_call] → [finding]
  disconfirmer: [one competing explanation]
  pre_mortem: [2-3 reasons this could fail]
  wysiati: [what's missing?]
  decision: [chosen approach]
  terminal: [done|ask|escalate]
  reflection: [was this mode justified?]
```

### 4. Terminal States
All modes end in one of:
- `done` — task complete, verified
- `ask` — need user input
- `escalate` — mode budget exhausted or fatal flaw found

### 5. Skill Compilation
After successful SLOW mode:
```
SAVE: topic_key="reasoning/pattern-name"
CONTENT: What, Why, Where, Learned
```
Next time, FAST mode finds it via memory query and executes directly.

## Key Differences from "Think Step by Step"

| Feature | Basic CoT | This Framework |
|--------|-----------|----------------|
| Mode awareness | None | Agent knows its tier and budget |
| Enforceable limits | None | Evidence pull counter + hard stops |
| Meta-cognition | None | Self-evaluates if mode was justified |
| Calibration | None | Learns which tasks need which mode |
| Pre-mortem | None | Imagines failure before acting |
| Integration | Ad-hoc | Drop-in templates for major frameworks |

## Sources

- Kahneman, D. (2011). *Thinking, Fast and Slow*. Farrar, Straus and Giroux.
- Anthropic. (2025). [The "think" tool](https://www.anthropic.com/engineering/claude-think-tool).
- OpenAI. (2024). [Learning to Reason with LLMs](https://openai.com/index/learning-to-reason-with-llms/).
- Snell et al. (2024). [Scaling LLM Test-Time Compute Optimally](https://arxiv.org/abs/2408.03314).

## License

MIT
