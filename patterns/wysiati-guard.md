## WYSIATI GUARD

**What You See Is All There Is** — System 1 builds coherent narratives from available evidence without accounting for missing information.

### Purpose
Forces explicit consideration of unknown unknowns before high-confidence completion.

### When to Use
- Mandatory before completing DELIBERATE or SLOW tasks
- Optional for FAST tasks that feel "too easy"
- Skip only for truly trivial tasks (single-word edit, typo fix)

### The Check
Answer these 4 questions:

1. **What critical evidence is still missing?**
   - What have I not examined that could change the conclusion?
   - What files, dependencies, or constraints are invisible to me?

2. **What competing explanation still fits?**
   - Is there another story that explains the same evidence?
   - Have I considered the null hypothesis?

3. **What memory, assumption, or prior pattern could be stale?**
   - When was the last time this codebase was in this state?
   - Am I relying on memory from a different project or version?

4. **What concrete file, test, or external source would falsify my story?**
   - If I am wrong, what evidence would prove it?
   - Can I name a specific test that would fail?

### Decision Rule
- If you can answer all 4 with specifics → proceed with confidence
- If you cannot answer #4 → lower confidence or escalate
- If you cannot answer #1 → you are likely victim to WYSIATI → escalate

### Example
```
WYSIATI CHECK:

1. Missing evidence:
   - Haven't checked if the auth middleware is applied globally or per-route
   - Haven't verified the Redis connection pool size

2. Competing explanation:
   - The 500 errors could be from database connection limits, not auth

3. Stale assumption:
   - Assumed JWT secret is in env var; might be in Vault now (changed 2 months ago)

4. Falsifier:
   - Check logs for "JWT verification failed" vs "Database connection timeout"
   - Run load test with 100 concurrent requests

VERDICT: Cannot answer #4 without checking logs. Lower confidence to MEDIUM.
ACTION: Check logs before claiming fix is correct.
```
