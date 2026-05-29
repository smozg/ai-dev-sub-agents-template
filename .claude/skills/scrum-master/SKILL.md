---
name: scrum-master
description: |
  Dispatches the Scrum Master Agent (Opus) for iterative review and retrospective.
  3 rounds: raw analysis → verify fixes → final clean check.
  Mandatory after every epic (Step 19 of deploy pipeline).

  Use when: "scrum master", "review docs", "done", "close task",
  "retro", "check documentation", "/done"
---

# Scrum Master — Agent Dispatch (3 Rounds)

## Overview

After completing a feature/epic, the scrum master agent reviews all documentation, process, and agent quality. **The orchestrator dispatches the agent — does NOT do the review itself.**

## Rule #1: NEVER Fix Without User Approval

```
Agent finds → orchestrator shows user → user decides → only then fix
```

## Process

### Round 1: Raw Analysis

Dispatch the scrum master agent:

```
Agent(
  prompt="Scrum Master. Read .claude/agents/scrum-master-agent.md for full instructions.
  Epic: {EPIC}. Feature: {FEATURE}. Round: 1.
  
  Read these sources:
  1. Conv_log: cd $PROJECT_DIR && run query for epic {EPIC}
  2. work/{feature}/pipeline-state.yml
  3. work/{feature}/findings.yml
  4. CLAUDE.md, docs/TODO.md, docs/BACKLOG.md
  5. docs/sub-agents/*.md, .claude/agents/*.md, .claude/skills/*/SKILL.md
  6. Memory files (paths per CLAUDE.md)
  
  Output: work/{feature}/retro.md — full analysis with process metrics, 
  agent improvements, documentation gaps, git repo sync, lessons learned.",
  model="opus"
)
```

**After agent returns:**
1. Read `work/{feature}/retro.md`
2. Show findings to user as a table
3. Wait for approval: "fix" / "fix 1,3" / "skip" / "stop"
4. Fix ONLY approved items
5. For memory files: show exact change → apply → commit + push

### Round 2: Verify Fixes

```
Agent(
  prompt="Scrum Master. Read .claude/agents/scrum-master-agent.md.
  Epic: {EPIC}. Feature: {FEATURE}. Round: 2.
  Previous retro: work/{feature}/retro.md.
  Verify Round 1 fixes applied. Look for NEW issues from fixes.
  Overwrite work/{feature}/retro.md with Round 2 report.",
  model="opus"
)
```

**After agent returns:** same approval cycle as Round 1.

**⛔ Round 2 mandatory grep** (lesson source: real epic where 2/3 placeholder hashes were eyeballed and 1 missed):
```bash
grep -rn 'pending commit\|TBD\|TODO\|placeholder\|<COMMIT_HASH>' work/{feature}/*.yml
```
Any hit = NOT done. Patch before declaring Round 2 clean.

### Round 3: Final Check

```
Agent(
  prompt="Scrum Master. Read .claude/agents/scrum-master-agent.md.
  Epic: {EPIC}. Feature: {FEATURE}. Round: 3.
  Final check: is everything clean? Can we close the epic?
  Overwrite work/{feature}/retro.md with Round 3 verdict.",
  model="opus"
)
```

If clean → proceed to Completion. If findings remain → show user, fix, accept.

## Completion

After final round:

1. Log: `cd $PROJECT_DIR && uv run python scripts/conv_log.py {EPIC} claude phase "scrum-master complete: 3 rounds"`
2. Update `docs/BACKLOG.md` — mark epic as done
3. Update `docs/TODO.md` — archive completed items
4. Move `work/{feature}/` → `work/completed/{feature}/`
5. If memory files changed → `cd <MEMORY_PATH> && git add . && git commit -m "retro {EPIC}: updates" && git push`
6. Git commit project changes

## Checklist Reference

For detailed checklist items, see [review-checklist.md](references/review-checklist.md).

## Rules

- **Mandatory approval.** Agent finds → user decides → then fix
- **3 rounds.** Raw → verify → final. Not more, not less.
- **Agent does the analysis.** Orchestrator dispatches, shows results, executes fixes
- **Retrospective is mandatory.** After every epic — no skipping
- **Be specific.** File names, line numbers — not "docs are outdated"

> Example from project (G2-E5): After broadcast Claude asked "what next?" without retro. User: "You were supposed to propose this mandatory retrospective step."
