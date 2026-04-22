## OpenCode Integration

Drop-in integration for OpenCode agent systems.

### 1. Agent Config

Add to your `opencode.json`:

```json
{
  "agent": {
    "my-agent": {
      "mode": "all",
      "prompt_file": "path/to/agent-prompt-footer.md"
    }
  }
}
```

### 2. Add Think Tool

Inject the think tool into your agent's tool definitions. The tool is invoked automatically in DELIBERATE/SLOW modes.

```json
{
  "tools": {
    "think": {
      "description": "Structured reasoning scratchpad. Use in DELIBERATE or SLOW mode.",
      "parameters": {
        "mode": {"type": "string", "enum": ["deliberate", "slow"]},
        "gist": {"type": "string"},
        "evidence_log": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "tool": {"type": "string"},
              "target": {"type": "string"},
              "finding": {"type": "string"}
            }
          }
        },
        "disconfirmer": {"type": "string"},
        "pre_mortem": {"type": "array", "items": {"type": "string"}},
        "wysiati": {"type": "string"},
        "decision": {"type": "string"},
        "terminal": {"type": "string", "enum": ["done", "ask", "escalate"]},
        "reflection": {
          "type": "object",
          "properties": {
            "was_justified": {"type": "string", "enum": ["yes", "no", "uncertain"]},
            "lesson": {"type": "string"}
          }
        }
      },
      "required": ["mode", "gist", "disconfirmer", "wysiati", "terminal", "reflection"]
    }
  }
}
```

### 3. Prompt Integration

Add the agent prompt footer (see `templates/agent-prompt-footer.md`) to the end of each agent's system prompt.

### 4. Mode Classification

Use the orchestrator routing template (see `templates/orchestrator-routing.md`) in your primary agent to classify requests before dispatching.

### 5. Memory Integration

Save mode calibration data to engram:

```javascript
engram_mem_save({
  title: "Mode calibration: [task-type]",
  content: "MODE_CALIBRATION:\n  task_type: [desc]\n  mode: [deliberate|slow]\n  pulls: [N]\n  outcome: [success|partial|failure]\n  was_justified: [yes|no]",
  type: "learning",
  topic_key: "reasoning/calibration"
});
```

### Example Config

See `examples/minimal.json` for a complete single-agent setup.
