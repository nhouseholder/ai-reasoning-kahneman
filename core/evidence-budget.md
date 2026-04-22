## EVIDENCE BUDGET

### The Rule
Every mode has a hard limit on evidence pulls. Breach = mandatory escalation.

### What Counts as a Pull
One evidence pull = one tool call that returns **new** information:

**Counts:**
- `read` (new file not yet read this task)
- `grep` / `glob` / `ast_grep`
- `brain-router_brain_query`
- `engram_mem_search`
- `mempalace_mempalace_search`
- `webfetch`

**Does NOT count:**
- Re-reading a previously read file
- Running tests (verification, not evidence gathering)
- Writing files (action, not research)
- Checking git status (context, not new info)

### Budget Table

| Mode | Budget | Starting Anchor | Pulls Allowed |
|------|--------|-----------------|---------------|
| FAST | 0 | N/A | 0 |
| DELIBERATE | 1 | Current context/memory | 1 |
| SLOW | 3 | Current context/memory | 3 |

**Anchor:** Your initial context, memory, or gist does NOT count toward the limit.

### Enforcement
- Agent maintains mental counter
- Think tool `evidence_log` tracks actual pulls
- Breach triggers immediate terminal state `escalate`
- No exceptions for "just one more check"

### Examples

**FAST (0 pulls):**
> User: "Rename UserProfile to UserProfileCard"
> Agent: Reads file → renames → verifies. Zero research pulls needed.

**DELIBERATE (1 pull):**
> User: "Add auth to this endpoint"
> Agent: MODE: DELIBERATE. Pull 1: Read existing auth middleware → confirms pattern → implements.

**SLOW (3 pulls):**
> User: "Should we use JWT or sessions?"
> Agent: MODE: SLOW.
> Pull 1: Read project auth requirements
> Pull 2: Search memory for past auth decisions
> Pull 3: Check security constraints
> Then: Pre-mortem → Decision → Terminal state.
