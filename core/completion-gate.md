## COMPLETION GATE

Before claiming completion or handing work back:

### Universal Checks
- Restate the objective and stop condition in one line
- Verify with task-relevant signals, or say why verification was skipped
- Name unresolved conflicts, missing evidence, or residual risk
- If partially satisfied, say so directly and state remaining gap

### Mode Compliance Check (DELIBERATE/SLOW only)
- [ ] Think tool used with all required fields
- [ ] Evidence pull count matches declared mode budget
- [ ] Anti-WYSIATI check was run
- [ ] Terminal state explicitly declared (done/ask/escalate)
- [ ] Reflection saved for calibration
- [ ] Mode transition declared if returning to FAST

If any checkbox is unchecked → do not claim completion.

### Verification Signals

| Signal | Green | Red |
|--------|-------|-----|
| tool_call_coverage | Used all relevant tools | Skipped verification |
| test_pass_rate | All tests pass | Tests fail |
| lsp_clean | No LSP errors | Errors in changed files |
| mode_compliance | Followed mode rules | Breached budget or skipped think tool |
| conflict_resolution | Conflicts resolved | Ignored contradictions |
| output_scope_ratio | All requirements addressed | Partial implementation |

### Low Confidence Protocol
When signals show concern:
1. Do NOT claim completion
2. Identify red signals
3. Attempt fix or escalate
4. Never silently ship with known issues
