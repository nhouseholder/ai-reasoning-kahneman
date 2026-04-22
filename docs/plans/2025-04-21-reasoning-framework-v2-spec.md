# Kahneman Agent Reasoning Framework v2 — Refinement Plan

**Status:** Strategic Plan  
**Scope:** Transform `ai-reasoning-kahneman` into a standalone, world-class reasoning framework portable to any agent architecture.  
**Target:** Not tied to the 8-agent-team. Usable by solo prompt engineers, LangChain devs, AutoGen builders, and Claude Code custom agents.

---

## 1. File Structure

```
ai-reasoning-kahneman/
├── README.md                                 # World-class overview + quickstart (5 min to integrate)
├── docs/
│   ├── 01-core-contract.md                   # The reasoning contract: modes, signals, budgets, gates
│   ├── 02-mode-state-machine.md              # States, transitions, terminal conditions
│   ├── 03-evidence-budget.md                 # What counts as an evidence pull; how to count and bound
│   ├── 04-verification-signals.md            # The confidence gate: 5 objective signals
│   ├── 05-meta-cognition.md                  # Post-task mode-evaluation + feedback loop
│   ├── 06-graduated-reasoning.md             # FAST vs DELIBERATE vs SLOW (3-tier, not binary)
│   ├── 07-implementation-guide.md            # Step-by-step porting guide for any agent system
│   ├── 08-integration-patterns.md            # Prompt templates, tool wrappers, memory hooks
│   ├── 09-compliance-checklist.md            # Audit your own implementation
│   ├── 10-theory-reference.md                # Kahneman + frontier LLM research synthesis
│   └── 11-examples-walkthrough.md            # 3 end-to-end examples with mode transition logs
├── templates/
│   ├── agent-prompt-footer.md                # Drop-in footer for any agent prompt
│   ├── orchestrator-mode-classifier.md       # Drop-in mode selection logic for router/orchestrator
│   ├── think-tool-schema.json                # Structured scratchpad for System 2 reasoning
│   ├── mode-transition-log.yaml              # Structured trace template
│   └── completion-gate-checklist.md          # Pre-completion verification checklist
├── examples/
│   ├── python-langchain/
│   │   ├── mode_classifier.py
│   │   ├── evidence_counter.py
│   │   └── agent_with_karp.py
│   ├── typescript-autogen/
│   │   ├── modeClassifier.ts
│   │   ├── evidenceBudget.ts
│   │   └── agentWithKARP.ts
│   └── claude-code/
│       ├── .claude/CLAUDE.md                 # Example CLAUDE.md integrating the framework
│       └── mode-transitions/                 # Sample mode transition logs
├── tests/
│   └── validate-reasoning-scenarios.js       # Compliance test suite (enhanced from 8-agent-team audit)
├── archive/                                  # Current flat files moved here for reference
│   ├── v1-orchestrator.md
│   ├── v1-generalist.md
│   ├── v1-explorer.md
│   └── v1-memory-systems.md
└── CHANGELOG.md
```

### File Rationale

- **Binary → 3-tier:** The old repo treated fast/slow as binary. The new `06-graduated-reasoning.md` introduces a `DELIBERATE` tier (System 1.5) that maps to Claude's think tool and OpenAI's `reasoning_effort=medium`.
- **Operational enforceability:** `03-evidence-budget.md` and `04-verification-signals.md` turn suggestions into countable, verifiable rules.
- **Portability:** `templates/` contains copy-paste artifacts with zero references to `@generalist`, `@strategist`, or other 8-agent-team concepts.
- **Compliance:** `tests/validate-reasoning-scenarios.js` lets any adopter verify their implementation against the spec.

---

## 2. Core Framework Specification

### 2.1 The Reasoning Contract (Every Agent Must Obey)

The contract has 8 enforceable rules. Every rule is measurable.

#### Rule 1: Intent Lock
Before any mode selection or evidence pull, lock the objective, deliverable, and stop condition.  
**Measurement:** The mode transition log must contain an `intent_locked_at` timestamp and a `stop_condition` string.  
**Reopening allowed only if:** (a) explicit user correction, (b) materially new evidence, or (c) verification failure.

#### Rule 2: Mode Selection Signals
The agent does not choose its mode. The task + environment choose it via signals.

| Mode | Activation Signals | Inhibitors |
|---|---|---|
| **FAST** | Narrow scope, familiar pattern, low risk, single-file or single-tool | Novel domain, ambiguous scope, user said "plan this", test failure |
| **DELIBERATE** | 2-3 valid approaches, moderate stakes, need structured thinking but bounded scope | Cross-system impact, unknown root cause, security domain |
| **SLOW** | Novel problem, high stakes, security/auth, cross-system, verification failed on first attempt, user explicitly requested deliberation | Clear pattern exists, single-file cosmetic |

**Measurement:** The mode transition log must record which signal fired.

#### Rule 3: Evidence Budget
An "evidence pull" is any action that retrieves external information into the agent's context: file read, directory listing, memory search, web fetch, database query, API call. Pattern-matching against existing context is NOT an evidence pull.

| Mode | Evidence Budget | Self-Correction Limit |
|---|---|---|
| **FAST** | 0–1 pulls (confirm only) | 0 (if confirm fails → escalate to DELIBERATE) |
| **DELIBERATE** | Up to 3 pulls | 1 cycle max |
| **SLOW** | Up to 7 pulls in research phase; 3 per execution phase | 2 cycles max |

**Measurement:** The framework provides an `evidence_counter` primitive. The agent must increment it on every pull. Breaching the budget is a mandatory escalation trigger.

#### Rule 4: Single Forward Pass (No Loops on Unchanged Evidence)
System 2 reasoning is a single forward pass with a visible start and a hard stop.  
**Phases:** Scope → Evidence → Disconfirm → Decision → Act → Verify.  
**Terminal states:** `DONE`, `ASK`, `ESCALATE`.  
**Measurement:** The think-tool schema includes a `phase` field. Returning to an earlier phase on unchanged evidence is a contract violation.

#### Rule 5: Anti-WYSIATI Check (Mandatory Before Completion)
Before claiming completion in DELIBERATE or SLOW mode, answer:
1. What critical evidence is still missing?
2. What competing explanation still fits the current evidence?
3. What memory, assumption, or prior pattern could be stale?
4. What concrete file, test, or external source would falsify my solution?

**Measurement:** The think-tool schema has a `wysiati_check` object with 4 fields. All must be non-empty before `phase: verify` can transition to `terminal: done`.

#### Rule 6: Verification Signal Gate
Confidence is computed, not self-reported. Before claiming completion, check 5 signals:

| Signal | Check | Green | Red |
|---|---|---|---|
| **tool_call_coverage** | Did I use the right tools for the task? | Used all relevant tools | Skipped verification tools |
| **test_pass_rate** | Do tests pass? | All pass or none exist | Tests fail or were skipped |
| **lsp_clean** | Any static errors in changed files? | Clean | Errors found |
| **output_scope_ratio** | Did I address everything requested? | All requirements met | Partial / TODOs left |
| **mode_compliance** | Did I stay within my evidence budget and phase rules? | Budget unbreached, no backward loops | Budget breached or loop detected |

**Measurement:** The completion gate checklist requires a boolean for each signal. Any red → fix or escalate. Never claim completion with red signals.

#### Rule 7: Meta-Cognitive Evaluation (Post-Task)
After any DELIBERATE or SLOW task, evaluate:
1. **Mode calibration:** Was the chosen mode appropriate? (correct / overkill / underkill)
2. **Evidence efficiency:** Did I use the right amount of evidence? (optimal / excessive / insufficient)
3. **Outcome quality:** Did the extra reasoning yield a measurably better result? (yes / no / unknown)
4. **Skill candidacy:** Should this pattern be cached for FAST mode next time? (yes / no)

**Measurement:** The mode transition log includes a `meta_evaluation` object. Framework implementations must persist this to memory.

#### Rule 8: Skill Compilation (System 2 → System 1)
When a DELIBERATE or SLOW task produces a reusable pattern, cache it. The pattern must include: trigger conditions, evidence strategy, decision template, verification steps.  
**Measurement:** Memory system contains a `skills/` namespace. FAST mode must query skills before escalating.

---

### 2.2 Mode State Machine

```
                    +-----------+
     +------------->|   IDLE    |<--------------+
     |              +-----------+               |
     |                   |                      |
     |     (task arrives, no signal)            |
     |                   v                      |
     |              +-----------+   (signal fires)  |
     |   +--------->|   FAST    |------------------+
     |   |          +-----------+      |
     |   |   (confirm fails /       (1 evidence pull max)
     |   |    surprise / error)         |
     |   |              |               |
     |   |              v               |
     |   |         +-----------+   (signal persists or escalates)
     |   +---------| DELIBERATE|--------------+
     |   (return)  +-----------+      |       |
     |   (success)     |              |       |
     |                 | (success)    |       |
     |                 v              |       |
     |            +---------+         |       |
     +------------|  DONE   |<--------+       |
                  +---------+                 |
                                              v
                                         +-----------+
                                         |   SLOW    |
                                         +-----------+
                                              |
                                              v
                                         +---------+
                                         | DONE /  |
                                         | ASK /   |
                                         | ESCALATE|
                                         +---------+
```

**Critical transitions:**
- **FAST → DELIBERATE:** Automatic on any trigger signal. No agent choice.
- **DELIBERATE → SLOW:** Triggered if DELIBERATE budget (3 pulls) is exhausted without resolution, OR if the anti-WYSIATI check reveals a cross-system dependency, OR if self-correction cycle is used and still fails.
- **Any → FAST (return):** After DELIBERATE or SLOW success, the agent returns to FAST for the next independent subtask. Never stay in SLOW across unrelated tasks.
- **Any → ASK:** Terminal. Agent needs user input to proceed.
- **Any → ESCALATE:** Terminal. Agent lacks capability or context; hand off to a more capable agent or human.

---

### 2.3 Think Tool Schema (Universal System 2 Scratchpad)

Inspired by Claude's explicit "think" tool. Every System 2 invocation must produce a structured think trace.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "KARPThinkTool",
  "type": "object",
  "required": ["mode", "phase", "intent", "evidence_log", "wysiati_check", "terminal"],
  "properties": {
    "mode": { "enum": ["DELIBERATE", "SLOW"] },
    "phase": { "enum": ["scope", "evidence", "disconfirm", "decision", "act", "verify"] },
    "intent": {
      "type": "object",
      "properties": {
        "objective": { "type": "string", "maxLength": 200 },
        "deliverable": { "type": "string", "maxLength": 100 },
        "stop_condition": { "type": "string", "maxLength": 200 },
        "locked_at": { "type": "string", "format": "date-time" }
      },
      "required": ["objective", "deliverable", "stop_condition"]
    },
    "evidence_log": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "pull_id": { "type": "integer" },
          "tool": { "type": "string" },
          "target": { "type": "string" },
          "gist_found": { "type": "string", "maxLength": 200 },
          "changed_decision": { "type": "boolean" }
        },
        "required": ["pull_id", "tool", "target"]
      },
      "maxItems": 7
    },
    "gist": { "type": "string", "maxLength": 300, "description": "Shortest decision-bearing summary" },
    "disconfirmer": {
      "type": "object",
      "properties": {
        "competing_explanation": { "type": "string" },
        "fatal_flaw_test": { "type": "string" },
        "what_would_kill_this_plan": { "type": "string" }
      }
    },
    "decision": {
      "type": "object",
      "properties": {
        "chosen_approach": { "type": "string" },
        "tradeoffs": { "type": "string" },
        "confidence": { "enum": ["high", "medium", "low"] }
      }
    },
    "wysiati_check": {
      "type": "object",
      "properties": {
        "missing_evidence": { "type": "string" },
        "competing_explanation": { "type": "string" },
        "stale_assumption": { "type": "string" },
        "falsifying_test": { "type": "string" }
      },
      "required": ["missing_evidence", "competing_explanation", "stale_assumption", "falsifying_test"]
    },
    "terminal": {
      "type": "object",
      "properties": {
        "state": { "enum": ["done", "ask", "escalate"] },
        "reason": { "type": "string" }
      }
    }
  }
}
```

**Enforcement:** The framework provides a validator function. Agents operating in DELIBERATE or SLOW mode must pass their think output through this schema. Invalid think traces = mode compliance failure.

---

## 3. Implementation Guide

### 3.1 Porting Checklist (Any Agent System)

**Step 1: Add Mode Classification**
Insert the mode classifier at the entry point of your agent/orchestrator. It runs before any tool use.

```python
# Pseudo-code (language-agnostic)
def classify_mode(task_descriptor, memory_lookup_result, past_pattern_exists: bool) -> Mode:
    if task_descriptor.user_explicitly_requested_deliberation:
        return Mode.SLOW
    if task_descriptor.novel_domain or task_descriptor.security_related:
        return Mode.SLOW
    if task_descriptor.cross_system_impact:
        return Mode.SLOW
    if past_pattern_exists and task_descriptor.narrow_scope:
        return Mode.FAST
    if task_descriptor.two_or_more_approaches or task_descriptor.ambiguous_scope:
        return Mode.DELIBERATE
    return Mode.FAST
```

**Step 2: Instrument Evidence Counting**
Wrap every tool that retrieves external data.

```python
class EvidenceCounter:
    def __init__(self, budget: int):
        self.budget = budget
        self.pulls = []

    def pull(self, tool: str, target: str) -> Any:
        if len(self.pulls) >= self.budget:
            raise EvidenceBudgetExceeded(f"Budget {self.budget} exhausted. Escalate required.")
        result = tool.execute(target)
        self.pulls.append({"tool": tool, "target": target, "gist": summarize(result)})
        return result
```

**Step 3: Embed the Think Tool**
For DELIBERATE and SLOW tasks, require the agent to produce a structured think trace matching the JSON schema. Parse it. Validate it. If invalid, halt and ask the agent to re-think.

**Step 4: Implement the Completion Gate**
Before the agent emits its final response, run the 5-signal check. Block completion if any signal is red.

```python
class CompletionGate:
    def check(self, task, evidence_counter, think_trace, tests_result, lsp_result) -> GateResult:
        signals = {
            "tool_call_coverage": self._check_tools(task, evidence_counter),
            "test_pass_rate": tests_result,
            "lsp_clean": lsp_result,
            "output_scope_ratio": self._check_scope(task, think_trace),
            "mode_compliance": self._check_compliance(evidence_counter, think_trace)
        }
        reds = [k for k, v in signals.items() if v == Signal.RED]
        return GateResult(pass_=len(reds)==0, reds=reds)
```

**Step 5: Add Meta-Cognitive Evaluation**
After task completion (especially for DELIBERATE/SLOW), append a meta-evaluation block to your memory/ledger.

**Step 6: Add Compliance Tests**
Run `tests/validate-reasoning-scenarios.js` against your agent's prompts and logs. Fix failures.

### 3.2 Integration by Architecture Type

| Architecture | Integration Point | Key File to Modify |
|---|---|---|
| **Single LLM call** | System prompt + stop sequences | Add `templates/agent-prompt-footer.md` to system prompt |
| **LangChain/LangGraph** | Custom tool + state graph node | Add `EvidenceCounter` as a tool wrapper; add `think` node to graph |
| **AutoGen** | Agent capability + reply function | Add `think` capability to agents; add `mode_classifier` to `select_speaker` |
| **Claude Code** | `.claude/CLAUDE.md` + custom skills | Rewrite `CLAUDE.md` to import framework; use skills for mode enforcement |
| **OpenAI Assistants** | Tool definitions + run step hooks | Add `think` tool; use `reasoning_effort` param mapped to framework modes |

---

## 4. Integration Patterns

### 4.1 Agent Prompt Footer (Drop-In)

```markdown
## Reasoning Protocol (KARP)

### Mode Classification
Before acting, classify this task into one mode:
- **FAST**: Narrow, familiar, low-risk. Pattern match → act → verify. Budget: 1 evidence pull.
- **DELIBERATE**: 2+ approaches, moderate stakes. Think once, challenge once, act. Budget: 3 evidence pulls.
- **SLOW**: Novel, high-stakes, cross-system. Research → plan → act → verify. Budget: 7 + 3 per phase.

### Evidence Budget Rules
- An "evidence pull" = any file read, search, web fetch, or data retrieval.
- Count your pulls. Do not exceed your mode's budget.
- If you exhaust your budget without resolution, ESCALATE.

### Anti-WYSIATI Check (Mandatory before completion in DELIBERATE/SLOW)
1. What critical evidence is still missing?
2. What competing explanation still fits?
3. What assumption or memory could be stale?
4. What test or source would falsify my solution?

### Completion Gate (Mandatory before claiming done)
Check all 5 signals. If any are red, fix or escalate. Never claim completion on red signals.
- [ ] tool_call_coverage
- [ ] test_pass_rate
- [ ] lsp_clean
- [ ] output_scope_ratio
- [ ] mode_compliance

### Meta-Evaluation (Mandatory after DELIBERATE/SLOW)
- Was the mode appropriate? (correct / overkill / underkill)
- Was evidence usage optimal? (optimal / excessive / insufficient)
- Should this pattern become a FAST skill? (yes / no)
```

### 4.2 Orchestrator Mode Classifier (Drop-In)

```markdown
## Mode Classification Protocol

For every incoming request, run this classifier BEFORE routing or acting:

1. **Explicit signal?** User said "plan this", "think through", "should we" → SLOW
2. **Novelty signal?** No past pattern in memory for this task type → SLOW
3. **Security signal?** Touches auth, secrets, user input, payment → SLOW
4. **Cross-system signal?** Affects 3+ independent modules → SLOW
5. **Ambiguity signal?** 2+ valid approaches, unclear scope → DELIBERATE
6. **Surprise signal?** Previous attempt failed, unexpected output → DELIBERATE or SLOW
7. **Familiar + narrow?** Past pattern exists, single file, clear scope → FAST

**Delegation metadata packet:** When dispatching to any agent, include:
- `reasoning_mode`: FAST | DELIBERATE | SLOW
- `model_tier`: fast-execution | balanced | reasoning-heavy | long-context
- `budget_class`: token_budget (e.g., 4k | 16k | 128k)
- `verification_depth`: quick | standard | deep
- `evidence_budget`: integer (1 | 3 | 7+)

**Model-aware damping:**
- If `model_tier=fast-execution` and mode=SLOW, warn: "Slow mode on fast model. Consider DELIBERATE unless stakes demand depth."
- If `model_tier=reasoning-heavy` and mode=FAST, warn: "Fast mode on reasoning model. Ensure task is truly narrow."
```

### 4.3 Memory Hooks

**Pre-Task:**
- Query skill cache: "Has this exact pattern been solved before?"
- If skill hit → preload pattern into context, default to FAST.

**Post-Task:**
- If mode was DELIBERATE/SLOW and outcome was good → compile skill.
- If mode was SLOW but outcome was trivial → flag as overkill for future calibration.
- Save meta-evaluation to memory under `karp/calibration/<task_type>`.

---

## 5. Key Innovations

### 5.1 Evidence Budget as First-Class Primitive
Unlike chain-of-thought or "think step by step," this framework makes **evidence consumption countable and bounded**. An agent cannot infinitely research. The budget is enforced at the tool layer, not suggested in a prompt.

### 5.2 Graduated Deliberate Mode (System 1.5)
Binary fast/slow forces a false dichotomy. The `DELIBERATE` tier captures the reality of Claude's think tool and OpenAI's medium reasoning effort: structured thinking with a tight bound. It prevents both WYSIATI (FAST failure) and analysis paralysis (SLOW failure).

### 5.3 Think Tool Schema (Structured Reasoning Trace)
Instead of free-text chain-of-thought, agents produce a **structured, validatable reasoning trace** with mandatory sections: gist, evidence log, disconfirmer, WYSIATI check, terminal state. This makes reasoning auditable and machine-verifiable.

### 5.4 Meta-Cognitive Feedback Loop
Agents evaluate their own mode choice after the fact. Over time, the system learns:
- Which task types are consistently over- or under-thought
- Whether the evidence budget is calibrated correctly
- Which patterns should graduate from SLOW → DELIBERATE → FAST
This is **empirical reasoning calibration**, not guesswork.

### 5.5 Mode Compliance as a Verification Signal
Existing completion gates check tools, tests, and LSP. This framework adds **mode_compliance**: did the agent stay within its evidence budget? Did it loop backward on unchanged evidence? Did it skip the WYSIATI check? This closes the enforcement gap.

### 5.6 Portability by Design
Every artifact is agent-agnostic. No `@generalist`, no `@strategist`, no 22-step decision tree. The framework is a **cognitive kernel** that any system can embed via prompt sections, tool wrappers, and state machine logic.

---

## 6. Migration Plan

### Phase 1: Archive Current State (Day 1)
1. Move current flat files to `archive/`:
   - `orchestrator.md` → `archive/v1-orchestrator.md`
   - `generalist.md` → `archive/v1-generalist.md`
   - `explorer.md` → `archive/v1-explorer.md`
   - `memory-systems.md` → `archive/v1-memory-systems.md`
2. Tag git: `git tag v1.0-flat-files`
3. Update `README.md` with v2 framing and links to new docs.

### Phase 2: Write Core Specification (Day 1–2)
1. Write `docs/01-core-contract.md` (the 8 rules)
2. Write `docs/02-mode-state-machine.md` (state diagram + transitions)
3. Write `docs/03-evidence-budget.md` (definitions + counting)
4. Write `docs/04-verification-signals.md` (5-signal gate)
5. Write `docs/06-graduated-reasoning.md` (FAST/DELIBERATE/SLOW)

### Phase 3: Build Templates (Day 2)
1. Write `templates/agent-prompt-footer.md`
2. Write `templates/orchestrator-mode-classifier.md`
3. Write `templates/think-tool-schema.json`
4. Write `templates/mode-transition-log.yaml`
5. Write `templates/completion-gate-checklist.md`

### Phase 4: Write Guides (Day 3)
1. Write `docs/07-implementation-guide.md`
2. Write `docs/08-integration-patterns.md`
3. Write `docs/09-compliance-checklist.md`
4. Write `docs/10-theory-reference.md`

### Phase 5: Create Examples (Day 3–4)
1. `examples/python-langchain/` — working minimal example
2. `examples/typescript-autogen/` — working minimal example
3. `examples/claude-code/` — example `.claude/CLAUDE.md`

### Phase 6: Compliance Test Suite (Day 4)
1. Port and enhance `tests/validate-reasoning-scenarios.js` from 8-agent-team.
2. Add new scenarios:
   - "evidence budget enforced"
   - "think tool schema validated"
   - "meta-evaluation persisted"
   - "mode compliance signal checked"
   - "anti-WYSIATI mandatory in DELIBERATE/SLOW"
3. Ensure tests pass against the new docs/templates.

### Phase 7: Refactor 8-Agent-Team Integration (Day 5 — separate from this repo)
1. In the 8-agent-team repo, replace the inline fast/slow reasoning with `@compose:insert` references to this framework.
2. Remove duplication between `orchestrator.md` and `_shared/cognitive-kernel.md`.
3. Add `mode_compliance` to the agent completion gate.
4. Point 8-agent-team README to this repo as its reasoning dependency.

### Critical Cleanup During Migration
- **Remove all agent-specific references** from core framework files. No `@generalist`, no `@strategist`.
- **Remove the 22-step decision tree** from core. It belongs in the 8-agent-team orchestrator, not in a portable reasoning framework.
- **Deduplicate:** The old `orchestrator.md` and `_shared/cognitive-kernel.md` (in 8-agent-team) have massive overlap. The new framework has a single source of truth: `docs/01-core-contract.md`.
- **Preserve valuable content:** The pre-escalation memory check and skill compilation protocol from `generalist.md` are excellent. Generalize them and move to `docs/05-meta-cognition.md` and `docs/07-implementation-guide.md`.

---

## Appendix: Addressing the 10 Critical Gaps

| Gap | Fix in v2 |
|---|---|
| 1. No meta-cognitive feedback loop | `docs/05-meta-cognition.md` + mandatory `meta_evaluation` in mode log |
| 2. No operational definition of evidence pull | `docs/03-evidence-budget.md` defines pull + provides `EvidenceCounter` primitive |
| 3. Binary fast/slow only | `docs/06-graduated-reasoning.md` introduces DELIBERATE tier |
| 4. No mode state machine | `docs/02-mode-state-machine.md` with explicit states/transitions |
| 5. No empirical cost/benefit tracking | Meta-evaluation captures mode appropriateness + evidence efficiency |
| 6. Content duplication | Single source of truth in `01-core-contract.md`; orchestrator imports via compose |
| 7. No inter-agent mode negotiation | Framework is agent-agnostic; negotiation handled by orchestrator using framework metadata |
| 8. No explicit exit slow mode protocol | State machine defines `Any → FAST` return after task completion |
| 9. WYSIATI check not adopted by all agents | Mandatory in think-tool schema; completion gate checks for it |
| 10. No fast/slow compliance in completion gate | `mode_compliance` added as 5th verification signal |

---

## Decision: Why Not Merge with 8-Agent-Team?

**Option A (rejected):** Merge framework into 8-agent-team as a monorepo package.  
**Option B (chosen):** Keep `ai-reasoning-kahneman` as a standalone repo.

**Rationale for B:**
- A reasoning framework has broader utility than one agent team. It should be star-able, forkable, and usable by LangChain/AutoGen/Claude Code communities.
- The 8-agent-team has fast-moving orchestrator logic that would pollute the stable reasoning contract.
- Versioning: the framework can evolve on its own release cycle.
- The 8-agent-team should **depend on** this framework, not contain it.
