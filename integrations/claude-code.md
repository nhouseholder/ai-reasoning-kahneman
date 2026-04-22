## Claude Code Integration

Drop-in integration for Claude Code.

### 1. Add to CLAUDE.md

Copy the contents of `templates/agent-prompt-footer.md` into your `~/.claude/CLAUDE.md` or project-specific `CLAUDE.md`.

### 2. Add Think Tool

Claude Code does not have custom tool definitions, but you can simulate the think tool by requiring structured output in DELIBERATE/SLOW modes.

Add this to your CLAUDE.md:

```markdown
## Think Tool (DELIBERATE and SLOW modes)

When in DELIBERATE or SLOW mode, output a structured reasoning block BEFORE acting:

<think>
MODE: [deliberate|slow]
GIST: [1-sentence decision]
EVIDENCE_LOG:
  - Pull 1: [tool] → [finding]
DISCONFIRMER: [competing explanation]
PRE_MORTEM:
  - [Reason 1 plan could fail]
  - [Reason 2 plan could fail]
WYSIATI: [what's missing?]
DECISION: [chosen approach]
TERMINAL: [done|ask|escalate]
REFLECTION:
  Was justified: [yes|no|uncertain]
  Lesson: [1 sentence]
</think>
```

### 3. Mode Classification

Add to CLAUDE.md:

```markdown
## Mode Classification

Before every task, classify:
- FAST: Single-file edit, rename, trivial lookup → 0 pulls
- DELIBERATE: One unknown, slight ambiguity → 1 pull max
- SLOW: Architecture, debugging, 3+ approaches → 3 pulls max

Declare MODE and JUSTIFICATION at the start of every task.
```

### 4. Skill Compilation

Use `.claude/memory/` or project-specific memory files to save patterns:

```markdown
## Saved Pattern: [name]

**What:** [Description]
**Why:** [Rationale]
**Where:** [Files]
**Learned:** [Gotchas]

**Usage:** FAST mode can reference this pattern directly.
```

### 5. Project-Specific Setup

For a project-specific reasoning setup:

1. Create `.claude/CLAUDE.md` in project root
2. Add the agent prompt footer
3. Add project-specific base rates (see `patterns/outside-view.md`)
4. Add project-specific WYSIATI checks (e.g., "Check for monorepo cross-package dependencies")

### Example

See `examples/claude-code-setup.md` for a complete project setup.
