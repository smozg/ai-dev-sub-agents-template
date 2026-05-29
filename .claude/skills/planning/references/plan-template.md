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

⛔ **MANDATORY 3-interface checklist** (per project rule "any feature in core/ → wired to TG, MAX, AND CLI" — lesson source: real project HF2 where CLI was forgotten twice in one session). Any feature in `core/` MUST be wired in **TG + MAX + CLI**:

- [ ] `<PROJECT_DIR>/src/<pkg>/tg/handlers.py` — TG command/callback handler
- [ ] `<PROJECT_DIR>/src/<pkg>/max/handlers.py` — MAX command/callback handler (with `mx_*` prefix awareness if using auto-prefix renderer)
- [ ] `<PROJECT_DIR>/src/<pkg>/cli/callbacks.py` — CLI dispatcher for test_cases.json coverage

If an interface is intentionally skipped (admin-only, CLI N/A) — **document the reason explicitly** in Architecture Decisions section. Default = include all three.

| File | What Changes |
|------|-------------|
| `<PROJECT_DIR>/src/<pkg>/core/texts.py` | New texts for {feature} |
| `<PROJECT_DIR>/src/<pkg>/core/flows.py` | New `build_{feature}_actions()` |
| `<PROJECT_DIR>/src/<pkg>/tg/handlers.py` | Handler for /{command} |
| `<PROJECT_DIR>/src/<pkg>/max/handlers.py` | Same handler for MAX |
| `<PROJECT_DIR>/src/<pkg>/cli/callbacks.py` | CLI callback for {feature} |
| `<PROJECT_DIR>/tests/test_cases.json` | New T0XX entries for new callbacks (CLI coverage) |
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

## Parallel Execution Plan (MANDATORY — G2-E10-S25)

Even if answer is "single-wave sequential", this section must exist with explicit justification.

### File overlap matrix

| Task # | Files touched | Conflicts with | Wave |
|---|---|---|---|
| 1 | `src/obrep/db.py` + `tests/test_X.py` | none | 1 |
| 2 | `src/obrep/tg/renderer.py` | none | 1 |
| 3 | `src/obrep/tg/handlers.py` | depends on task 1 (imports updated signature) | 2 |
| 4 | `src/obrep/max/renderer.py` + `max/handlers.py` | none | 1 |

### Wave breakdown

- **Wave 1 (parallel):** tasks 1, 2, 4 — independent file scopes
- **Wave 2 (sequential after wave 1):** task 3 — depends on task 1's signature change
- **Wave 3:** integration tests run on merged worktrees

### Estimated savings

- Sequential time: {N tasks} × {avg min} = {total} min
- Parallel waves: max(wave 1 tasks) + max(wave 2) + ... = {realistic} min
- **Save: {percent}%**

### Anti-pattern checklist

- [ ] No two tasks in same wave edit same file (verified via grep)
- [ ] No task in Wave N depends on incomplete task in Wave N
- [ ] Worktree merge order documented (alphabetical by branch / by wave order / etc.)
- [ ] No "parallel" claims that are actually sequential by hidden dependency

### Single-wave justification (if no parallelization possible)

If all tasks must be sequential, document WHY explicitly:
> "Single-wave sequential — all tasks share signature changes in db.py. Wave split increases merge risk without net gain."

Common valid reasons:
- Single file change
- Signature cascade through tightly-coupled modules
- TDD pair (tests + impl in one atomic unit)

Architect agent will verify this section. Missing section → CHANGES_REQUIRED.

## Architect Review (dispatched agent — Opus)

Architect agent (`.claude/agents/architect.md`) verifies:
- 5 code rules compliance
- Parallel Execution Plan correctness (grep file overlaps)
- Dependency graph (no circulars)
- Architecture pattern compliance (vs memory files)
- Risk identification

Run after skeptic + completeness:
```
Agent(
  prompt="Validate tech-spec at work/{feature}/tech-spec.md per .claude/agents/architect.md.
  Report verdict APPROVED/CHANGES_REQUIRED.",
  subagent_type="architect",
  model="opus"
)
```

If APPROVED → user approval → development.
If CHANGES_REQUIRED → fix tech-spec, re-run.

## Testing Strategy

### Unit Tests (DEV)
- Setup: `uv run python tests/setup_test_matrix.py --db data/test_obrep.db`
- Run: `BOT_ID=geobrand_bot uv run python tests/cli_test_runner.py --users all --suite all --db data/test_obrep.db`
- Expected: all pass (currently 618+)

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
