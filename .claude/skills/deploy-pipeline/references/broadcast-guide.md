# Broadcast Guide

## Script: broadcast.py

Location: `scripts/broadcast.py`

### Where to run

**ONLY on Run server (`<RUN_SERVER>`) via SSH!** STAGE/PROD databases are only there.

Do NOT run locally on Code server — no active databases there.

### Bots

| Environment | Bot | `--env` |
|-------------|-----|---------|
| STAGE | @<stage_bot> | `stage` |
| PROD | @<prod_bot> | `prod` |

### Usage

```bash
# First copy text to Run:
scp docs/marketing/broadcast_{feature}.txt root@<RUN_SERVER>:<STAGE_PATH>/docs/marketing/

# For PROD — also copy to prod path:
scp docs/marketing/broadcast_{feature}.txt root@<RUN_SERVER>:<PROD_PATH>/docs/marketing/

# STAGE preview (heredoc — cd is required!)
ssh root@<RUN_SERVER> << 'REMOTE'
cd <STAGE_PATH>
uv run python scripts/broadcast.py --env stage --preview <ADMIN_USER_ID> --file docs/marketing/broadcast_{feature}.txt --button "Label" callback
REMOTE

# PROD preview
ssh root@<RUN_SERVER> << 'REMOTE'
cd <PROD_PATH>
uv run python scripts/broadcast.py --env prod --preview <ADMIN_USER_ID> --file docs/marketing/broadcast_{feature}.txt --button "Label" callback
REMOTE

# Full send (echo y — no --yes flag)
ssh root@<RUN_SERVER> << 'REMOTE'
cd <PROD_PATH>
echo "y" | uv run python scripts/broadcast.py --env prod --max-level 7 --file docs/marketing/broadcast_{feature}.txt --button "Label" callback
REMOTE
```

### Flags

| Flag | What |
|------|------|
| `--env stage\|prod` | Target environment |
| `--preview {user_id}` | Send only to this user (test) |
| `--personalized` | Use user's own data in message |
| `--max-level {N}` | Only users with level < N (haven't bought this level) |
| `--button "Label" callback` | Add inline button |
| `--file path` | Load message text from file |

### Personalization

When `--personalized` is used, the script:
1. Loads each user's data from DB
2. Substitutes placeholders in the message template
3. If user has no data — sends generic version

### Recipients

- TG users: sent via Telegram API
- MAX users: sent via MAX API (if applicable)
- Skipped: users with `is_blocked=True` or missing token

### Output

Reports delivery stats:
```
Delivered: 7/9 (TG: 5, MAX: 2)
Skipped: 2 (no MAX token)
```

## Message Writing Tips

### Structure
1. **Hook** — one sentence about the user's result (personalized if possible)
2. **Value** — what the new feature does (2-3 sentences)
3. **CTA** — button to try it

### Tone
- Gamified, friendly
- Bot speaks as "I" or addresses user as "you"
- Short — readable in 10 seconds
- Lines ≤ 40 chars (mobile screen)

### Example (gamified notification)
```
🆕 New command: /fresh

Your card is filled in — but competitor is still ranked higher?
It might be about content freshness.

/fresh shows 7 metrics and a score out of 100.
Find out what needs updating.

[🕐 Check freshness]
```
