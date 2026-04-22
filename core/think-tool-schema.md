## THINK TOOL SCHEMA v2.0

Structured reasoning scratchpad for DELIBERATE and SLOW modes. No free-text chain-of-thought.

### Schema

```yaml
THINK_TOOL:
  version: "2.0"
  mode: [deliberate|slow]
  
  gist: string                    # 1-sentence decision-bearing summary
  
  evidence_log:                   # Must match actual tool calls
    - pull_n:
        tool: string              # Tool name
        target: string            # What was queried
        finding: string           # What was learned
  
  disconfirmer: string            # One competing explanation or falsifier
  
  pre_mortem:                     # SLOW mode mandatory, DELIBERATE optional
    - string                      # Reason 1 this plan could fail
    - string                      # Reason 2
    - string                      # Reason 3
  
  wysiati: string                 # What critical evidence is still missing?
  
  decision: string                # Chosen approach with explicit trade-offs
  confidence: [high|medium|low]   # Self-reported confidence
  
  terminal: [done|ask|escalate]
  
  mode_transition:
    from: [fast|deliberate|slow]
    to: [fast|deliberate|slow|none]
    reason: string
  
  reflection:
    was_justified: [yes|no|uncertain]
    would_fast_have_sufficed: [yes|no|uncertain]
    lesson: string                # 1 sentence for future calibration
```

### Validation Rules

1. **evidence_log length** must equal actual evidence pulls. No phantom pulls.
2. **disconfirmer** is mandatory. If absent → reasoning incomplete → do not claim completion.
3. **pre_mortem** is mandatory in SLOW mode. Optional in DELIBERATE.
4. **wysiati** is mandatory. "Nothing is missing" is a WYSIATI violation → lower confidence.
5. **reflection** is mandatory after every DELIBERATE/SLOW task.
6. **terminal** must be one of `done`, `ask`, `escalate`.

### Example: DELIBERATE Mode

```yaml
THINK_TOOL:
  version: "2.0"
  mode: deliberate
  
  gist: "Add rate limiting to /api/users using existing express-rate-limit pattern"
  
  evidence_log:
    - pull_1:
        tool: read
        target: "src/middleware/rateLimit.ts"
        finding: "Already uses express-rate-limit with Redis store. Pattern: 100 req/15min per IP."
  
  disconfirmer: "Could use a custom middleware instead for more granular control per user ID."
  
  pre_mortem: null  # DELIBERATE mode — optional
  
  wysiati: "Haven't checked if /api/users has special requirements (e.g., lower limit for auth endpoints)."
  
  decision: "Use existing express-rate-limit pattern. Add /api/users to the routes array."
  confidence: high
  
  terminal: done
  
  mode_transition:
    from: fast
    to: fast
    reason: "Pull confirmed existing pattern. No ambiguity."
  
  reflection:
    was_justified: yes
    would_fast_have_sufficed: uncertain
    lesson: "Always check for existing middleware before implementing new rate limiting."
```

### Example: SLOW Mode

```yaml
THINK_TOOL:
  version: "2.0"
  mode: slow
  
  gist: "Choose between JWT and session-based auth for new API"
  
  evidence_log:
    - pull_1:
        tool: read
        target: "docs/security-requirements.md"
        finding: "Requires stateless auth for microservices. No server-side session storage available."
    - pull_2:
        tool: engram_mem_search
        target: "auth decisions"
        finding: "Past decision (2024-03): chose JWT for similar microservice. No issues reported."
    - pull_3:
        tool: webfetch
        target: "OWASP JWT best practices 2025"
        finding: "Short expiry + refresh tokens recommended. RS256 for asymmetric signing."
  
  disconfirmer: "Sessions with Redis would give better revocation control and are simpler to implement."
  
  pre_mortem:
    - "JWT secret rotation could break all active tokens"
    - "Token size could exceed HTTP header limits with large claims"
    - "Logout implementation requires token blacklist (adds complexity)"
  
  wysiati: "Haven't checked exact token size with our claim structure. Haven't verified RS256 library support."
  
  decision: "Use JWT with RS256, 15-min access tokens, 7-day refresh tokens. Implement token blacklist for logout."
  confidence: medium
  
  terminal: done
  
  mode_transition:
    from: fast
    to: fast
    reason: "Decision made, documented, ready for implementation."
  
  reflection:
    was_justified: yes
    would_fast_have_sufficed: no
    lesson: "Auth decisions always need SLOW mode. Past patterns help but security requirements vary."
```
