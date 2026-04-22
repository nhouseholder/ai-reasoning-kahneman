## MODE STATE MACHINE

### States
- `fast`
- `deliberate`
- `slow`
- `escalated` (terminal)

### Entry Points

**FAST** (default): Agent starts here unless signals fire.

**DELIBERATE** (triggered by):
- Task requires verifying one assumption
- Slight ambiguity (2 viable approaches)
- Need to check one file/memory/doc before proceeding
- User asks for quick check

**SLOW** (triggered by):
- 3+ viable approaches
- High-stakes architectural impact
- Unfamiliar domain
- Verification failure or contradiction
- Cross-file reasoning
- Prior DELIBERATE pull revealed fatal flaw

### Exit Protocols

**From FAST:**
- Complete task → `done`
- Need user input → `ask`
- Hit unknown/ambiguity → escalate to `deliberate` or `slow`

**From DELIBERATE:**
- Pull confirms gist, task complete → `done`
- Pull reveals no issues, can proceed in FAST → transition to `fast`, then `done`
- Pull reveals ambiguity/fatal flaw → escalate to `slow`
- Need user input → `ask`

**From SLOW:**
- All phases complete, verified → `done`
- Budget exhausted → `escalate`
- Need user input → `ask`
- Fatal flaw holds after self-correction → `escalate`

### Transition Rules
1. Always declare mode at start
2. Always justify transitions in 1 sentence
3. Never skip DELIBERATE — it's the buffer between FAST and SLOW
4. After `done`, declare `MODE_TRANSITION: [current] → fast` if returning to idle
