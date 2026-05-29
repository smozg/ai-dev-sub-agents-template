# /scrum-master-weekly

Aggregate conv_log across past 7 days. Run manually OR via Monday 09:00 MSK cron.

## Step 1: Query window

```bash
cd <PROJECT_DIR> && uv run python -c "
import sqlite3
db = sqlite3.connect('data/conversation.db')
rows = db.execute(\"\"\"
    SELECT id, epic, ts, role, type, substr(content,1,200)
    FROM messages
    WHERE ts >= datetime('now', '-7 days')
    ORDER BY ts
\"\"\").fetchall()
for r in rows: print(f'#{r[0]} [{r[1]}] [{r[3]}/{r[4]}] {r[5]}')
"
```

## Step 2: Dispatch scrum-master agent

```
Agent(
  prompt="Weekly retrospective. Read conv_log from last 7 days (query above).
  Group by epic. For each epic touched, audit:
  - Conv_log discipline (entries per phase vs expected min from pipeline-state)
  - User CORRECTION/REMINDER count — recurring patterns?
  - Cross-epic systemic issues (e.g., 'forgot to log buttons' twice = systemic)

  Look for:
  - Process drift (rules being skipped)
  - Repeated user feedback
  - Bottlenecks (epics taking longer than expected)
  - Quality regressions (hotfix loops increasing?)

  Output: docs/retros/weekly-YYYY-WW.md
  If systemic finding → propose memory update OR rule change in CLAUDE.md.",
  subagent_type="scrum-master-agent",
  model="opus"
)
```

## Step 3: Process findings

Show user the weekly retro. If action items → approval cycle (same as feature retro).

## Step 4: Commit

```bash
git add docs/retros/weekly-*.md && git commit -m "weekly retro: YYYY-WW"
```

## Cron schedule (Monday 09:00 MSK)

`/etc/cron.d/obrep-weekly-retro` reminds user at 09:00 every Monday to run this command.

## When to skip

- Если за неделю не было активности (≤5 conv_log entries) — note "quiet week" и пропустить
