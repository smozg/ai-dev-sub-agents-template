---
name: deploy-pipeline
description: |
  19-step Deploy + GTM pipeline: DEV → STAGE → PROD → Go To Market → Close.
  Ensures no deploy happens without automated E2E, human smoke test, and marketing launch.

  Use when: "deploy", "stage", "prod", "scp", "/deploy"
---

# Deploy + GTM Pipeline — 19 Steps

## Overview

Every epic deploys through this pipeline. No batching — each story deploys immediately after completion. GTM is NOT optional — it's part of the deploy.

**Rule:** Sub-agents verify BEFORE user smoke-tests. Never ask user to test until sub-agent confirms.

**Conv_log:** Epic tag = from `docs/TODO.md` header (e.g. `G2-E7`), NOT from previous session memory. Check TODO.md before first log entry.

**Findings default:** Fix ALL findings (not just P1/High). Only ask "which to fix?" if > 15 findings.

---

## Step 0: Initialize Pipeline State (BLOCKING — nothing else runs without this)

Before starting any deploy, the orchestrator MUST:

1. Copy `references/pipeline-state-template.yml` → `work/{feature}/pipeline-state.yml`
2. Fill `epic`, `feature`, `started` fields
3. Create `work/{feature}/findings.yml` from `references/findings-template.yml` (if not exists)

**BLOCKING GATE:** If `work/{feature}/pipeline-state.yml` does NOT exist → STOP.
No step can proceed. "Pipeline state not initialized. Run Step 0 first."

**ALSO REQUIRED:** `work/{feature}/findings.yml` must exist (initialize empty if no findings yet). Even if review steps don't run (infra story), findings.yml documents the absence. Missing = retro can't verify finding coverage.

Example from project (G2-E8): pipeline-state.yml was not maintained from step 0 → steps 5,6,10-12 skipped → 7 bugs reached PROD → 2 user corrections. Pipeline state = the only protection against skipped steps.

**Pipeline state is the source of truth.** After every step:
- Update `status: done` and `gate_passed: true/false`
- Update `current_step` and `current_phase`
- Update `updated` timestamp
- Update `findings_count` / `findings_fixed` after sub-agent steps

**Pre-flight gates:** Before starting each step, READ pipeline-state.yml and check conditions in `references/preflight-gates.md`.
If ANY condition fails → STOP and report. **NEVER skip a step.**

**COMPOUND GATES (Steps 7 and 13):**
- Step 7 REQUIRES Steps 4+5+6 ALL `gate_passed: true` — user smoke test impossible without E2E + PM + UX
- Step 13 REQUIRES Steps 10+11+12 ALL `gate_passed: true` — same pattern for PROD

**After autocompress:** post-compact hook reads pipeline-state.yml and outputs exact step to resume.

---

## Phase 1: DEV (Step 1)

### Step 1: Code + Tests

```bash
cd $PROJECT_DIR

# Run full test suite
uv run python tests/setup_test_matrix.py --db data/test_db.db
uv run python tests/cli_test_runner.py --users all --suite all --db data/test_db.db

# Import check
uv run python -c "import <project>; print('OK')"
```

**Gate:** ALL tests must pass. No exceptions.

Log: `cd $PROJECT_DIR && uv run python scripts/conv_log.py {EPIC} claude phase "DEV tests passed, deploying to STAGE"`

---

## Phase 2: STAGE (Steps 2-7)

### Step 2: Deploy to STAGE

**Orchestrator creates** `work/{feature}/deploy-request.md`:
```yaml
action: deploy
target: stage
files:
  - core/flows.py
  - core/texts.py
  # ... all changed files from git diff --name-only
restart: [<stage-service>, <stage-service-2>]
```

**Dispatch Deploy Agent:**
```
Agent(
  prompt="Deploy Agent. Read .claude/agents/deploy-agent.md for instructions.
  Deploy request: work/{feature}/deploy-request.md.
  Execute: import deps check → scp → pycache → restart → verify.",
  model="sonnet"
)
```

**Gate:** Deploy Agent reports SUCCESS. Update pipeline-state: step 2 done, gate_passed: true.

### Step 3: Start STAGE Parser (if applicable)

```bash
ssh root@<RUN_SERVER> "systemctl start <project>-worker-staging"
```

### Step 4: E2E Test — Tester (Sonnet)

```
Agent(
  prompt="You are a tester. Read your instructions from docs/sub-agents/tst-stage.md.
  Brief: {path to brief or tech-spec}. E2E plan: {steps from tech-spec}.
  SSH to <RUN_SERVER>, run project CLI, test the feature, check DB.
  Report pass/fail for each check.
  Read the 'Lessons Learned' section in your instructions and add 'Applied lessons' table to report.",
  model="sonnet"
)
```

**Gate:** All E2E checks must PASS. If any FAIL → go to [Failure Path](references/failure-paths.md).

**Findings:** Orchestrator records ALL tester findings in `work/{feature}/findings.yml`. Fix all before proceeding.

### Step 5: PM Review (Opus)

```
Agent(
  prompt="You are a PM reviewer. Read your instructions from docs/sub-agents/pm-stage.md.
  Brief: {path to brief}. Walk the CJM via CLI on STAGE.
  Check texts, buttons, tone, navigation. Write findings.
  Read the 'Lessons Learned' section in your instructions and add 'Applied lessons' table to report.",
  model="opus"
)
```

**Findings:** Orchestrator records ALL PM findings in `work/{feature}/findings.yml`.

### Step 6: UX Review (Opus)

```
Agent(
  prompt="You are a UX reviewer. Read docs/sub-agents/ux-review.md.
  Brief: {path to brief}. Walk the CJM via CLI on STAGE.
  Check tone, button labels, icons, screen flow, consistency.
  Write findings: UX-N: P0/P1/P2 — description — recommendation.
  Read the 'Lessons Learned' section in your instructions and add 'Applied lessons' table to report.",
  model="opus"
)
```

UX findings: P0 = blocker, P1 = fix before user test, P2 = backlog.

**Findings:** Orchestrator records ALL UX findings in `work/{feature}/findings.yml`.

### Step 7: User Smoke Test

**Pre-flight:** Steps 4+5+6 ALL `gate_passed: true`. Check `findings.yml` — NO `status: open` findings from steps 4-6.

```bash
/root/.claude/notify_user.sh "Claude is waiting: smoke test STAGE"
```

Wait for user confirmation.

---

## Phase 3: PROD (Steps 8-13)

**Only after user says "ok" on STAGE.**

### Step 8: Deploy to PROD

```yaml
action: deploy
target: prod
files:
  - core/flows.py
  - core/texts.py
  # ... same files as Step 2
restart: [<prod-service>, <prod-service-2>]
```

**Dispatch Deploy Agent** (same agent, different target).

**Gate:** Deploy Agent reports SUCCESS.

### Step 9: Start PROD Parser (if applicable)

```bash
ssh root@<RUN_SERVER> "systemctl start <project>-worker"
```

### Step 10: E2E Test — Tester (Sonnet)

Same as Step 4 but with `tst-prod.md` and PROD paths.

### Step 11: PM Review (Opus)

Same as Step 5 but with `pm-prod.md`.

### Step 12: UX Review (Opus)

Same as Step 6 but on PROD environment.

### Step 13: User Smoke Test

**Pre-flight:** Steps 10+11+12 ALL `gate_passed: true`. Check `findings.yml` — NO `status: open` findings from steps 10-12.

```bash
/root/.claude/notify_user.sh "Claude is waiting: smoke test PROD"
```

---

## Phase 4: Go To Market (Steps 14-18)

**Only after user confirms PROD smoke test.**

### Step 14: Marketing Materials (Opus)

```
Agent(
  prompt="You are a marketing strategist. Read your instructions from docs/sub-agents/marketing.md.
  Epic: {epic}. Brief: {path to user-spec or brief}.
  Create positioning, sales scripts, and launch materials.
  Output to docs/marketing/{epic}.md",
  model="opus"
)
```

### Step 15: Website Update + Site Tester

**Dispatch `webdevops` agent** — do NOT edit site inline.

```
Agent(
  prompt="Update site for {feature}. Brief: docs/marketing/{epic}.md.
  Edit source files per brief.
  Workflow: edit → commit → npm build → scp dist to Run → cache-bust verify via WebFetch on live URL.
  tst-site 8/8 checklist mandatory (includes Visual rendering + Mobile viewport). Report all anchors verified on LIVE URL, not local bundle.",
  subagent_type="webdevops",
  model="sonnet"
)
```

### Step 16: Broadcast — 3-Gate Review

Draft broadcast message → save to `docs/marketing/broadcast_{feature}.txt`

Three gates (mandatory, sequential):

| Gate | Agent | Model | What It Checks |
|------|-------|-------|----------------|
| **1. PM** | `docs/sub-agents/pm-review.md` | **Opus** | Spelling, voice, clarity, all code branches, CTA, tone |
| **2. UX** | `docs/sub-agents/ux-review.md` | **Opus** | Style guide, emoji density, tone, line length |
| **3. Tester** | `docs/sub-agents/tst-stage.md` | Sonnet | Deploy broadcast handler on STAGE, buttons work, callbacks fire |

Fix issues between gates. If a gate fails — fix and re-run that gate.

### Step 17: Broadcast — 3-Step Send

See [broadcast-guide.md](references/broadcast-guide.md) for flags and personalization.

**Step 17a: STAGE preview** → wait for explicit approval
**Step 17b: PROD preview** → wait for explicit approval
**Step 17c: Full send**

Example from project (G2-E5): broadcast sent on PROD without STAGE preview → user very upset. Rule: 3 steps without shortcuts.

### Step 18: Talking Head + Distribution

**18a: Talking Head Script (Opus)** — 30-second conversational script for video:
- 3-4 sentences, as if telling a friend
- Structure: problem → what we did → result/benefit
- No "Hey friends!" — get to the point

**18b: Distribution (User's Responsibility)** — remind user to:
- Send to trainees/clients
- Send script to sales team
- Record video based on script

---

## Phase 5: Close (Step 19)

### Step 19: Scrum Master Review

Load the `scrum-master` skill for iterative review (up to 3 rounds).

**After review:**
1. Update `docs/TODO.md` — mark epic as COMPLETE
2. Update `docs/BACKLOG.md` — check off completed items
3. Archive `work/{feature}/` → `work/completed/`
4. Propose next step

---

## After deploy — auto-pipeline (DO NOT STOP)

After successful deploy and user smoke test:
1. **Continue to GTM** — steps 14-18. Never stop and wait silently after step 13.
2. **Notify user** — notify when waiting for action
3. **Log phase** — conv_log every transition

---

## Rules

- **NEVER `systemctl restart` on Code server** — only on Run
- **NEVER skip sub-agent verification** — even for hotfixes
- **NEVER partial deploy** — all changed files in one batch (check import deps!)
- **Deploy to BOTH paths on PROD** if project uses hardlinks
- **One story = one deploy cycle** — no batching
- **Sub-agent before user** — don't ask user to test until sub-agent passes
- **SSH timeout ≠ success** — always verify with grep on server
- **3 broadcast gates mandatory** — PM → UX → Tester, no shortcuts
- **3-step broadcast send** — STAGE preview → PROD preview → full send
- **Notify user when waiting** — always

## Hotfix bundle rules

During hotfix loop (after PROD reviews found findings):
- **Max bundle size:** ≤ 5 fixes OR ≤ 3 files. More → split into 2+ deploy cycles.
- **Re-review between bundles** — otherwise new bugs only discovered in next iteration.
- **Log in hotfix_loop section of pipeline-state.yml** — which iter, which findings closed/added.

## Lessons enforcement in Agent() dispatches

In each sub-agent dispatch prompt, **include**:

> Read the "Lessons Learned" section in your instructions (`docs/sub-agents/<name>.md`).
> Add "Applied lessons" section to your final report as a table:
> | Lesson | Where checked | Result |
> If a lesson is relevant and NOT applied — this is an automatic FAIL of the report.

Without this, lessons remain read-only — agent reads but doesn't enforce. Example from project (G2-E9): G2-E8 lessons on line length and tone style didn't apply in G2-E9 → 5 findings on PROD.
