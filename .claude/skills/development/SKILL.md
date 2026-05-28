---
name: development
description: |
  Implements features using TDD for core/ modules. Handles code writing, testing,
  code review, and granular commits. Supports wave-based task execution.

  Use when: "write code", "implement", "develop", "TDD", "code", "fix bug",
  "/write-code", "/do-task"
---

# Development — TDD + Code Review Pipeline

## Overview

Development follows a strict order: **plan → tests → code → review → commit**. For core/ changes, tests are written FIRST (TDD). Code review agent runs after implementation.

## Orchestrator Dispatch (default for story-level work)

For tasks touching **3+ files**, the orchestrator dispatches a worktree agent instead of coding directly:

```
Agent(
  prompt="Developer. Read .claude/skills/development/SKILL.md.
  Implement Story {S} from work/{feature}/tech-spec.md.
  TDD for core/. Run tests. Run code-reviewer agent.
  Commit: 'feat({epic}): {description}'.
  Report: files changed, tests (N/N), review verdict, commit hash.",
  model="sonnet",
  isolation="worktree"
)
```

After agent returns, the orchestrator:
1. Reads the agent's report (files changed, tests, review)
2. Merges worktree changes to main (or asks user to review)
3. Runs full test suite on main
4. Updates `docs/TODO.md`

**When to code directly (no worktree):** hotfix <3 files, single-line fixes, text-only changes.

## Phase 0: Load Context

1. Read `docs/TODO.md` — find current step
2. Read tech-spec from `work/{feature}/tech-spec.md` (if exists)
3. Read relevant code rules from [code-rules.md](references/code-rules.md)
4. If task file exists (`work/{feature}/tasks/{N}.md`) — read it for acceptance criteria

## Phase 1: Plan the Work (HARD GATE — no code without approval)

Before writing ANY code:

1. **Identify what changes:** which files, what functions, what tests
2. **Check existing patterns:** grep for similar implementations in the codebase
3. **If UX/flow decision involved** → discuss flow with user BEFORE writing code
4. **Show plan to user:** "Here's what I plan to change: [files, approach, tests]"
5. **WAIT for explicit "yes" / "do it" / "agreed"** — do NOT proceed on silence or implicit approval
6. **Determine task scope:**
   - Single task → proceed directly
   - Multiple tasks → check `wave` and `depends_on` fields (see Wave Execution below)

> **STOP RULE:** No explicit approval from user = NO Edit/Write calls.
> Violation is the most common complaint (9+ incidents in the reference project).

## Phase 2: TDD — Tests First (for core/ changes)

**When to TDD:** any change to `src/<project>/core/` — always write tests first.
**When to skip TDD:** pure UI/text changes, config changes, documentation.

### TDD Process

1. **Write failing test** in `tests/` that describes the expected behavior
2. **Run tests** — confirm the new test FAILS:
   ```bash
   cd $PROJECT_DIR && uv run python tests/setup_test_matrix.py --db data/test_db.db
   uv run python tests/cli_test_runner.py --users all --suite all --db data/test_db.db
   ```
3. **Write implementation** — make the test PASS
4. **Run tests again** — confirm ALL tests pass (no regression)

## Phase 3: Write Code

Follow the code rules strictly (see [code-rules.md](references/code-rules.md)):

1. Business logic ONLY in `core/`
2. Texts ONLY in `core/texts.py`
3. Every feature in core/ → wrap in ALL frontends (TG, MAX, CLI)
4. No platform imports in `core/`
5. DB operations via `get_db_path()`

### Sync Checklist (when changing UI)

After every change that affects user-facing messages:

- [ ] Text identical across all frontends? (from `core/texts.py`)
- [ ] Buttons identical? (same `callback_data`)
- [ ] New callback added to all handlers?
- [ ] New `build_*_actions()` called from all frontends?
- [ ] **If signature changed in shared function → grep all callers? All updated?** (Signature Drift lesson)

### Phase 3.5: Signature Change Discipline

If implementation changes signature of a shared function (add required arg, change keyword, rename param):

1. **Before commit** — grep all callers:
   ```bash
   func=function_name_here
   grep -rn "${func}(" src/<project>/ --include="*.py" | grep -v "def ${func}"
   ```
2. **Update every caller** — even if outside tech-spec's explicit list
3. **If caller should NOT use new arg** — explicitly document (default value, comment)
4. **Test:** add test that asserts the new field is written for at least one caller per file

**Why this matters:** tech-spec rarely enumerates every caller. Worktree agent implements by list → misses callers outside the list → audit/safety columns get silent empty values.

Example from project (G2-E10-S20): worktree agent added `terminal_id` parameter, updated only callers listed in tech-spec. 6 missed sites in other files — each INSERT wrote row without terminal_id → broken audit trail.

### Progress Tracking

After each logical unit of work:
- Mark step done in `docs/TODO.md`
- Log decision if non-obvious: `cd $PROJECT_DIR && uv run python scripts/conv_log.py {EPIC} claude decision "..."`

## Phase 4: Code Review — Dual reviewer pattern

**Worktree agent does NOT self-review** (author bias). Orchestrator dispatches **2 independent reviewers**:

### Reviewer 1: code-reviewer (Sonnet) — architecture, sync, tests, generic correctness

```
Agent(
  prompt="Independent code review for commit {hash}. You have NO context from the developer.
  Read tech-spec at work/{feature}/tech-spec.md.
  Run: git diff HEAD~1
  Check against code-reviewing skill (all checks including Signature Change check 8).
  Report findings with severity and file paths.",
  subagent_type="code-reviewer",
  model="sonnet"
)
```

### Reviewer 2: domain-specific (Opus) — financial / security / data-integrity

For payment / billing / migration / audit-related diffs — dispatch the specialized `financial-reviewer` agent:

```
Agent(
  prompt="Financial-correctness review for commit {hash}. Money flow focus only.
  Read tech-spec at work/{feature}/tech-spec.md.
  Trace through ALL known user scenarios from agent's scenario list.
  Report verdict: APPROVED for STAGE / CHANGES_REQUIRED.",
  subagent_type="financial-reviewer",
  model="opus"
)
```

### Processing findings

- **Convergence** (both reviewers find same issues) = high confidence
- **Divergence** = investigate why (different definitions of "critical"?)
- Apply fixes BEFORE STAGE deploy
- Re-run reviewer after fixes (max 3 rounds)

### When to use dual review

**Always:** payment code, security/permissions, data migrations, audit/billing changes, schema migrations
**Skip:** pure UI/text changes (single code-reviewer is sufficient)

## Phase 5: Commit

**Granular commits** — after each story, not batch:

```bash
cd $PROJECT_DIR
git add {specific files}
git commit -m "feat({epic}): {what changed}"
```

**Commit per story**, not per feature. If a feature has 5 stories, that's 5 commits.

## Wave Execution (Optional)

When tasks have `wave` and `depends_on` fields in task files:

### Sequential Mode (default)
Execute tasks one by one in wave order.

### Parallel Mode (opt-in)
For tasks within the same wave that are independent:

```
# Example: Wave 1 has independent tasks
Agent(
  prompt="Execute task 1 from work/{feature}/tasks/1.md ...",
  isolation="worktree",
  model="sonnet"
)
Agent(
  prompt="Execute task 2 from work/{feature}/tasks/2.md ...",
  isolation="worktree",
  model="sonnet"
)
```

**When to parallelize:**
- Tasks touch different files
- Tasks are in the same wave (no dependency between them)
- User explicitly requests it

**When NOT to parallelize:**
- Tasks modify the same file
- One task depends on another's output
- Small tasks (faster to do sequentially)

## Quick Mode (/write-code)

For small tasks without full spec:

1. Skip phases 0-1 (no tech-spec needed)
2. Read TODO.md or user's description
3. Proceed directly to Phase 2 (TDD) or Phase 3 (code)
4. Still run code review (Phase 4)
5. Commit (Phase 5)

## Rules

- **No code without approval.** Show plan → wait for explicit "yes" → only then Edit/Write.
- **TDD for core/.** No exceptions. Tests first, code second.
- **New callback/flow = new test.** When adding callback or `build_*_actions()` → add test case.
- **Sync all frontends.** Every feature in all frontends.
- **Mark progress.** Update TODO.md after each step.
- **Review before commit.** Code review agent catches architecture violations.
- **Granular commits.** One story = one commit.
- **PM agent = Opus.** Never use Sonnet for PM review.
- **Conv_log on every phase change.** Log decisions, questions, insights at checkpoints.
- **Notify user when waiting.** Always notify before waiting for user action.
