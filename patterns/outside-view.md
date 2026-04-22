## OUTSIDE VIEW & BASE RATES

From Kahneman: When forecasting, ignore specific details initially and first ask "What is the base rate for this class?"

### Purpose
Prevents anchoring bias by forcing base-rate reasoning before inside-view specifics.

### When to Use
- Estimating task duration
- Forecasting outcomes
- Predicting success/failure rates
- Comparing options probabilistically

### Process
1. **Outside view:** What is the base rate for tasks/projects of this class?
   - Ignore the specific details of this case
   - Use historical data, industry averages, or past experience
2. **Inside view:** Adjust for the specific details
   - Only after anchoring on base rate
   - Document each adjustment and rationale
3. **Final estimate:** Base rate + adjustments

### Template
```
QUESTION: [What are we estimating?]

OUTSIDE VIEW (Base Rate):
- Historical average for similar tasks: [X]
- Source: [memory / data / experience]
- Confidence in base rate: [high/medium/low]

INSIDE VIEW (Adjustments):
- Factor 1: [Specific detail] → Adjustment: [+/- X] → Rationale: [Why]
- Factor 2: [Specific detail] → Adjustment: [+/- X] → Rationale: [Why]
- Factor 3: [Specific detail] → Adjustment: [+/- X] → Rationale: [Why]

FINAL ESTIMATE: [Base rate + adjustments]
RANGE: [Lower bound — Upper bound]
CONFIDENCE: [high/medium/low]
```

### Example
```
QUESTION: "How long will this refactor take?"

OUTSIDE VIEW:
- Historical average for similar refactors in this codebase: 2-4 hours
- Source: engram_mem_search("refactor duration")
- Confidence: medium (3 data points)

INSIDE VIEW:
- Factor 1: "Touches 3 more files than typical" → Adjustment: +1 hour → Rationale: "Cross-file dependencies require additional testing"
- Factor 2: "Uses unfamiliar library (Effect TS)" → Adjustment: +1.5 hours → Rationale: "Team has limited experience with this library"
- Factor 3: "Has comprehensive test coverage" → Adjustment: -0.5 hours → Rationale: "Tests will catch issues quickly"

FINAL ESTIMATE: 4-6.5 hours
RANGE: 4 — 6.5 hours
CONFIDENCE: low (unfamiliar library increases variance)
```
