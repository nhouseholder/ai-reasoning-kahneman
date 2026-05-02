## COMPLETION GATE v3.0

Before claiming completion or handing work back:

### Universal Checks (all modes)
- Restate the objective and stop condition in one line
- Verify with task-relevant signals, or say why verification was skipped
- Name unresolved conflicts, missing evidence, or residual risk
- If partially satisfied, say so directly and state remaining gap

### Completion Gate Lite (SLOW mode only)

Answer 4 questions:

1. **FALSIFIER:** What single fact would kill this plan?
2. **ALTERNATIVE:** What's the simplest approach I didn't consider?
3. **FAILURE:** How could this fail in production?
4. **SPEED:** Could FAST mode have solved this? Why not?

**Rules:**
- Cannot answer #1 → lower confidence or escalate
- #4 answer is "yes" → save calibration: "would_fast_have_sufficed: yes"
- No JSON. No schema. Just answer the questions.

### Mode Compliance Check (DELIBERATE/SLOW only)
- [ ] Evidence pulls within budget
- [ ] Anti-WYSIATI check was run
- [ ] Terminal state explicitly declared (done/ask/escalate)
- [ ] Calibration data auto-saved

If any checkbox is unchecked → do not claim completion.

### Verification Signals

| Signal | Green | Red |
|--------|-------|-----|
| tool_call_coverage | Used all relevant tools | Skipped verification |
| test_pass_rate | All tests pass | Tests fail |
| lsp_clean | No LSP errors | Errors in changed files |
| mode_compliance | Followed mode rules | Breached budget |
| output_scope_ratio | All requirements addressed | Partial implementation |

### Concrete Verification Criteria

Replace subjective checks with these concrete questions:
- Tests pass?
- No new lint errors?
- No hardcoded values left in?
- No console.log / debug statements?
- Diff is minimal (only necessary changes)?

### Low Confidence Protocol
When signals show concern:
1. Do NOT claim completion
2. Identify red signals
3. Attempt fix or escalate
4. Never silently ship with known issues
