---
description: "Start a new feature: 3-cycle discovery interview → planning with validators. Use for new levels, features, or significant changes."
argument-hint: "[feature name or description]"
---

# New Feature Pipeline

## Step 1: Discovery

Load the `discovery` skill and begin the 3-cycle interview for: $ARGUMENTS

The discovery skill will:
1. Create `work/{feature}/` directory with interview.yml
2. Run 3 interview cycles (Business → UX/CJM → Technical)
3. Score all fields (85% threshold on required)
4. Create and get approval for user-spec.md

**Do NOT proceed to Step 2 until user approves the user-spec.**

## Step 2: Planning

After user-spec is approved, load the `planning` skill:
1. Research codebase based on user-spec requirements
2. Create tech-spec (implementation plan) in `work/{feature}/tech-spec.md`
3. Run skeptic agent (verify files/functions exist)
4. Run completeness-validator agent (all requirements covered)
5. Run architect review checklist
6. Create/update `docs/TODO.md`
7. Get user approval for the plan

**Do NOT write code until user approves the plan.**

## Step 3: Handoff

After plan is approved:
- Update `docs/BACKLOG.md` with new stories (if epic)
- Log: `cd $PROJECT_DIR && uv run python scripts/conv_log.py {EPIC} claude phase "planning complete, ready for development"`
- Inform user: "Plan approved. Use `/write-code` or `/deploy` when ready."
