## EVIDENCE BUDGET v3.0

### The Rule
Every mode has a hard limit on evidence pulls AND reasoning tokens. Breach = mandatory escalation.

### What Counts as a Pull
One evidence pull = one of:
- `read` (new file not yet read this task)
- `grep` / `glob` / `ast_grep`
- `memory_search` / `mem_search`
- `webfetch`
- Reasoning block exceeding 500 tokens (internal monologue)
- Re-reading a previously read file (cognitive loop = 0.5 pull)

**Does NOT count:**
- Re-reading within 30 seconds of first read (working memory)
- Running tests (verification)
- Writing files (action)
- Checking git status (context)

### Budget Table

| Mode | Pulls | Reasoning Tokens | Starting Anchor |
|------|-------|-----------------|----------------|
| FAST | 0 | 200 max | N/A |
| DELIBERATE | 1 | 1,500 max | Context + memory |
| SLOW | 3 | Unlimited | Context + memory |

### Enforcement
- **Honor system** with mandatory self-reporting
- Budget breach triggers escalation to next mode
- No exceptions for "just one more check"

### Examples

**FAST (0 pulls, 200 tokens):**
> User: "Rename UserProfile to UserProfileCard"
> Agent: Reads file → renames → verifies. Zero research. Zero reasoning chain.

**DELIBERATE (1 pull, 1,500 tokens):**
> User: "Add auth to this endpoint"
> Agent: MODE: DELIBERATE. Pull 1: Read existing auth middleware → confirms pattern → implements.

**SLOW (3 pulls, unlimited tokens):**
> User: "Should we use JWT or sessions?"
> Agent: MODE: SLOW.
> Pull 1: Read project auth requirements
> Pull 2: memory_search("past auth decisions")
> Pull 3: webfetch("OWASP JWT best practices 2025")
> Then: Completion Gate Lite → Decision → Terminal state.
