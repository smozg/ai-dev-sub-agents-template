# Deploy Commands Reference

<!-- 
CONFIGURATION: Fill in your project-specific values.
These are the commands your Deploy Agent will use.
-->

## Servers

| Server | Role | IP |
|--------|------|----|
| **Code** | Development | `<CODE_SERVER>` (you are here) |
| **Run** | STAGE + PROD | `<RUN_SERVER>` |

## Project Paths on Run Server

| Environment | Base Path |
|-------------|-----------|
| **STAGE** | `<STAGE_PATH>` |
| **PROD** | `<PROD_PATH>` |

## scp Patterns

### Single file (core/)
```bash
scp src/<project>/core/flows.py root@<RUN_SERVER>:<STAGE_PATH>/src/<project>/core/flows.py
```

### Handler files (specify full subpath!)
```bash
# Wrong: generic glob may overwrite wrong file
# Right: specify full path for each handler
scp src/<project>/tg/handlers.py root@<RUN_SERVER>:<STAGE_PATH>/src/<project>/tg/handlers.py
scp src/<project>/max/handlers.py root@<RUN_SERVER>:<STAGE_PATH>/src/<project>/max/handlers.py
scp src/<project>/cli/callbacks.py root@<RUN_SERVER>:<STAGE_PATH>/src/<project>/cli/callbacks.py
```

### PROD — deploy to BOTH paths (scp breaks hardlinks!)
```bash
# STAGE path:
scp src/<project>/core/flows.py root@<RUN_SERVER>:<STAGE_PATH>/src/<project>/core/flows.py

# PROD path (same file, second copy):
scp src/<project>/core/flows.py root@<RUN_SERVER>:<PROD_PATH>/src/<project>/core/flows.py
```

### Batch multiple files (use && to chain)
```bash
scp src/<project>/core/flows.py root@<RUN_SERVER>:<STAGE_PATH>/src/<project>/core/flows.py && \
scp src/<project>/core/texts.py root@<RUN_SERVER>:<STAGE_PATH>/src/<project>/core/texts.py && \
scp src/<project>/tg/handlers.py root@<RUN_SERVER>:<STAGE_PATH>/src/<project>/tg/handlers.py
```

## Restart Commands

### STAGE
```bash
ssh root@<RUN_SERVER> "systemctl restart <stage-service-1> <stage-service-2>"
```

### PROD
```bash
ssh root@<RUN_SERVER> "systemctl restart <prod-service-1> <prod-service-2>"
```

### Parser (if applicable)
```bash
# STAGE parser (auto-stops PROD via Conflicts= in systemd):
ssh root@<RUN_SERVER> "systemctl start <project>-worker-staging"

# PROD parser (auto-stops STAGE):
ssh root@<RUN_SERVER> "systemctl start <project>-worker"
```

## Verify Deploy

```bash
# Check file was copied (grep for a unique string):
ssh root@<RUN_SERVER> "grep -c 'UNIQUE_STRING' <STAGE_PATH>/src/<project>/core/flows.py"

# Check service is running:
ssh root@<RUN_SERVER> "systemctl is-active <stage-service-1>"

# Check logs for errors:
ssh root@<RUN_SERVER> "journalctl -u <stage-service-1> --since '5 min ago' --no-pager | tail -20"
```

## CLI on Run Server

```bash
# STAGE CLI
ssh root@<RUN_SERVER>
cd <STAGE_PATH>
uv run <project>-cli --user <TEST_USER_ID> --db data/db.sqlite

# PROD CLI
ssh root@<RUN_SERVER>
cd <PROD_PATH>
uv run <project>-cli --user <TEST_USER_ID> --db data/db.sqlite
```

## Website Deploy

```bash
# Build on Code server
cd <SITE_SRC> && npm run build

# Copy to Run server
scp -r <SITE_SRC>/dist/ root@<RUN_SERVER>:<SITE_DIST>/

# Never edit dist/ directly — only src/ → build → scp
```

## CRITICAL RULES

1. **NEVER** `systemctl restart` on Code server
2. **ALWAYS** scp to BOTH paths on PROD if hardlinks are used
3. **ALWAYS** verify with grep after scp (file actually arrived)
4. **ALWAYS** specify full subpath for handler files (tg/, max/, cli/)
