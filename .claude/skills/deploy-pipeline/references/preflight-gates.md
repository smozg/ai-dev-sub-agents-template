# Pre-flight Gates Reference

Before starting ANY step, the orchestrator MUST:

1. Read `work/{feature}/pipeline-state.yml`
2. Check the pre-flight conditions for the target step (table below)
3. If ANY condition FAILS → STOP and report what's missing
4. If all PASS → update `status: running` and proceed

## Conditions Table

| Step | Pre-flight Conditions |
|------|-----------------------|
| 1 | tech-spec exists at `work/{feature}/tech-spec.md` |
| 2 | Step 1 `gate_passed: true` |
| 3 | Step 2 `gate_passed: true` |
| 4 | Step 3 `gate_passed: true` |
| 5 | Step 4 `gate_passed: true` AND `findings_fixed >= findings_count` |
| 6 | Step 5 `gate_passed: true` |
| **7** | **Steps 4 + 5 + 6 ALL `gate_passed: true`** |
| 8 | Step 7 `gate_passed: true` AND `user_confirmed` is set |
| 9 | Step 8 `gate_passed: true` |
| 10 | Step 9 `gate_passed: true` |
| 11 | Step 10 `gate_passed: true` AND `findings_fixed >= findings_count` |
| 12 | Step 11 `gate_passed: true` |
| **13** | **Steps 10 + 11 + 12 ALL `gate_passed: true`** |
| 14 | Step 13 `gate_passed: true` AND `user_confirmed` is set |
| 15 | Step 14 `gate_passed: true` |
| 16 | Step 15 `gate_passed: true` |
| 17 | Step 16 `gate_passed: true` (all 3 broadcast gates passed) |
| 18 | Step 17 `gate_passed: true` |
| 19 | Step 18 `gate_passed: true` |

## Critical Gates

Steps **7** and **13** are the compound gates:
- They require ALL THREE preceding sub-agent steps (Tester + PM + UX) to pass
- This prevents skipping PM/UX reviews
- If any of the three is still `pending` or `failed`, the gate BLOCKS

## How to Check (orchestrator reads pipeline-state.yml)

```
1. Read work/{feature}/pipeline-state.yml
2. Find the target step number
3. Look up conditions in table above
4. For each condition: check the referenced step's gate_passed field
5. If ANY condition is false/null → STOP, report: "Pre-flight FAILED for Step N: Step M not passed"
6. If ALL pass → update target step status to "running", proceed
```

## After Step Completion

```
1. Update step status: done
2. Update gate_passed: true (or false if failed)
3. Update findings_count / findings_fixed if applicable
4. Update current_step and current_phase in header
5. Update updated timestamp
```
