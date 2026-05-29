# /scrum-master-monthly

Aggregate conv_log + weekly retros across past 30 days. Process-level retro.

## Step 1: Query window

```bash
cd <PROJECT_DIR> && uv run python -c "
import sqlite3
db = sqlite3.connect('data/conversation.db')
rows = db.execute(\"\"\"
    SELECT epic, COUNT(*) as entries,
           SUM(CASE WHEN type='decision' THEN 1 ELSE 0 END) as decisions,
           SUM(CASE WHEN type='insight' THEN 1 ELSE 0 END) as insights,
           SUM(CASE WHEN type='message' AND role='user' THEN 1 ELSE 0 END) as user_msgs
    FROM messages WHERE ts >= datetime('now','-30 days')
    GROUP BY epic ORDER BY entries DESC
\"\"\").fetchall()
print('Epic activity (30 days):')
for r in rows: print(f'  {r[0]}: {r[1]} entries (D{r[2]} I{r[3]} U{r[4]})')
"
```

## Step 2: Read all weekly retros from this month

```bash
ls docs/retros/weekly-*.md | tail -4  # last 4 weeks
```

## Step 3: Dispatch scrum-master agent

```
Agent(
  prompt="Monthly process retrospective. Read:
  1. Conv_log query (last 30 days, query above)
  2. All weekly retros from docs/retros/weekly-*.md from this month
  3. CLAUDE.md + memory files (feedback_*.md) — check freshness

  Process-level analysis:
  - Which epics shipped vs slipped? Trends.
  - Discipline gaps reproducing across multiple features → systemic
  - Memory churn — how many feedback_*.md files created/updated?
  - Are rules being followed (Hotfix threshold, audit gates, etc.)?
  - Trust events — any user CORRECTIONs that hurt confidence?

  Output: docs/retros/monthly-YYYY-MM.md with:
  - Top 5 process wins
  - Top 5 process losses
  - Recommended CLAUDE.md or agent prompt updates
  - Memory file consolidation suggestions",
  subagent_type="scrum-master-agent",
  model="opus"
)
```

## Step 4: Process + commit

Same approval cycle. Major rule changes (CLAUDE.md) — explicit user OK first.

## Schedule

- Run **last Friday of each month** OR by command
- Cron entry: optional (months don't divide evenly into days, easier manual)
