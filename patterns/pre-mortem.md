## PRE-MORTEM PROTOCOL

From Kahneman: Before committing to a plan, imagine it has already failed and generate reasons why.

### Purpose
Overrides optimism bias and forces System 2 to surface risks System 1 suppresses.

### When to Use
- Mandatory in SLOW mode
- Optional in DELIBERATE mode (if pull reveals non-trivial risk)
- Skip in FAST mode

### Process
1. State your plan in one sentence
2. Imagine it is 6 months later and the plan has failed
3. List 2-3 plausible reasons why it failed
4. For each reason: Is it plausible enough to address now?
5. If yes → modify plan or add safeguards
6. If no → document the risk and proceed

### Template
```
PLAN: [One sentence]

IMAGINED FAILURE SCENARIO:
It is 6 months later. This plan has failed. Here is why:

1. [Reason 1 — e.g., "The JWT secret rotation broke all active tokens because we didn't implement graceful rotation"]
   Plausible? [yes/no]
   Action: [Modify plan / Add safeguard / Accept risk]

2. [Reason 2 — e.g., "Token size exceeded HTTP header limits with our claim structure"]
   Plausible? [yes/no]
   Action: [Modify plan / Add safeguard / Accept risk]

3. [Reason 3 — e.g., "The Redis dependency introduced a single point of failure"]
   Plausible? [yes/no]
   Action: [Modify plan / Add safeguard / Accept risk]

FINAL PLAN: [Modified plan with safeguards]
```

### Example
```
PLAN: "Use JWT with RS256 for auth, 15-min access tokens, 7-day refresh tokens"

IMAGINED FAILURE SCENARIO:
1. "Secret rotation broke all tokens"
   Plausible? Yes
   Action: Implement key ID (kid) header to support graceful rotation

2. "Token size exceeded header limits"
   Plausible? No — claims are minimal
   Action: Accept risk, monitor token size

3. "Redis dependency became single point of failure"
   Plausible? Yes
   Action: Add fallback to in-memory blacklist with TTL

FINAL PLAN: "JWT with RS256 + kid header + Redis blacklist with in-memory fallback"
```
