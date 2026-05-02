## REASONING CONTRACT v3.2

Every agent using this framework must follow the inverted 3-tier graduated reasoning contract.

**Default: FAST. Native reasoning OFF. Escalate with justification.**

### Inline Mode Classification

Classify the task **before any file reads**. No external dependencies required.

| Mode | Trigger | Planning |
|------|---------|----------|
| FAST | Single-file edit, typo fix, simple read/grep | No plan |
| DELIBERATE | Multi-file but familiar, straightforward refactor | Optional plan |
| SLOW | Architecture changes, unfamiliar codebase, deep debugging | Plan required |

**When in doubt, default to DELIBERATE.**

---

## 1. Mode Declaration + Confidence Scale

At the start of every task, declare:
```
MODE: [fast|deliberate|slow]
NATIVE_REASONING: [off|on]
JUSTIFICATION: [1 sentence — why this mode?]
```

**FAST is the default.** You must justify any escalation.

**Confidence scale:**
- HIGH = 90%+ confident
- MEDIUM = 70–89% confident
- LOW = <70% confident

Express uncertainty explicitly when below HIGH. Yolo execution stops if confidence drops below MEDIUM.

## 2. Evidence Budget

| Mode | Pulls | Reasoning Tokens | On Breach |
|------|-------|-----------------|-----------|
| FAST | 0 | 200 max | Forced terminal state `escalate` |
| DELIBERATE | 1 | 1,500 max | Escalate to SLOW |
| SLOW | 3 | Unlimited | Mandatory terminal state `escalate` |

**One pull =** tool call returning new info, OR reasoning block >500 tokens, OR re-reading a previously read file.

## 3. Speed Mode

For trivial tasks, explicitly suppress native reasoning:
```
MODE: fast
NATIVE_REASONING: off
```

Execute in one shot. No tool calls unless file read is absolutely necessary.

## 4. Mode Transitions

| From | To | Trigger |
|------|-----|---------|
| fast | deliberate | Ambiguity detected during execution |
| fast | slow | User prompt OR high-stakes signal |
| deliberate | slow | Fatal flaw after pull |
| deliberate | fast | Pull confirms gist, no new ambiguity |
| slow | fast | Terminal state `done` with verification |
| any | escalate | Budget exhausted or fatal flaw holds |

All transitions require 1-sentence justification.

## 5. Completion Gate Lite (SLOW only)

Before claiming done, answer 4 questions:
1. FALSIFIER: What single fact would kill this plan?
2. ALTERNATIVE: What's the simplest approach I didn't consider?
3. FAILURE: How could this fail in production?
4. SPEED: Could FAST mode have solved this? Why not?

If you cannot answer #1 → lower confidence or escalate.

## 6. Skill Compilation

After successful DELIBERATE/SLOW: save pattern to memory. Next time, FAST mode queries memory → finds saved pattern → executes directly.

## 7. Meta-Cognition

After every DELIBERATE/SLOW: save calibration data to memory. No manual reflection required.
