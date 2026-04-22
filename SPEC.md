# Kahneman AI Reasoning Framework — Full Specification

## 1. Design Philosophy

**Default to FAST. Escalate to DELIBERATE or SLOW only when evidence warrants.**

This framework treats reasoning as a **resource allocation problem**, not a personality trait. Every task gets exactly the compute it needs — no more, no less. Agents must justify their mode choice and self-evaluate afterward.

## 2. Three-Tier Model

### 2.1 FAST Mode (System 1)
- **Speed**: Instant, automatic
- **Effort**: Zero cognitive load
- **Control**: Unconscious, always-on
- **Evidence budget**: 0 additional pulls
- **Tasks**: Pattern matching, single-file edits, renames, formatting, trivial lookups
- **Failure mode**: WYSIATI — jumps to conclusions, ignores missing context

### 2.2 DELIBERATE Mode (System 1.5)
- **Speed**: Quick, bounded
- **Effort**: Low cognitive load
- **Control**: Conscious, brief
- **Evidence budget**: 1 pull maximum
- **Tasks**: Verify one assumption, slight ambiguity, quick sanity check
- **Failure mode**: Under-thinking — treats ambiguous task as obvious

### 2.3 SLOW Mode (System 2)
- **Speed**: Slow, deliberate
- **Effort**: High cognitive load
- **Control**: Conscious, invoked selectively
- **Evidence budget**: Anchor + 3 additional pulls
- **Tasks**: Architecture, debugging, planning, security, refactoring
- **Failure mode**: Analysis paralysis — token exhaustion, overthinking

## 3. Mode State Machine

### 3.1 States
- `fast`
- `deliberate`
- `slow`
- `escalated` (terminal)

### 3.2 Transitions
| From | To | Trigger | Justification required |
|------|-----|---------|----------------------|
| fast | deliberate | One unknown, slight ambiguity | Yes, 1 sentence |
| fast | slow | 3+ approaches, high-stakes, unfamiliar | Yes, 1 sentence |
| deliberate | slow | Fatal flaw, new ambiguity after pull | Yes, 1 sentence |
| deliberate | fast | Pull confirms gist, no new ambiguity | Yes, 1 sentence |
| slow | fast | Terminal state `done` with verification | Yes, 1 sentence |
| any | escalated | Budget exhausted or fatal flaw holds | N/A (forced) |

### 3.3 Terminal States
All modes end in one of:
- `done` — task complete, verified
- `ask` — need user input
- `escalate` — mode budget exhausted or fatal flaw found after self-correction

## 4. Evidence Budget

### 4.1 Definition
One evidence pull = one tool call that returns new information:
- `read` (new file)
- `grep` / `glob` / `ast_grep`
- `brain-router_brain_query`
- `engram_mem_search`
- `mempalace_mempalace_search`
- `webfetch`

Re-reading a previously read file does NOT count as a new pull.

### 4.2 Budget by Mode
| Mode | Budget | What happens on breach |
|------|--------|----------------------|
| FAST | 0 | Escalate to DELIBERATE |
| DELIBERATE | 1 | Escalate to SLOW |
| SLOW | 3 | Mandatory terminal state `escalate` |

### 4.3 Budget Tracking
Agents must maintain a mental counter. The think tool schema includes `evidence_log` to track actual pulls against budget.

## 5. Think Tool Schema

Mandatory for DELIBERATE and SLOW modes. No free-text chain-of-thought allowed.

```yaml
THINK_TOOL:
  version: "2.0"
  mode: [deliberate|slow]
  
  # Core reasoning
  gist: string           # 1-sentence decision-bearing summary
  evidence_log:          # Must match actual tool calls
    - pull_n: 
        tool: string     # Tool name
        target: string   # What was queried
        finding: string  # What was learned
  
  # Critical thinking
  disconfirmer: string   # One competing explanation or falsifier
  pre_mortem:            # SLOW mode only
    - string             # Reason 1 this plan could fail
    - string             # Reason 2
    - string             # Reason 3
  wysiati: string        # What critical evidence is still missing?
  
  # Decision
  decision: string       # Chosen approach with explicit trade-offs
  confidence: [high|medium|low]  # Self-reported confidence
  
  # Terminal state
  terminal: [done|ask|escalate]
  
  # Mode management
  mode_transition: 
    from: [fast|deliberate|slow]
    to: [fast|deliberate|slow|none]
    reason: string
  
  # Meta-cognition
  reflection:
    was_justified: [yes|no|uncertain]
    would_fast_have_sufficed: [yes|no|uncertain]
    lesson: string        # 1 sentence for future calibration
```

### 5.1 Validation Rules
- `evidence_log` length must match actual evidence pulls
- `disconfirmer` is mandatory. If absent, reasoning is incomplete.
- `pre_mortem` is mandatory in SLOW mode, optional in DELIBERATE
- `wysiati` is mandatory. "Nothing is missing" is a WYSIATI violation.
- `reflection` is mandatory after every DELIBERATE/SLOW task

## 6. Slow Mode Phases

SLOW mode follows a 7-phase pipeline:

### Phase 1: Scope
- State bottom-line gist
- Lock objective and deliverable
- Name stop condition
- **Exit criteria**: Decision question is stable

### Phase 2: Evidence
- Gather only files/docs/memory that can materially change the decision
- **Exit criteria**: One more read would not change the call

### Phase 3: Disconfirm
- Name one competing explanation, stale-memory risk, or falsifier
- Run explicit fatal-flaw test: "What single fact would kill this plan?"
- **Exit criteria**: One serious challenge completed

### Phase 4: Pre-Mortem (from Kahneman)
- Imagine the plan has already failed
- List 2-3 plausible reasons why
- Address them or escalate
- **Exit criteria**: Risks surfaced and addressed

### Phase 5: Decision
- Choose approach with explicit trade-offs
- **Exit criteria**: Alternatives closed on current evidence

### Phase 6: Act
- Execute, delegate, or recommend
- **Exit criteria**: Concrete next move taken

### Phase 7: Verify
- Objective checks (tests, LSP, constraints)
- Hand off gist + minimum supporting detail
- **Exit criteria**: Terminal state declared

### Phase Rules
- No backward movement unless materially new evidence appears
- One self-correction pass allowed per phase
- If corrected approach still fails, escalate

## 7. Anti-WYSIATI Check

Before high-confidence completion on DELIBERATE/SLOW tasks:

1. What critical evidence is still missing?
2. What competing explanation still fits the current evidence?
3. What memory, assumption, or prior pattern could be stale?
4. What concrete file, test, or external source would falsify the current story?

If you cannot answer #4, lower confidence or escalate.

## 8. Anti-Loop Guard

- If output is materially identical to previous pass → STOP
- Unknowns become a short list, not another research loop
- One self-correction cycle max before escalation
- If unchanged evidence would reopen Scope or Decision → choose terminal state

## 9. Skill Compilation

After successfully solving a novel problem in DELIBERATE or SLOW mode:

```
SAVE:
  topic_key: "reasoning/pattern-[name]"
  type: "pattern"
  content:
    what: [What was done]
    why: [Why it worked]
    where: [Files affected]
    learned: [Gotchas, edge cases]
```

Next time, FAST mode finds it via memory query and executes directly. Successful slow patterns graduate to fast skills.

## 10. Meta-Cognitive Feedback Loop

After every DELIBERATE or SLOW task:

```
MODE_CALIBRATION:
  task_type: [brief description]
  mode_assigned: [deliberate|slow]
  evidence_pulls_actual: [N]
  outcome: [success|partial|failure]
  was_justified: [yes|no|uncertain]
  would_fast_have_sufficed: [yes|no|uncertain]
  timestamp: [ISO 8601]
```

Save to persistent memory with `topic_key: "reasoning/calibration"`. Over time, this enables data-driven mode assignment.

## 11. Model-Aware Damping

Adjust mode expectations based on model capabilities:

| Model tendency | FAST | DELIBERATE | SLOW |
|---------------|------|-----------|------|
| Fast-execution (Haiku, small local) | Standard | Add 1 extra pull | Escalate earlier |
| Balanced (Sonnet, GPT-4o) | Standard | Standard | Standard |
| Reasoning-heavy (Opus, o1, o3) | Standard | Compress by 30% | Tighter bounds |
| Long-context (Gemini 1.5 Pro) | Standard | Standard | Allow broader retrieval |

Agents should identify their model and adjust accordingly.

## 12. Outside View & Base Rates

For estimation, forecasting, or prediction tasks:

1. **Start with outside view**: What is the base rate for this class of task? Ignore specifics.
2. **Adjust for inside view**: Only after anchoring on base rate, adjust for specifics.
3. **Document both**: Save base rate + adjustment rationale.

Example:
> "How long will this refactor take?"
> Base rate: "Similar refactors took 2-4 hours"
> Adjustment: "This touches 3 more files → +1 hour"
> Final estimate: 3-5 hours

## 13. Completion Gate

Do not claim completion unless:

| Signal | Check |
|--------|-------|
| tool_call_coverage | Right tools used? |
| test_pass_rate | Tests pass? |
| lsp_clean | No LSP errors? |
| mode_compliance | Followed declared mode's rules? |
| conflict_resolution | Conflicts resolved? |
| output_scope_ratio | Everything addressed? |

### Mode Compliance Checklist (DELIBERATE/SLOW only)
- [ ] Think tool used with all required fields
- [ ] Evidence pulls within budget
- [ ] Anti-WYSIATI check run
- [ ] Terminal state declared
- [ ] Reflection saved
- [ ] Mode transition declared (if applicable)

## 14. Integration Points

### 14.1 Agent Prompt Footer
Add to any agent system prompt:
```
## Reasoning Contract
You operate in one of three modes: FAST (0 pulls), DELIBERATE (1 pull), or SLOW (3 pulls).
Declare MODE and JUSTIFICATION at the start of every task.
Use think tool for DELIBERATE/SLOW modes.
Follow evidence budget strictly. Breach = escalation.
```

### 14.2 Orchestrator Routing
Classify before routing:
```
if single-file-edit or trivial-lookup → FAST
if one-unknown or slight-ambiguity → DELIBERATE
if architecture or debugging or 3+-approaches → SLOW
if irreversible or high-stakes-tradeoff → SLOW + council
```

### 14.3 Tool Injection
Add think tool to agent tool definitions:
```json
{
  "name": "think",
  "description": "Structured reasoning scratchpad. Use in DELIBERATE or SLOW mode.",
  "parameters": {
    "mode": "deliberate|slow",
    "gist": "string",
    "evidence_log": [{"tool": "string", "target": "string", "finding": "string"}],
    "disconfirmer": "string",
    "pre_mortem": ["string"],
    "wysiati": "string",
    "decision": "string",
    "terminal": "done|ask|escalate",
    "reflection": {"was_justified": "yes|no|uncertain", "lesson": "string"}
  }
}
```

## 15. Version History

- **v2.0** (2025-04-21): 3-tier model, think tool schema, meta-cognitive feedback, skill compilation
- **v1.0** (2025-04-20): Binary fast/slow, basic Kahneman framework

## 16. References

- Kahneman, D. (2011). *Thinking, Fast and Slow*. Farrar, Straus and Giroux.
- Anthropic. (2025). [The "think" tool](https://www.anthropic.com/engineering/claude-think-tool).
- OpenAI. (2024). [Learning to Reason with LLMs](https://openai.com/index/learning-to-reason-with-llms/).
- Snell et al. (2024). [Scaling LLM Test-Time Compute Optimally](https://arxiv.org/abs/2408.03314).
- DeepMind. (2025). [Gemini Deep Think](https://deepmind.google/blog/accelerating-mathematical-and-scientific-discovery-with-gemini-deep-think/).
