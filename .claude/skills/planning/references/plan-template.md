# Tech-Spec Template

Copy this to `work/{feature}/tech-spec.md` and fill in.

---

```markdown
# Tech-Spec: {feature_name}

Epic: {epic}
User-spec: `work/{feature}/user-spec.md`
Created: YYYY-MM-DD
Status: draft | approved

## Context

{Problem from user-spec + proposed solution, 2-3 sentences}

## Files to Modify

| File | What Changes |
|------|-------------|
| `src/<project>/core/texts.py` | New texts for {feature} |
| `src/<project>/core/flows.py` | New `build_{feature}_actions()` |
| `src/<project>/tg/handlers.py` | Handler for /{command} |
| `src/<project>/max/handlers.py` | Same handler for MAX |
| `src/<project>/cli/callbacks.py` | CLI callback for {feature} |
| ... | ... |

## Architecture Decisions

### Decision 1: {topic}
**Choice:** {what we chose}
**Why:** {rationale}
**Alternatives considered:** {what we rejected and why}

## Implementation Steps

### 1. {Step name}
- What: {concrete changes}
- Files: {which files}
- Tests: {what to test}

### 2. ...

## Architect Review

- [ ] Business logic in core/, not in handlers?
- [ ] No duplication between all frontends (TG/MAX/CLI)?
- [ ] All consumers covered?
- [ ] No platform imports in core/?
- [ ] `get_db_path()` instead of `DB_PATH`?
- [ ] Level progression formula respected (if applicable)?
- [ ] Rule of 5: no more than 5 items in any list?

## Testing Strategy

### Unit Tests (DEV)
- Setup: `uv run python tests/setup_test_matrix.py --db data/test_db.db`
- Run: `uv run python tests/cli_test_runner.py --users all --suite all --db data/test_db.db`
- Expected: all pass

### E2E Plan (STAGE, CLI)

1. **Prep:** user {id}, level {N}, reset if needed
2. **Actions:** {CLI commands to run}
3. **Expected:** {what should appear}
4. **DB check:** `SELECT ... FROM ... WHERE ...`

### Test Data
Reference: `docs/TEST_DATA.md`

## Deploy Plan

Stories in order, each with its own deploy cycle:

| Story | What | Deploy? |
|-------|------|---------|
| S1 | ... | No |
| S2 | ... | Yes |
| ... | ... | ... |

## Risks

| Risk | Impact | Mitigation |
|------|--------|-----------|
| {risk} | {what breaks} | {how to handle} |

## User-Spec Traceability

| User-Spec Requirement | Tech-Spec Coverage |
|----------------------|-------------------|
| {requirement 1} | Step {N}: {how it's covered} |
| {requirement 2} | Step {M}: {how it's covered} |
| ... | ... |
```
