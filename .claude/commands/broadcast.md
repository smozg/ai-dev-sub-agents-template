---
description: "Launch marketing broadcast: 3-gate review → preview → production send."
---

# Broadcast — Marketing Pipeline

## Step 1: Load Marketing Skill

Load the `marketing` skill and execute S5c (Broadcast — 3-Gate Review):

1. **Draft message** — personalized, with CTA button, filtered by level
2. **Gate 1: PM review** — spelling, voice, clarity, tone
3. **Gate 2: UX review** — style guide, emoji, structure
4. **Gate 3: Tester** — deploy on STAGE, buttons work, callbacks fire

## Step 2: Preview (⛔ Run server only!)

```bash
# Сначала скопировать текст на Run:
scp docs/marketing/broadcast_{feature}.txt root@<RUN_SERVER_IP>:<PROJECT_DIR>/docs/marketing/

# STAGE preview:
ssh root@<RUN_SERVER_IP> "uv --directory <PROJECT_DIR> run python scripts/broadcast.py --env stage --preview <ADMIN_USER_ID> --file docs/marketing/broadcast_{feature}.txt --button 'Label' callback"
```

**⛔ Wait for user approval before production preview.**

```bash
# PROD preview:
ssh root@<RUN_SERVER_IP> "uv --directory <PROJECT_DIR> run python scripts/broadcast.py --env prod --preview <ADMIN_USER_ID> --file docs/marketing/broadcast_{feature}.txt --button 'Label' callback"
```

**⛔ Wait for user approval before full send.**

## Step 3: Production Send (⛔ Run server only!)

```bash
ssh root@<RUN_SERVER_IP> "echo y | uv --directory <PROJECT_DIR> run python scripts/broadcast.py --env prod --file docs/marketing/broadcast_{feature}.txt --max-level {N} --button 'Label' callback"
```

## Step 4: Log

```bash
cd <PROJECT_DIR> && uv run python scripts/conv_log.py {EPIC} claude decision "broadcast sent: {delivered} delivered, {skipped} skipped"
```
