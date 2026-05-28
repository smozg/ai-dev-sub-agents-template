---
description: "Start the 19-step deploy pipeline: DEV → STAGE → PROD → GTM → Close. Tracks state in pipeline-state.yml."
argument-hint: "[story ID, e.g. G2-E8]"
---

# Deploy Pipeline

Load the `deploy-pipeline` skill and execute the 19-step pipeline for: $ARGUMENTS

## Orchestrator Checklist

**Step 0 — Initialize (before anything else):**
1. Read `docs/TODO.md` for epic tag and current state
2. Copy `references/pipeline-state-template.yml` → `work/{feature}/pipeline-state.yml`
3. Fill `epic`, `feature`, `started` fields
4. Create `work/{feature}/findings.yml` from template (if not exists)
5. Log: `conv_log {EPIC} claude phase "deploy started, pipeline-state initialized"`

**Every step — Orchestrator loop:**
1. Read `work/{feature}/pipeline-state.yml` — know current step
2. Check pre-flight gates (`references/preflight-gates.md`) — ALL conditions must pass
3. Update step `status: running`
4. Execute step (dispatch sub-agent / deploy agent / wait for user)
5. Record findings in `work/{feature}/findings.yml` (after sub-agent steps)
6. Update step `status: done`, `gate_passed`, `findings_count/fixed`
7. Update `current_step`, `current_phase`, `updated`
8. Proceed to next step

**Rules:**
- Do NOT skip sub-agent verification (steps 4-6, 10-12)
- Do NOT ask user to smoke-test until sub-agents pass
- Steps 7 and 13 require ALL THREE preceding steps (Tester + PM + UX) to pass
- After step 19: load `scrum-master` skill (3 rounds via agent)
