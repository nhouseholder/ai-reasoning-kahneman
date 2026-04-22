## AutoGen Integration

Adapter pattern for Microsoft AutoGen multi-agent systems.

### 1. User Proxy Agent with Mode Classification

```python
from autogen import UserProxyAgent, AssistantAgent

class ReasoningUserProxy(UserProxyAgent):
    def classify_mode(self, message: str) -> str:
        """Classify request into fast/deliberate/slow"""
        fast_keywords = ["rename", "format", "typo", "edit", "fix"]
        deliberate_keywords = ["check", "verify", "review", "quick"]
        
        msg_lower = message.lower()
        if any(kw in msg_lower for kw in fast_keywords):
            return "fast"
        elif any(kw in msg_lower for kw in deliberate_keywords):
            return "deliberate"
        return "slow"
    
    def generate_reply(self, messages, sender, config):
        # Add mode classification to message metadata
        last_msg = messages[-1]["content"]
        mode = self.classify_mode(last_msg)
        
        return {
            "content": last_msg,
            "mode": mode,
            "budget": {"fast": 0, "deliberate": 1, "slow": 3}[mode]
        }
```

### 2. Assistant Agent with Budget Tracking

```python
class ReasoningAssistant(AssistantAgent):
    def __init__(self, name, system_message, **kwargs):
        super().__init__(name, system_message, **kwargs)
        self.pulls = 0
        self.mode = "fast"
        self.budget = 0
    
    def receive(self, message, sender, request_reply=True, silent=False):
        # Extract mode and budget from message
        if isinstance(message, dict):
            self.mode = message.get("mode", "fast")
            self.budget = message.get("budget", 0)
            self.pulls = 0
        
        super().receive(message, sender, request_reply, silent)
    
    def generate_reply(self, messages, sender, config):
        # Add think tool requirement for deliberate/slow modes
        if self.mode in ["deliberate", "slow"]:
            self.system_message += "\n\nUse the think tool before responding."
        
        return super().generate_reply(messages, sender, config)
```

### 3. Group Chat with Mode Negotiation

```python
from autogen import GroupChat

class ReasoningGroupChat(GroupChat):
    def select_speaker(self, last_speaker, selector):
        # Mode-aware speaker selection
        last_message = self.messages[-1] if self.messages else {}
        mode = last_message.get("mode", "fast")
        
        if mode == "slow" and last_speaker.name != "critic":
            # Force critic agent for disconfirmation
            return next(a for a in self.agents if a.name == "critic")
        
        return super().select_speaker(last_speaker, selector)
```

### 4. Think Tool as Code Execution

```python
# Register think tool as a code execution function
def think_tool(
    mode: str,
    gist: str,
    evidence_log: list,
    disconfirmer: str,
    wysiati: str,
    decision: str,
    terminal: str,
    reflection: dict
):
    """Structured reasoning scratchpad"""
    return {
        "validated": True,
        "pulls": len(evidence_log),
        "terminal": terminal
    }

assistant.register_function(
    function_map={"think": think_tool}
)
```

### Example

See `examples/autogen_example.py` for a complete multi-agent setup.
