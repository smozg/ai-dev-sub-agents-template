---
description: "Quick code task without full discovery/planning. For bug fixes, small features, experiments."
argument-hint: "[task description]"
---

# Write Code (Quick Mode)

For small tasks that don't need full Discovery → Planning pipeline.

## When to Use
- Bug fixes
- Small UI changes
- Adding a callback handler
- Experiments
- Tasks where the user gave specific instructions

## Process

1. Load the `development` skill in **Quick Mode** (skip tech-spec)
2. Read `docs/TODO.md` for context
3. **If task touches 3+ files** → dispatch worktree agent (see development skill "Orchestrator Dispatch")
4. **If task touches 1-2 files** → code directly:
   - If touches `core/`: TDD — write tests first
   - Write code following the 5 rules
   - Run tests: `uv run python tests/cli_test_runner.py`
   - Run code-reviewer agent
5. Commit

Task: $ARGUMENTS

**Note:** For features that need business validation (new level, new command, CJM changes), use `/new-feature` instead.
