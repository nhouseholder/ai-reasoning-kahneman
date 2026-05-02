# Kahneman AI Reasoning Framework — Specification v3.0

## 1. Design Philosophy

**Default FAST. Escalate to DELIBERATE or SLOW only with explicit justification.**

Modern reasoning models overthink by default. They burn thousands of tokens and dozens of tool calls on trivial tasks. This framework treats reasoning as a **resource allocation problem** with a hard ceiling, not a personality trait.

**The inversion:** v2.0 forced agents to think MORE. v3.0 forces models to think LESS — with strategic depth when it matters.

## 2. Three-Tier Model

### 2.1 FAST Mode (System 1)
- **Speed**: Instant, automatic
- **Effort**: Zero cognitive load
- **Evidence budget**: 0 pulls, max 200 reasoning tokens
- **Tasks**: Pattern matching, single-file edits, renames, formatting, trivial lookups, known patterns from memory
- **Failure mode**: WYSIATI — jumps to conclusions

### 2.2 DELIBERATE Mode (System 1.5)
- **Speed**: Quick, bounded
- **Effort**: Low cognitive load
- **Evidence budget**: 1 pull max, max 1,500 reasoning tokens
- **Tasks**: Verify one assumption, slight ambiguity, quick sanity check, confirm known pattern still applies
- **Failure mode**: Under-thinking — treats ambiguous task as obvious

### 2.3 SLOW Mode (System 2)
- **Speed**: Slow, deliberate
- **Effort**: High cognitive load
- **Evidence budget**: 3 pulls, reasoning unlimited
- **Tasks**: Architecture, debugging, planning, security, refactoring, novel problems
- **Failure mode**: Analysis paralysis

## 3. Mode State Machine

### 3.1 States
- `fast` (default)
- `deliberate`
- `slow`
- `escalated` (terminal)

### 3.2 Entry Points

**FAST** is the default. Every task starts here unless user explicitly requests deep thinking.

**DELIBERATE triggers:**
- One unknown in an otherwise clear task
- Slight ambiguity (2 viable approaches)
- Need to verify one assumption before proceeding
- User asks for "quick check"

**SLOW triggers:**
- User explicitly says "think carefully" or "analyze deeply"
- 3+ viable approaches
- High-stakes architectural impact
- Unfamiliar domain
- Security or irreversible decision
- Prior DELIBERATE pull revealed fatal flaw

### 3.3 Transition Rules
| From | To | Trigger | Justification |
|------|-----|---------|--------------|
| fast | deliberate | Ambiguity detected during execution | 1 sentence |
| fast | slow | User prompt OR high-stakes signal | 1 sentence |
| deliberate | slow | Fatal flaw after pull | 1 sentence |
| deliberate | fast | Pull confirms gist, no new ambiguity | 1 sentence |
| slow | fast | Terminal state `done` with verification | 1 sentence |
| any | escalate | Budget exhausted or fatal flaw holds | N/A (forced) |

**Critical rule:** Never skip DELIBERATE when escalating from FAST. It's the buffer.

## 4. Evidence Budget

### 4.1 Definition
One evidence pull = one of:
- Tool call returning **new** information (read new file, grep, search, fetch)
- Reasoning block exceeding 500 tokens (internal monologue)
- Re-reading a previously read file (cognitive loop — counts as 0.5)

**Does NOT count:**
- Running tests (verification, not evidence)
- Writing files (action, not research)
- Checking git status (context, not new info)
- Re-reading a file within 30 seconds of first read (working memory)

### 4.2 Budget Table
| Mode | Pulls | Reasoning Tokens | Starting Anchor |
|------|-------|-----------------|----------------|
| FAST | 0 | 200 max | N/A |
| DELIBERATE | 1 | 1,500 max | Context + memory |
| SLOW | 3 | Unlimited | Context + memory |

### 4.3 Enforcement
- **Honor system** with mandatory self-reporting
- Budget breach triggers escalation to next mode
- No exceptions for "just one more check"

## 5. Completion Gate Lite

Before claiming done on SLOW tasks:

1. **FALSIFIER**: What single fact would kill this plan?
2. **ALTERNATIVE**: What's the simplest approach I didn't consider?
3. **FAILURE**: How could this fail in production?
4. **SPEED**: Could FAST mode have solved this? Why not?

**Rules:**
- If you cannot answer #1 → lower confidence or escalate
- If #4 answer is "yes" → save calibration: "would_fast_have_sufficed: yes"
- No JSON schema. No structured scratchpad. Just answer the 4 questions.

## 6. Anti-WYSIATI Check

Before claiming completion on DELIBERATE/SLOW:

1. What critical evidence is still missing?
2. What competing explanation still fits the current evidence?
3. What memory, assumption, or prior pattern could be stale?
4. What concrete file, test, or external source would falsify the current story?

If you cannot answer #4 → lower confidence or escalate.

## 7. Pre-Mortem (SLOW Mode Only)

Imagine the plan has already failed. List 2-3 plausible reasons why. Address them or escalate.

**Exit criteria:** Risks surfaced and addressed.

## 8. Skill Compilation

After successfully solving a novel problem in DELIBERATE or SLOW mode:

```
SAVE to memory:
  title: "Pattern: [name]"
  content: "## Compiled Truth\n**What**: ...\n**Why**: ...\n**Where**: ...\n**Learned**: ..."
  type: "pattern"
  topic_key: "reasoning/pattern-[name]"
```

Next time, FAST mode queries memory → finds saved pattern → executes directly.

## 9. Meta-Cognitive Feedback Loop

After every DELIBERATE or SLOW task, auto-save calibration data:

```json
{
  "task_type": "bugfix",
  "mode_declared": "slow",
  "pulls_actual": 4,
  "tokens_actual": 3200,
  "outcome": "success",
  "would_fast_have_sufficed": "no"
}
```

Saved to memory with topic_key: `reasoning/calibration`.

## 10. Integration with Agent Memory

### 10.1 Session Start
```
1. Start session tracking (project name)
2. Load recent context from memory
3. Check for codebase diagram / past work on this topic
4. NOW reply to user
```

### 10.2 During Work
```
- Trivial task → stay FAST
- Ambiguity detected → declare DELIBERATE
- Architecture/security → declare SLOW
```

### 10.3 Before Done (SLOW only)
```
- Run Completion Gate Lite (4 questions)
- Save calibration data
- Save pattern if novel solution
```

## 11. Version History

- **v3.0** (2026-04-26): Inverted default (FAST first), token-aware budget, Completion Gate Lite, soft enforcement
- **v2.0** (2025-04-21): 3-tier model, think tool schema, meta-cognitive feedback
- **v1.0** (2025-04-20): Binary fast/slow, basic Kahneman framework
