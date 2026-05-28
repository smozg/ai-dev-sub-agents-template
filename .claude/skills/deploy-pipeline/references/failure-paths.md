# Failure Paths — What To Do When Something Goes Wrong

## Sub-agent Found a Bug (Steps 4, 5, 9, 10)

```
Bug found by sub-agent
  → Fix code on Code server
  → Run tests locally: cli_test_runner.py
  → scp fixed files to Run server
  → Restart service
  → Re-launch the SAME sub-agent from scratch (fresh context)
  → Sub-agent passes? → Continue to next step
  → Sub-agent fails again? → Fix and repeat (max 3 attempts)
  → Still failing after 3? → Notify user, discuss
```

## SSH Timeout During scp/restart

```
SSH timeout
  → This is NOT a successful deploy
  → Retry the SSH command
  → If persistent: check if Run server is reachable (ping <RUN_SERVER>)
  → After reconnect: verify with grep that files arrived
  → Re-run sub-agent — do NOT skip verification
```

## Tests Fail After Fix (Regression)

```
Tests fail
  → Read the failing test output carefully
  → Check: did the fix break an existing test?
  → If yes: fix the regression, run full suite again
  → If false positive: investigate test setup
  → NEVER deploy with failing tests
```

## User Reports Bug During Smoke Test

```
User reports bug
  → Ask for screenshot / exact steps to reproduce
  → Check: does the bug reproduce in CLI?
  → If yes: fix → tests → scp → restart → sub-agent → user
  → If CLI works but TG/MAX doesn't: check handler sync
  → Same deploy cycle applies to hotfixes
```

## Parser Not Returning Data (if applicable)

```
Parser returns empty/wrong data
  → Check: is the right parser running? (staging vs prod)
  → ssh root@<RUN_SERVER> "systemctl is-active <project>-worker-staging <project>-worker"
  → Check queue for pending jobs
  → Run parser manually if needed
  → NEVER call external APIs directly from within a handler — only through worker
```

## Wrong Service Restarted

```
Accidentally restarted wrong service
  → If restarted on Code server: stop it immediately
  → If restarted prod instead of stage: no damage, but verify prod is stable
  → Correct the command and proceed
```

## Rollback

If a deploy causes critical issues on PROD:

```bash
# 1. Check git for last known good commit
cd $PROJECT_DIR
git log --oneline -5

# 2. Checkout the good version of specific files
git checkout {commit} -- src/<project>/{file}.py

# 3. Re-deploy to PROD
scp src/<project>/{file}.py root@<RUN_SERVER>:<STAGE_PATH>/src/<project>/{file}.py
scp src/<project>/{file}.py root@<RUN_SERVER>:<PROD_PATH>/src/<project>/{file}.py
ssh root@<RUN_SERVER> "systemctl restart <prod-service-1> <prod-service-2>"

# 4. Verify rollback
ssh root@<RUN_SERVER> "grep -c 'KNOWN_GOOD_STRING' <PROD_PATH>/src/<project>/{file}.py"
```

For full rollback to tagged version:
```bash
git checkout v0.X-stable -- src/<project>/
# Then scp all files...
```
