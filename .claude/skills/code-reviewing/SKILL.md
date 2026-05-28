---
name: code-reviewing
description: |
  Methodology for reviewing code quality across 11 dimensions.
  Used by the code-reviewer agent.

  Use when: "code review", "review code", "check code quality"
---

# Code Review Methodology

## 11 Review Dimensions

### 1. Architecture (Critical)
- [ ] Business logic in `core/`, not in handlers?
- [ ] No platform imports (`telegram`, `maxapi`, etc.) in `core/`?
- [ ] Follows Action-Response pattern? (build_*_actions → list[Action])
- [ ] Uses `get_db_path()`, not hardcoded paths?

### 2. Sync across all frontends (Critical)
- [ ] New handler in ALL frontends (TG, MAX, CLI, etc.)?
- [ ] Same texts from `core/texts.py` in all frontends?
- [ ] Same callback_data in all frontends?
- [ ] New `build_*_actions()` called from all frontends?

### 3. Texts & UX
- [ ] All strings in `core/texts.py`?
- [ ] Tone: gamified, friendly, no filler?
- [ ] Rule of 5: no list longer than 5 items?
- [ ] CTA present where expected?

### 4. Error Handling
- [ ] Missing data → graceful fallback (not crash)?
- [ ] DB query returns None → handled?
- [ ] Division by zero → guarded?

### 5. Database
- [ ] New columns → migration handled?
- [ ] Indexes on frequently queried columns?
- [ ] Transactions where needed (multi-table writes)?

### 6. Tests
- [ ] New functions have tests?
- [ ] Edge cases tested (no data, max values, zero)?
- [ ] Tests use test DB, not production?

### 7. Security
- [ ] No hardcoded tokens or secrets?
- [ ] No raw SQL with user input (SQL injection)?
- [ ] API tokens only in environment vars, never in code?

### 8. Performance
- [ ] No N+1 queries (loop of DB calls)?
- [ ] Parser not called unnecessarily?
- [ ] Large text building uses list join, not string concat?

### 9. Callback Safety
- [ ] All `query.answer()` wrapped in safe answer helper?
- [ ] No bare `await query.answer()` in handlers?
- [ ] Callback handlers continue even if answer fails (30sec Telegram timeout)?

### 10. Markdown/URL Safety
- [ ] Messages with dynamic URLs use `parse_mode="plain"`?
- [ ] Tokens with `_`, `*`, `` ` `` won't break Markdown parsing?
- [ ] URL messages tested with special chars in tokens?

### 11. Signature Change → All-Callers Exhaustiveness
- [ ] If diff changes signature of any shared function (new param, removed param, renamed) — grep ALL callers and verify each is updated?
  ```bash
  grep -rn "func_name(" src/<project>/ --include="*.py" | grep -v "def func_name"
  ```
- [ ] For audit/safety columns (e.g. `terminal_id`, `bot_id`) — does every INSERT site pass the new value? Not just the ones tech-spec lists.
- [ ] If caller intentionally doesn't pass new arg → explicitly documented (default value, comment)?

## Severity Classification

| Severity | What | Action |
|----------|------|--------|
| **Critical** | Architecture violation, security issue, missing handler | Must fix before deploy |
| **Major** | Missing edge case, test gap, sync issue | Fix before deploy |
| **Minor** | Naming, style, small optimization | Fix if quick |

## Report Format

```
## Code Review: {feature}

### Critical
- {file}:{line} — {issue} → {suggestion}

### Major
- {file}:{line} — {issue} → {suggestion}

### Minor
- {file}:{line} — {issue} → {suggestion}

### Passed
- Architecture: ✅
- Sync all frontends: ✅
- ...

**Verdict: approve | changes_required**
```
