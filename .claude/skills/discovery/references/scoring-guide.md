# Interview Scoring Guide

## Score Scale

| Score | Meaning | Example |
|-------|---------|---------|
| **0%** | Not mentioned at all | Field not discussed |
| **20-40%** | Vague, unclear | "Something with reviews" |
| **50-70%** | Brief but directional | "Show rating target to users" |
| **80-95%** | Detailed, actionable | "Show how many 5★ reviews needed to reach 4.5, with monthly plan" |
| **100%** | Complete, no gaps | All edge cases covered, metric defined, no ambiguity |

## Scoring Rules

### After initial description
Score ALL fields based on what user mentioned:
- Directly addressed with detail → 80-95%
- Mentioned briefly → 50-70%
- Implied but not stated → 20-40%
- Not mentioned → 0%

### After each answer
- 1-sentence answer → +20-30% (cap at 70%)
- 3+ sentence detailed answer → score 80-95%
- "I don't know" / "you decide" → score 50% and note as conscious gap
- Contradicts earlier answer → reset to 40%, ask for clarification

## Threshold: 85%

A field reaches 85% when:
1. `value` is filled with concrete, actionable content
2. `gaps` is empty or contains only conscious limitations ("not applicable because...")
3. No TBD, no placeholders, no "will decide later"

## When to Move to Next Cycle

**Both conditions must be true:**
1. All `required: true` fields in current cycle scope ≥ 85%
2. Structural check: every required field has non-empty `value` and resolved `gaps`

If stuck after 3 batches of questions in one cycle → summarize what we know, ask user "enough to proceed?" If yes → mark remaining gaps as conscious limitations and move on.

## Question Batching

### Good: 3-4 questions per batch, different gaps
```
Three questions:
1. What specific pain does the customer have? (pain)
2. How will we know the feature works — what metric? (success_metric)
3. What happens if we don't build this — what does the customer lose? (what_if_not)
```

### Bad: one question at a time
```
What pain are we solving?
[wait]
And the metric?
[wait]
And what if we don't build it?
```

### Bad: dump all 12 questions at once
Too overwhelming. 3-4 is the sweet spot.

## Proposing Solutions

Don't just ask — **propose and validate**:

### Good:
```
Based on the journey, Feature X logically comes after Feature Y. 
Pain: user sees the goal but doesn't know HOW to achieve it. 
Success metric: user tries the feature at least once in the first week.
What do you think? Or different focus?
```

### Bad:
```
What pain does the customer have?
```

## Project-Specific Context for Questions

When asking about features, reference existing patterns:

| Field | Ask in context of... |
|-------|---------------------|
| `pain` | "What pain do current features NOT solve?" |
| `cjm_step` | "After what level/command does this appear?" |
| `affected_screens` | "New command or extension of existing feature?" |
| `edge_cases` | "What if user has 0 data? No competitor? Already at max level?" |
| `data_source` | "Does parser already collect this data or do we need new fields?" |
| `risks` | "What external dependency could change? How do we handle missing data?" |
