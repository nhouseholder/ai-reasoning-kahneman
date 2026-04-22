## THINK TOOL PROMPT

Inject this into your agent's tool definitions or prompt to enable structured reasoning.

### For OpenCode / Tool-Based Systems

```json
{
  "name": "think",
  "description": "Structured reasoning scratchpad. Use in DELIBERATE or SLOW mode before acting. Validates reasoning quality.",
  "parameters": {
    "type": "object",
    "properties": {
      "mode": {
        "type": "string",
        "enum": ["deliberate", "slow"],
        "description": "Current reasoning mode"
      },
      "gist": {
        "type": "string",
        "description": "1-sentence decision-bearing summary"
      },
      "evidence_log": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "tool": {"type": "string", "description": "Tool name used"},
            "target": {"type": "string", "description": "What was queried"},
            "finding": {"type": "string", "description": "What was learned"}
          },
          "required": ["tool", "target", "finding"]
        },
        "description": "Must match actual tool calls. One entry per pull."
      },
      "disconfirmer": {
        "type": "string",
        "description": "One competing explanation or falsifier. Mandatory."
      },
      "pre_mortem": {
        "type": "array",
        "items": {"type": "string"},
        "description": "2-3 reasons this plan could fail. Mandatory in SLOW mode."
      },
      "wysiati": {
        "type": "string",
        "description": "What critical evidence is still missing? 'Nothing' is a WYSIATI violation."
      },
      "decision": {
        "type": "string",
        "description": "Chosen approach with explicit trade-offs"
      },
      "terminal": {
        "type": "string",
        "enum": ["done", "ask", "escalate"],
        "description": "Terminal state"
      },
      "reflection": {
        "type": "object",
        "properties": {
          "was_justified": {
            "type": "string",
            "enum": ["yes", "no", "uncertain"]
          },
          "would_fast_have_sufficed": {
            "type": "string",
            "enum": ["yes", "no", "uncertain"]
          },
          "lesson": {
            "type": "string",
            "description": "1 sentence for future calibration"
          }
        },
        "required": ["was_justified", "lesson"]
      }
    },
    "required": ["mode", "gist", "disconfirmer", "wysiati", "terminal", "reflection"]
  }
}
```

### For Prompt-Only Systems

Add this to your system prompt:

```markdown
## THINK TOOL (DELIBERATE and SLOW modes only)

Before acting, output a structured reasoning block:

<think>
MODE: [deliberate|slow]
GIST: [1-sentence decision]
EVIDENCE_LOG:
  - Pull N: [tool] [target] → [finding]
DISCONFIRMER: [one competing explanation]
PRE_MORTEM:  # SLOW only
  - [Reason 1 plan could fail]
  - [Reason 2 plan could fail]
WYSIATI: [what critical evidence is missing?]
DECISION: [chosen approach with trade-offs]
TERMINAL: [done|ask|escalate]
REFLECTION:
  Was justified: [yes|no|uncertain]
  Would fast have sufficed: [yes|no|uncertain]
  Lesson: [1 sentence]
</think>

**Rules:**
- EVIDENCE_LOG must match actual tool calls you make
- DISCONFIRMER is mandatory. If you can't name one, you haven't thought critically enough.
- WYSIATI: "Nothing is missing" is a violation. Lower confidence.
- REFLECTION is mandatory. Save the lesson for calibration.
```
