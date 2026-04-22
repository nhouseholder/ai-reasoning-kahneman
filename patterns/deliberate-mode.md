## DELIBERATE MODE

**Bounded check. Gist + 1 evidence pull + go/no-go.**

### When to Use
- Task requires verifying one assumption before acting
- Slight ambiguity in scope (2 viable approaches, not 3+)
- Need to check one file, one memory entry, or one doc before proceeding
- User asks for a quick check or sanity review
- Something "feels off" in FAST mode

### Processing Flow
```
State gist → Run 1 evidence pull → Verify pull changes/confirms gist → Act or escalate
```

### Evidence Budget
**1 pull maximum.**

### Rules
1. State your gist clearly
2. Run exactly 1 evidence pull
3. If pull confirms gist → proceed in FAST mode from there
4. If pull reveals new ambiguity or contradiction → escalate to SLOW
5. Use think tool (see `core/think-tool-schema.md`)

### Pre-Mortem
Optional in DELIBERATE mode. Use only if the pull reveals a non-trivial risk.

### Example
```
User: "Add auth to this endpoint"

MODE: deliberate
JUSTIFICATION: "Need to verify existing auth pattern before implementing"

Pull 1: Read src/middleware/auth.ts
Finding: "Uses JWT with passport-jwt. Pattern: verify token, attach user to req."

THINK_TOOL:
  mode: deliberate
  gist: "Use existing JWT middleware pattern for new endpoint"
  evidence_log:
    - pull_1:
        tool: read
        target: "src/middleware/auth.ts"
        finding: "JWT with passport-jwt pattern confirmed"
  disconfirmer: "Could use RBAC middleware instead for role-based access"
  wysiati: "Haven't checked if this endpoint needs special roles"
  decision: "Use existing JWT pattern. Add endpoint to auth whitelist."
  terminal: done
  reflection:
    was_justified: yes
    would_fast_have_sufficed: no
    lesson: "Auth patterns vary — always verify existing middleware first"
```
