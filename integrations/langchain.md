## LangChain Integration

Adapter pattern for LangChain/LangGraph agents.

### 1. Custom Callback Handler

Track evidence pulls and enforce budgets:

```python
from langchain.callbacks.base import BaseCallbackHandler

class ReasoningBudgetHandler(BaseCallbackHandler):
    def __init__(self, mode: str = "fast"):
        self.mode = mode
        self.pulls = 0
        self.budgets = {"fast": 0, "deliberate": 1, "slow": 3}
    
    def on_tool_start(self, serialized, input_str, **kwargs):
        # Count evidence pulls
        if serialized["name"] in ["read", "grep", "search", "fetch"]:
            self.pulls += 1
            if self.pulls > self.budgets[self.mode]:
                raise BudgetExceededError(
                    f"Mode {self.mode} budget exceeded: {self.pulls} pulls"
                )
```

### 2. Mode Router

Classify requests before routing:

```python
from langchain_core.runnables import RunnableBranch

def classify_mode(request: str) -> str:
    """Classify request into fast/deliberate/slow"""
    # Heuristic classification
    if any(kw in request.lower() for kw in ["rename", "format", "typo"]):
        return "fast"
    elif any(kw in request.lower() for kw in ["check", "verify", "quick"]):
        return "deliberate"
    else:
        return "slow"

mode_router = RunnableBranch(
    (lambda x: x["mode"] == "fast", fast_chain),
    (lambda x: x["mode"] == "deliberate", deliberate_chain),
    slow_chain
)
```

### 3. Think Tool as Structured Output

Use Pydantic for structured think tool output:

```python
from pydantic import BaseModel, Field
from typing import List, Literal

class EvidencePull(BaseModel):
    tool: str
    target: str
    finding: str

class Reflection(BaseModel):
    was_justified: Literal["yes", "no", "uncertain"]
    lesson: str

class ThinkToolOutput(BaseModel):
    mode: Literal["deliberate", "slow"]
    gist: str
    evidence_log: List[EvidencePull]
    disconfirmer: str
    pre_mortem: List[str] = Field(default_factory=list)
    wysiati: str
    decision: str
    terminal: Literal["done", "ask", "escalate"]
    reflection: Reflection
```

### 4. Integration with LCEL

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

# Add reasoning instructions to system prompt
system_prompt = ChatPromptTemplate.from_messages([
    ("system", open("templates/agent-prompt-footer.md").read()),
    ("human", "{request}")
])

# Use structured output for think tool
model = ChatOpenAI(model="gpt-4o").with_structured_output(ThinkToolOutput)

chain = system_prompt | model
```

### Example

See `examples/langchain_example.py` for a complete implementation.
