# Code Rules — <PROJECT>

<!--
CUSTOMIZE: Replace these 5 rules with your project's architecture invariants.
These are rules whose violation = a guaranteed bug. Keep to exactly 5.
-->

## 5 Rules (violation = bug)

### 1. Business logic ONLY in core/
All logic lives in `src/<project>/core/`. Platform handlers (TG/MAX/CLI) are thin wrappers that call core functions and render results.

**Good:** `core/flows.py` → `build_feature_actions(db, user_id)` → returns `list[Action]`
**Bad:** Computing scores inside `tg/handlers.py`

### 2. Texts ONLY in core/texts.py
Every user-facing string is defined in `core/texts.py`. No hardcoded strings in handlers.

**Good:** `texts.MSG_FEATURE_TITLE` used in `core/flows.py`
**Bad:** `"Feature Title"` hardcoded in `tg/handlers.py`

### 3. Every feature → ALL frontends
When adding a feature to core/, it MUST be available in all platform frontends:
- `src/<project>/tg/handlers.py` — Telegram handler
- `src/<project>/max/handlers.py` — MAX handler
- `src/<project>/cli/callbacks.py` — CLI callback

### 4. No platform imports in core/
`core/` must never import `telegram`, `maxapi`, or CLI modules. Core is platform-agnostic.

**Check:** `grep -r "import telegram\|import maxapi\|from telegram\|from maxapi" src/<project>/core/` → must return nothing.

### 5. DB via get_db_path()
Always use `get_db_path()` function, never hardcode `DB_PATH` or path strings.

## Code Structure

```
src/<project>/
  core/                  ← ALL business logic
    actions.py           ← Action dataclasses (SendMessage, EditMessage, Button, ...)
    renderer.py          ← Renderer Protocol + render_actions() dispatcher
    texts.py             ← All user-facing strings and button specs
    flows.py             ← build_*_actions() → list[Action]
  tg/handlers.py         ← Telegram handlers (thin wrappers)
  max/handlers.py        ← MAX handlers (thin wrappers)
  cli/
    main.py              ← CLI REPL
    callbacks.py         ← CLI callback dispatcher
  db.py                  ← Database operations
  parser.py              ← External data parser
```

## Patterns to Follow

### Adding a new command (/xxx)

1. `core/texts.py` — add text constants
2. `core/flows.py` — add `build_xxx_actions(db, user_id)` → `list[Action]`
3. `tg/handlers.py` — add handler calling `build_xxx_actions` + `render_actions`
4. `max/handlers.py` — same handler for MAX
5. `cli/callbacks.py` — add callback mapping
6. Tests — add to test matrix

### Adding parser fields (if applicable)

1. `parser.py` — add extraction
2. `db.py` — add column to relevant table (if new data)
3. `core/flows.py` — use new fields in build_*_actions()

### Level progression formula (if applicable)

Congratulation → crystals/reward → preview → unlock:
1. `core/texts.py` — congratulation text, reward, preview text (PAIN + DEMO + HOOK)
2. `core/flows.py` — `build_level_up_actions()` sequence
3. `core/subscriptions.py` — pricing if paid level
