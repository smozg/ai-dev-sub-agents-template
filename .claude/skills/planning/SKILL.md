---
name: planning
description: |
  Transforms an approved user-spec into a validated tech-spec (implementation plan).
  Runs skeptic and completeness validators, then architect review.
  Creates TODO.md with implementation steps.

  Use when: "create plan", "tech spec", "implementation plan", "planning",
  "план", "как будем делать", "напиши план", "архитектор",
  "/new-tech-spec", "plan", "decompose"
---

# Planning — User-Spec to Tech-Spec with Validation

## Overview

Planning transforms an approved user-spec (WHAT) into a tech-spec (HOW). Two automated validators ensure the plan is grounded in reality and covers all requirements.

## Phase 0: Load Context

1. Read the approved user-spec from `work/{feature}/user-spec.md`
2. Verify status is `approved` — if not, redirect to discovery skill
3. Read relevant project context:
   - `<PROJECT>-architecture.md` — code structure, patterns
   - `<PROJECT>-cjm.md` — where this fits in the journey (section relevant to the level)
   - Domain-specific source files for existing patterns
4. **Conditional (added 2026-06-01):** if tech-spec will reference docker services/stacks, application models, SSH/sudoers/nginx/cron, PHP-FPM, Apache, or host-level services — read `.claude/memory/server-factbook.md` FIRST and quote exact values from it. Do not write tech-spec claims from memory.
5. **For infra-epics (deploy/discovery/ops): `ss -tlnp` first.** Before drafting factbook scope or referencing services, enumerate ALL listening services on the target host:
   ```bash
   ssh <server> "ss -tlnp"
   ssh <server> "systemctl list-units --type=service --state=running"
   ```
   Tech-spec scope = derived from this enumeration, not "what I remember exists". This prevents hallucination-class findings (a real production system might exist on the host but be invisible to memory).

## Phase 0.5: Epic split protocol (added from real-project retro 2026-06-01)

When user-spec is the result of **splitting a previously-planned monolith** (e.g. one big epic was broken into smaller scoped sub-epics), there is a hazard of **losing findings** from the monolith planning phase. Each split-off finding may apply to a different sub-epic, but if not explicitly classified, it disappears — and re-emerges as a hotfix later.

Mandatory steps before drafting tech-spec for a split-off epic:

1. **Locate monolith findings.** The old monolith's `archive_v1_monolith/findings.yml` or `tech-spec`'s changelog/Risks.
2. **Per-finding classification** with explicit decision:
   - `addressed_in_<this_epic>` — covered by this epic's tech-spec (link section)
   - `addressed_in_<other_epic>` — explicitly redirected to another split
   - `deferred_to_<future_epic>` — added to BACKLOG
   - `accepted_risk` — won't fix; document why
3. **Write findings audit table** in tech-spec preamble:
   ```markdown
   ## Monolith findings carry-over
   | Old ID | Summary | Decision | Where |
   |---|---|---|---|
   | F007 | Token scope `workflow` needed | addressed_in_this_epic | Decision 3 |
   ```
4. **Architect mandatory check** (per architect.md): verify monolith findings list is non-empty for split epics, and no item lacks classification.

**Real example:** In a project's monolith r1, architect flagged "workflow scope insufficient". At split, this was reclassified as "next-epic manual user step". But mirror push **requires** workflow scope for history-with-workflow-files, not just for current edits. Same root cause reappeared mid-execution → token rotation hotfix. The lesson existed; the carry-over protocol didn't.

## Phase 1: Codebase Research

Before writing the plan, understand the existing code:

1. **Find similar features:** Grep for patterns matching the new feature
   - If adding a new command: look at how `/fresh`, `/target`, `/cardcheck` are implemented
   - If adding parser fields: check `src/obrep/parser.py` for existing extractors
   - If changing onboarding: check `src/obrep/core/onboarding.py`

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
7. **Backfill / Data migration strategy** (if adding column or changing data) — explicit SQL UPDATE statements, dry-run mode, audit accuracy default

### Backfill strategy guidelines (G2-E10-S20 lesson)

If tech-spec adds a column or transforms existing data:

- **Audit accuracy as default** — prefer actual data values (e.g. `DEMO`/`PROD`/`SYSTEM_TEST`) over opaque markers (`LEGACY_UNKNOWN`). Opaque markers lose audit info and require iteration.
- **Justify opaque markers** if used — explain why actual values can't be derived from existing data.
- **Idempotent script** — re-runnable, matches both new and previously-backfilled states.
- **Dry-run as default** — `--apply` flag required to write.
- **PRAGMA prerequisite check** — abort if migration column doesn't exist on target DB.
- **Document rollback SQL** — in script docstring, reversible to pre-backfill state.

### Writing Rules

- Reference **specific files and line numbers** when possible
- Reference **specific functions** that will be modified or called
- Use **existing patterns** — don't invent new architecture
- Every claim about the codebase must be **verifiable** (skeptic will check)
- Every requirement from user-spec must be **traceable** (completeness will check)

## Phase 3: Validation Round (up to 3 iterations)

Run two validators **in parallel**:

### Validator 1: Skeptic Agent

Launch via Agent tool:
```
Agent(
  prompt="Read the tech-spec at work/{feature}/tech-spec.md. For every file path, function name, 
  API endpoint, DB table, and parser field mentioned — verify it actually exists in the codebase 
  at /root/Automations/Telegram_Bot/obrep_bot/. Use Grep and Glob to check. Report findings as 
  JSON: {status: 'approved'|'changes_required', findings: [{severity, claim, reality, fix}]}",
  subagent_type="general-purpose",
  model="sonnet"
)
```

### Validator 2: Completeness Agent

Launch via Agent tool:
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

## Phase 4: Architect Review (dispatched agent — Opus, architect-agent-and-parallel-planning)

⛔ **Replaces orchestrator checklist with dedicated architect agent on Opus.**

```
Agent(
  prompt="Validate tech-spec at work/{feature}/tech-spec.md per .claude/agents/architect.md.
  Checks: 5 code rules + Parallel Execution Plan section (grep file overlap verification) +
  dependency graph + architecture pattern compliance + risk identification.
  Read project CLAUDE.md for code rules.
  Read relevant memory files for architecture patterns.
  Report verdict APPROVED for development | CHANGES_REQUIRED with action items.",
  subagent_type="architect",
  model="opus"
)
```

**Processing verdict:**
- **APPROVED** → proceed to Phase 5 (TODO update)
- **CHANGES_REQUIRED** → orchestrator addresses each finding:
  - 5 code rule FAIL → must fix in tech-spec
  - Parallel Execution Plan missing/incorrect → add or correct
  - Architecture pattern deviation → justify or revise
  - Re-run architect after fixes (max 3 rounds total)

**Why agent + Opus:**
- Architecture decisions benefit from Opus deliberation (vs Sonnet)
- Independent context catches issues orchestrator might rationalize
- Validates Parallel Execution Plan via grep — not "trust me, it's parallel"
- trust-event-recovery lesson: bypass routes were missed because no one verified hook coverage across all parallel pathways

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
4. Log: `cd /root/Automations/Telegram_Bot/obrep_bot && uv run python scripts/conv_log.py {EPIC} claude decision "tech-spec approved"`

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

## Phase 1.4: Payment / Webhook / LLM checklist (G2-E9 lesson)

Если фича касается **оплат**, **webhook'ов**, или **LLM** — обязательно ответить в tech-spec на эти вопросы.
Иначе планирование пропустит фундаментальный недостаток (см. F31→F33→F36 эволюцию, F35-F36 LLM
formatting bugs).

### Payment / Webhook
1. **Idempotency:** какой ключ предотвращает дубль? PRIMARY KEY на order_id в payment ledger (как `tbank_orders`)? Time-window dedup? **Time-window не подходит** для T-Bank-style retransmits (могут идти часами).
2. **Webhook retransmits:** что произойдёт если T-Bank пришлёт CONFIRMED 5 раз за час? Логика должна быть idempotent на уровне БД (UNIQUE constraint), не на уровне приложения.
3. **Service map:** какой сервис (obrep-qr / obrep-billing) обрабатывает webhook? Должен ли он рестартовать при deploy этой фичи? (Deploy-agent service map в `.claude/agents/deploy-agent.md`.)
4. **Audit journal:** где хранится trail "user X пытался купить Y, статус Z, время T"? Если использовать только systemd journal — данные ротируются. Нужна persistent table (e.g. `payment_events`).
5. **Race conditions:** что если user нажимает кнопку 2 раза за 0.5 сек? (UI lock через `edit_message`, in-memory lock per user_id, DB-level INSERT с UNIQUE).

### LLM (Claude / GPT / etc)
1. **Output format:** prompt должен запретить `#` headers, `|` tables, `*-/list markers`. Telegram их не рендерит. См. `docs/style-guide.md` → "LLM prompt output rules".
2. **Tone enforcement:** обращение "ты" должно быть в самом prompt'е, иначе модель пишет formal "Вы".
3. **Length:** `max_tokens` ≥ expected output. Если limit=1200 tokens — реально влезет ~2500 ch RU.
4. **Cost tracking:** для Opus pricing `$15 in + $75 out per M tokens`, не `$4/M Sonnet`. Хранить input/output отдельно.

## Rules

- **NEVER skip validation.** Even for "simple" features — skeptic catches hallucinations
- **Fix, don't dismiss.** If skeptic says a file doesn't exist, verify and fix the plan
- **Trace everything.** Every tech-spec item should trace to a user-spec requirement
- **Show the plan.** User must approve before any code is written
- **Log decisions.** Every non-obvious choice goes into decisions.md
