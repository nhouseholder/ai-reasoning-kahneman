## ORCHESTRATOR ROUTING TEMPLATE

Drop-in mode classification and routing layer for orchestrator agents.

### Mode Classification Heuristics

```python
def classify_request(request: str, context: dict) -> dict:
    """
    Classify incoming request into fast/deliberate/slow.
    Returns: {"mode": str, "justification": str, "budget": int}
    """
    
    # FAST patterns
    fast_patterns = [
        r"rename\s+\w+",
        r"format\s+(code|file)",
        r"fix\s+typo",
        r"run\s+(tests?|command)",
        r"git\s+(status|log|diff)",
    ]
    
    # DELIBERATE patterns
    deliberate_patterns = [
        r"check\s+(if|whether)",
        r"verify\s+",
        r"quick\s+(review|check)",
        r"sanity\s+check",
        r"is\s+this\s+correct",
    ]
    
    request_lower = request.lower()
    
    # Check FAST
    for pattern in fast_patterns:
        if re.search(pattern, request_lower):
            return {
                "mode": "fast",
                "justification": "Pattern matches FAST task",
                "budget": 0
            }
    
    # Check DELIBERATE
    for pattern in deliberate_patterns:
        if re.search(pattern, request_lower):
            return {
                "mode": "deliberate",
                "justification": "Task requires verifying one assumption",
                "budget": 1
            }
    
    # Default to SLOW
    return {
        "mode": "slow",
        "justification": "Task requires full analysis",
        "budget": 3
    }
```

### Routing Table

| Request Type | Mode | Agent | Packet Fields |
|-------------|------|-------|--------------|
| Single-file edit, rename, format | FAST | @generalist | `reasoning_mode=fast, verification_depth=light` |
| Verify assumption, quick check | DELIBERATE | @generalist or @auditor | `reasoning_mode=deliberate, verification_depth=standard` |
| Architecture decision | SLOW | @strategist | `reasoning_mode=slow, model_tier=smart` |
| Debug unknown root cause | SLOW | @auditor | `reasoning_mode=slow, verification_depth=deep` |
| UI/UX design | SLOW | @designer | `reasoning_mode=slow, model_tier=smart` |
| Research external API | SLOW | @researcher | `reasoning_mode=slow, budget_class=standard` |
| Codebase exploration | SLOW | @explorer | `reasoning_mode=slow, verification_depth=light` |
| "Should we..." high-stakes | SLOW | @council | `reasoning_mode=slow, model_tier=council` |

### Delegation Packet Builder

```python
def build_packet(mode: str, agent: str, request: str) -> dict:
    """Build delegation packet with mode-appropriate settings"""
    
    packets = {
        "fast": {
            "reasoning_mode": "fast",
            "model_tier": "fast",
            "budget_class": "low",
            "verification_depth": "light"
        },
        "deliberate": {
            "reasoning_mode": "deliberate",
            "model_tier": "smart",
            "budget_class": "standard",
            "verification_depth": "standard"
        },
        "slow": {
            "reasoning_mode": "slow",
            "model_tier": "smart",
            "budget_class": "standard",
            "verification_depth": "deep"
        }
    }
    
    packet = packets[mode].copy()
    packet.update({
        "route_rationale": f"{request[:50]}... → {mode} → @{agent}",
        "scope_boundary": f"Agent owns: {agent_scope[agent]}",
        "stop_condition": "Task complete or budget exhausted",
        "evidence_checked": [],
        "open_unknowns": [],
        "escalation_rule": "Escalate to orchestrator on budget breach or fatal flaw"
    })
    
    return packet
```

### Council Gate

```python
def should_use_council(request: str, context: dict) -> bool:
    """
    Council ONLY fires when:
    1. User explicitly requests it
    2. Decision is irreversible (data loss, migration, rewrite)
    3. Decision has 2+ genuinely competing paths AND high cost if wrong
    """
    
    explicit_request = any(kw in request.lower() for kw in 
                          ["council", "debate", "multi-model", "fan out"])
    
    irreversible = any(kw in request.lower() for kw in 
                      ["migrate", "rewrite", "delete", "schema change"])
    
    competing_paths = request.count(" or ") >= 2
    high_stakes = any(kw in request.lower() for kw in 
                     ["production", "critical", "security", "auth"])
    
    return explicit_request or (irreversible and competing_paths) or (competing_paths and high_stakes)
```
