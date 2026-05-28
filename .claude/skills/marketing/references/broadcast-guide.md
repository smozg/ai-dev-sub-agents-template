# Broadcast Guide

## Script: broadcast.py

Location: `scripts/broadcast.py`

### Where to run

**ONLY on Run server (`<RUN_SERVER>`) via SSH!** STAGE/PROD databases are only there.

Do NOT run locally on Code server — no active databases there.

### Usage

See [deploy-pipeline/references/broadcast-guide.md](../../deploy-pipeline/references/broadcast-guide.md) for full usage guide.

This file mirrors the deploy-pipeline reference for skill-level access.

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

### Example broadcast message
```
🆕 New command: /fresh

Your card is filled in — but competitor is ranked higher? 
It might be about content freshness.

/fresh shows 7 metrics and a score out of 100.
Find out what needs updating.

[🕐 Check freshness]
```
