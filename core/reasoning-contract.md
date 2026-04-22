## REASONING CONTRACT (MANDATORY)

Every agent using this framework must follow the 3-tier graduated reasoning contract.

**Three tiers:**
- **FAST** (System 1): Pattern-matching, single-pass, zero research
- **DELIBERATE** (System 1.5): Bounded check — gist + 1 evidence pull + go/no-go
- **SLOW** (System 2): Full 7-phase pipeline with hard stops

---

## 1. Mode Declaration

At the start of every task, declare:
```
MODE: [fast|deliberate|slow]
JUSTIFICATION: [1 sentence — why this mode?]
```

## 2. Evidence Budget

| Mode | Budget | On Breach |
|------|--------|-----------|
| FAST | 0 additional pulls | Escalate to DELIBERATE |
| DELIBERATE | 1 pull max | Escalate to SLOW |
| SLOW | Anchor + 3 pulls | Mandatory terminal state ESCALATE |

**Definition:** One pull = one tool call returning new info (read, grep, search, fetch). Re-reading a previously read file does NOT count.

## 3. Mode Transitions

| From | To | Trigger |
|------|-----|---------|
| fast | deliberate | One unknown, slight ambiguity |
| fast | slow | 3+ approaches, high-stakes, unfamiliar |
| deliberate | slow | Fatal flaw after pull |
| deliberate | fast | Pull confirms gist |
| slow | fast | Terminal state DONE with verification |
| any | escalate | Budget exhausted or fatal flaw holds |

All transitions require 1-sentence justification.

## 4. Think Tool

Mandatory for DELIBERATE and SLOW modes. See `core/think-tool-schema.md`.

## 5. Terminal States

All modes end in one of:
- `done` — task complete, verified
- `ask` — need user input
- `escalate` — mode budget exhausted or fatal flaw found

## 6. Skill Compilation

After successful DELIBERATE/SLOW: save pattern to memory. Next time, FAST mode finds it via query and executes directly.

## 7. Meta-Cognition

After every DELIBERATE/SLOW: evaluate whether mode was justified. Save for calibration.
