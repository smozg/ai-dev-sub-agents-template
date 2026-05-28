# Testing Patterns — <PROJECT>

## Test Infrastructure

### Unit + Regression Tests (DEV)

```bash
cd $PROJECT_DIR

# Setup test matrix (creates test users with different levels/states)
uv run python tests/setup_test_matrix.py --db data/test_db.db

# Run all tests
uv run python tests/cli_test_runner.py --users all --suite all --db data/test_db.db
```

### E2E Tests (STAGE, via CLI)

```bash
# Connect to Run server
ssh root@<RUN_SERVER>

# STAGE CLI
cd <STAGE_PATH>
uv run <project>-cli --user <TEST_USER_ID> --db data/db.sqlite

# PROD CLI
cd <PROD_PATH>
uv run <project>-cli --user <TEST_USER_ID> --db data/db.sqlite
```

### Import Check

```bash
cd $PROJECT_DIR
uv run python -c "import <project>; print('OK')"
```

## Test Matrix

<!-- Define your test accounts and their states here -->
Test accounts with different states:
- User `<TEST_USER_ID>`: Level 1, fresh start
- Additional users at various levels/states as needed

See: `docs/TEST_DATA.md` for full list.

## TDD Pattern

### 1. Write failing test

```python
def test_new_feature_basic(self):
    """New feature returns expected actions."""
    actions = build_xxx_actions(self.db, self.user_id)
    assert len(actions) > 0
    assert any(isinstance(a, SendMessage) for a in actions)
```

### 2. Run — confirm FAIL

```bash
uv run python tests/cli_test_runner.py --users <TEST_USER_ID> --suite xxx --db data/test_db.db
```

### 3. Implement — make it PASS

### 4. Run full suite — confirm no regression

```bash
uv run python tests/cli_test_runner.py --users all --suite all --db data/test_db.db
```

## What to Test

| Change Type | What to Test |
|------------|-------------|
| New command | All user states (L1-LN), with/without data |
| New text | Rendering in all frontends (TG/MAX/CLI) |
| Parser change | Parse result saved correctly, missing field handled |
| DB migration | Old data still works, new column populated |
| Level progression | Unlock flow, reward, preview text |
