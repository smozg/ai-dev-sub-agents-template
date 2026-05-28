---
name: planning
description: |
  Transforms an approved user-spec into a validated tech-spec (implementation plan).
  Runs skeptic and completeness validators, then architect review.
  Creates TODO.md with implementation steps.

  Use when: "create plan", "tech spec", "implementation plan", "planning",
  "/new-tech-spec", "plan", "decompose"
---

# Planning — User-Spec to Tech-Spec with Validation

## Overview

Planning transforms an approved user-spec (WHAT) into a tech-spec (HOW). Two automated validators ensure the plan is grounded in reality and covers all requirements.

## Phase 0: Load Context

1. Read the approved user-spec from `work/{feature}/user-spec.md`
2. Verify status is `approved` — if not, redirect to discovery skill
3. Read relevant project context:
   - Architecture file — code structure, patterns
   - CJM file — where this fits in the journey
   - `src/<project>/core/flows.py` — existing build_*_actions() patterns
   - `src/<project>/core/texts.py` — existing text patterns

## Phase 1: Codebase Research

Before writing the plan, understand the existing code:

1. **Find similar features:** Grep for patterns matching the new feature
   - If adding a new command: look at how existing commands are implemented
   - If adding parser fields: check `parser.py` for existing extractors
   - If changing onboarding: check `onboarding.py`

2. **Identify files to modify:** List every file that will be touched
   - core/ files (business logic)
   - tg/handlers.py, max/handlers.py, cli/callbacks.py (all three!)
   - db.py (if schema changes)
   - parser.py (if new data fields)
   - tests/ (what tests exist for similar features)

3. **Check dependencies:** Do required parser fields already exist? Do DB tables have the columns needed?

## Phase 2: Write Tech-Spec

Create `work/{feature}/tech-spec.md` using the template from [plan-template.md](references/plan-template.md).

The tech-spec must include:

1. **Context** — what user-spec says, why we're doing this
2. **Files to modify** — every file, with what changes
3. **Implementation steps** — ordered, with dependencies
4. **Architecture decisions** — key choices and why
5. **Testing strategy** — unit tests, E2E plan, test data
6. **Deploy plan** — which stories, what order
7. **Backfill / Data migration strategy** (if adding column or changing data)

### Backfill strategy guidelines

If tech-spec adds a column or transforms existing data:

- **Audit accuracy as default** — prefer actual data values over opaque markers (`LEGACY_UNKNOWN`). Opaque markers lose audit info and require iteration.
- **Justify opaque markers** if used — explain why actual values can't be derived from existing data.
- **Idempotent script** — re-runnable, matches both new and previously-backfilled states.
- **Dry-run as default** — `--apply` flag required to write.
- **PRAGMA prerequisite check** — abort if migration column doesn't exist on target DB.
- **Document rollback SQL** — in script docstring, reversible to pre-backfill state.

Example from project (G2-E10-S20): initial backfill used opaque `LEGACY_UNKNOWN` marker — user corrected to audit-accurate values. Re-apply on STAGE cost significant time. Lesson: default to actual values.

### Writing Rules

- Reference **specific files and line numbers** when possible
- Reference **specific functions** that will be modified or called
- Use **existing patterns** — don't invent new architecture
- Every claim about the codebase must be **verifiable** (skeptic will check)
- Every requirement from user-spec must be **traceable** (completeness will check)

## Phase 1.4: Payment / Webhook / LLM checklist

If feature touches **payments**, **webhooks**, or **LLM** — answer these questions in tech-spec.
Otherwise planning skips a fundamental gap.

### Payment / Webhook
1. **Idempotency:** what key prevents duplicates? PRIMARY KEY on order_id in payment ledger? Time-window dedup? **Time-window is NOT suitable** for payment gateway retransmits (can come hours later).
2. **Webhook retransmits:** what happens if gateway sends CONFIRMED 5 times in an hour? Logic must be idempotent at DB level (UNIQUE constraint), not application level.
3. **Service map:** which service processes the webhook? Should it restart during this deploy? (Deploy-agent service map in `.claude/agents/deploy-agent.md`.)
4. **Audit journal:** where is the trail "user X tried to buy Y, status Z, time T" stored? If only system journal — data rotates. Need persistent table (e.g. `payment_events`).
5. **Race conditions:** what if user presses button 2 times in 0.5 sec? (UI lock via edit_message, in-memory lock per user_id, DB-level INSERT with UNIQUE)

### LLM (Claude / GPT / etc)
1. **Output format:** prompt must forbid `#` headers, `|` tables, `*-/list markers`. Telegram doesn't render these. See style-guide.
2. **Tone enforcement:** "you" addressing must be in the prompt itself, otherwise model writes formal.
3. **Length:** `max_tokens` ≥ expected output.
4. **Cost tracking:** track input/output tokens separately. Opus pricing ≠ Sonnet pricing.

## Phase 3: Validation Round (up to 3 iterations)

Run two validators **in parallel**:

### Validator 1: Skeptic Agent

```
Agent(
  prompt="Read the tech-spec at work/{feature}/tech-spec.md. For every file path, function name, 
  API endpoint, DB table, and parser field mentioned — verify it actually exists in the codebase 
  at $PROJECT_DIR. Use Grep and Glob to check. Report findings as 
  JSON: {status: 'approved'|'changes_required', findings: [{severity, claim, reality, fix}]}",
  subagent_type="general-purpose",
  model="sonnet"
)
```

### Validator 2: Completeness Agent

```
Agent(
  prompt="Read user-spec at work/{feature}/user-spec.md and tech-spec at work/{feature}/tech-spec.md.
  Check bidirectional traceability:
  1. Every requirement in user-spec → is it covered in tech-spec? 
  2. Everything in tech-spec → does it trace back to a user-spec requirement (no scope creep)?
  Report: {uncovered_requirements: [...], extra_in_techspec: [...], status: 'approved'|'changes_required'}",
  subagent_type="general-purpose",
  model="sonnet"
)
```

### Processing findings

For each finding:
- **Critical** (missing file, wrong function name) → fix immediately
- **Major** (uncovered requirement) → fix or discuss with user
- **Minor** (style, optimization suggestion) → note in decisions.md, skip

After fixes: re-run the validator that found the issue (max 3 rounds total).

## Phase 4: Architect Review

Run the architect checklist (from the code rules):

- [ ] Business logic in core/, not in handlers?
- [ ] No duplication between TG and MAX and CLI?
- [ ] No TG/MAX/CLI imports in core/?
- [ ] `get_db_path()` instead of `DB_PATH`?
- [ ] All new texts in `core/texts.py`?
- [ ] Level progression formula respected (if applicable)?
- [ ] Rule of 5: no more than 5 items in any list?

If any check fails → update tech-spec.

## Phase 5: Create TODO

Create/update `docs/TODO.md` with implementation steps from tech-spec:

```markdown
# TODO: {feature_name} ({epic})

Plan: `work/{feature}/tech-spec.md`
Brief: `work/{feature}/user-spec.md`

## Steps
- [ ] Step 1: ...
- [ ] Step 2: ...
...
```

## Phase 6: User Approval

1. Show tech-spec summary to user (key decisions, files to modify, risks)
2. Wait for explicit approval
3. Set tech-spec status to `approved`
4. Log: `cd $PROJECT_DIR && uv run python scripts/conv_log.py {EPIC} claude decision "tech-spec approved"`

## Phase 7: Create Decisions Log

Create `work/{feature}/decisions.md`:

```markdown
# Decisions: {feature_name}

## Planning Decisions
- Decision 1: {what} — because {why}
- ...

## Validation Results
- Skeptic: {N} claims checked, {M} issues found and fixed
- Completeness: {N} requirements traced, {M} gaps found and resolved
```

## Rules

- **NEVER skip validation.** Even for "simple" features — skeptic catches hallucinations
- **Fix, don't dismiss.** If skeptic says a file doesn't exist, verify and fix the plan
- **Trace everything.** Every tech-spec item should trace to a user-spec requirement
- **Show the plan.** User must approve before any code is written
- **Log decisions.** Every non-obvious choice goes into decisions.md
