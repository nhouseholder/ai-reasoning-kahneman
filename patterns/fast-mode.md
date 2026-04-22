## FAST MODE

**The default. Pattern-matching, single-pass, zero research.**

### When to Use
- Single-file edits with clear scope
- Renames, formatting, boilerplate
- Running known commands
- CRUD operations
- Cosmetic changes
- Trivial lookups (`grep`, `ls`, `git status`)
- Executing an existing plan
- Tasks where memory returns a matching pattern

### Processing Flow
```
Pattern match → Read what you need → Act → Quick verify → Done
```

### Evidence Budget
**0 additional pulls.**

Your starting context (user request, memory, current file) is your anchor. You may act on it directly. If you need to pull new evidence, escalate to DELIBERATE.

### Rules
1. Start with a working gist
2. Read only what you need to act safely
3. Prefer established repo patterns
4. Do not trigger multi-step research
5. If gist depends on missing evidence → escalate to DELIBERATE

### Failure Mode: WYSIATI
System 1 jumps to conclusions based on available data, ignoring what's missing. It solves the *easier* version of the problem (attribute substitution).

**Guard:** If anything feels off → escalate to DELIBERATE immediately. Do not push through uncertainty in FAST mode.

### Example
```
User: "Rename UserProfile to UserProfileCard"

MODE: fast
JUSTIFICATION: "Single-file rename with clear scope"

Action: Read UserProfile.tsx → Rename component and file → Update imports → Verify with LSP

THINK_TOOL: null  # FAST mode — think tool not required

Terminal: done
```
