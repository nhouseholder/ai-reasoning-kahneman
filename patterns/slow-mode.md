## SLOW MODE

**Full analysis. 7-phase pipeline with hard stops.**

### When to Use
- Ambiguous scope or 3+ viable approaches
- High-stakes architectural or product impact
- Unfamiliar domain or missing prior pattern in memory
- Unexpected verification failure, user correction, or contradictory evidence
- Cross-file/cross-system reasoning where local fixes are unsafe
- Prior DELIBERATE pull revealed fatal flaw or new ambiguity

### Processing Flow
```
Scope → Evidence → Disconfirm → Pre-Mortem → Decision → Act → Verify
```

### Evidence Budget
**Anchor + 3 additional pulls maximum.**

The starting anchor (your initial context, memory, or gist) does NOT count toward the 3-pull limit.

### Phase 1: Scope
- State the bottom-line gist
- Lock objective and deliverable
- Name stop condition
- **Exit criteria:** Decision question is stable

### Phase 2: Evidence
- Gather only files/docs/memory that can materially change the decision
- **Exit criteria:** One more read would not change the call

### Phase 3: Disconfirm
- Name one competing explanation, stale-memory risk, or falsifier
- Run explicit fatal-flaw test: "What single fact would kill this plan?"
- **Exit criteria:** One serious challenge completed

### Phase 4: Pre-Mortem (from Kahneman)
- Imagine the plan has already failed
- List 2-3 plausible reasons why
- Address them or escalate
- **Exit criteria:** Risks surfaced and addressed

### Phase 5: Decision
- Choose approach with explicit trade-offs
- **Exit criteria:** Alternatives closed on current evidence

### Phase 6: Act
- Execute, delegate, or recommend
- **Exit criteria:** Concrete next move taken

### Phase 7: Verify
- Objective checks (tests, LSP, constraints)
- Hand off gist + minimum supporting detail
- **Exit criteria:** Terminal state declared

### Hard Rules
1. No backward movement unless materially new evidence appears
2. One self-correction pass allowed per phase
3. If corrected approach still fails → escalate
4. After 3 pulls, MUST choose terminal state
5. Use think tool (mandatory)

### Example
See `core/think-tool-schema.md` for full SLOW mode think tool example.
