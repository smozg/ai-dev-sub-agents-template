---
description: "Finish current task: dispatch Scrum Master Agent (3 rounds) → update docs → archive feature."
---

# Done — Close Current Task

## Step 1: Scrum Master Agent (3 Rounds)

Load the `scrum-master` skill. It dispatches the Scrum Master Agent (Opus) for 3 rounds:

| Round | Focus |
|-------|-------|
| 1 | Raw analysis — process, agents, docs, git repo |
| 2 | Verify Round 1 fixes applied, check for new issues |
| 3 | Final — "clean? can we close?" |

**Remember:** agent finds → show user → user approves → then fix.

## Step 2: Update Documentation

After review completes:

1. **BACKLOG.md** — mark story/epic as ✅ with date
2. **TODO.md** — archive completed items
3. **Memory files** — update if architecture/CJM/product decisions changed (with user consent)
4. **Git commit:**
   ```bash
   cd <PROJECT_DIR> && git add . && git commit -m "docs: close {epic}" && git push
   ```

## Step 3: Archive Feature (if epic complete)

If the entire epic is done:
```bash
mv work/{feature}/ work/completed/{feature}/
```

## Step 4: Log

```bash
cd <PROJECT_DIR> && uv run python scripts/conv_log.py {EPIC} claude phase "task closed"
```

## Step 5: Second Brain Entry

Ask user if they want to record achievement:
- File: `/root/Automations/Telegram_Bot/Agent_Second_Brain/vault/daily/YYYY-MM-DD.md`
- Format: `## HH:MM [achievement]\nWhat was done (1-3 lines)`
