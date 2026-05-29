# Trust Event Recovery Playbook

**Trigger:** user discovers Claude misrepresented capability — "ты говорил что можешь X, но реально не можешь / данных нет / table пустая".

**Examples to date:**
- G2-E10-S9 (2026-05-28): `message_log` была schema-only weeks. Claude несколько раз обещал "посмотрю что бот отправлял user X" не проверяя что table пустая.

**Severity:** ALWAYS treat as P0. Trust is harder to rebuild than to maintain.

---

## Sequence (proven on G2-E10-S9 → S23)

### 1. **Immediately acknowledge honestly** (текст, не код)

Don't:
- "Я не специально"
- "Это было давно"
- "Будем фиксить"

Do:
- Explicitly name the failure: "Я misrepresented capability — table была пустая, я не проверял, говорил что данные есть"
- Trace the chain: schema → no implementation → multiple sessions assumed it works
- Apologize without excuse

### 2. **Read user's directives carefully**

After trust event, user typically asks for one or more of:
- Full process (no shortcuts)
- Additional rule changes
- Specific verification gates
- Quality bar higher than usual

**Don't propose shortcuts.** Honour exact scope.

### 3. **Inc-prob entry in memory FIRST** (before code changes)

```
/root/.claude/projects/-root/memory/obrep-bot-inc-prob.md
```

Add entry with:
- Severity tag "TRUST EVENT"
- Root cause (multi-layered: schema-only, no validation, false claim)
- Impact (what was misled, for how long)
- Process changes triggered

### 4. **Process changes commit (`process(...)`) BEFORE feature commit**

Pattern: separate the "rule change" from the "feature fix":

```
6c78721  process(G2-E10-S23): trust event response — 9-item rule changes
94145ef  feat(G2-E10-S23): message_log full audit — base
c2e98b5  fix(G2-E10-S23): close 7+ reviewer findings
e410f78  fix(G2-E10-S23): Round 2 nits
```

Why: future contributors can see "we changed rules because X" independently from "we shipped feature Y". Process changes outlive features.

### 5. **Full `/new-feature` pipeline — no shortcuts**

Even if scope seems clear:
- Discovery 3-cycle YAML interview (even if pre-fill 80%, ASK explicit questions)
- Planning + skeptic + completeness + architect review
- Worktree agent for code (≤1 hotfix rule applies)
- TWO reviewers (Sonnet code + Opus financial/data) — convergence pattern
- Multiple commit iterations if needed
- PROD smoke with EXPLICIT verification queries (not vague "check DB")

### 6. **PROD smoke uses EXPLICIT verification queries**

Don't: "проверь что работает"

Do:
- Predefine SQL queries / commands
- Run them after smoke
- Show user the actual output
- Map to user-spec acceptance criteria

Example (G2-E10-S23):
```sql
SELECT id, user_id, direction, substr(text,1,50), buttons, context, created_at
FROM message_log
WHERE created_at >= datetime('now','-15 minutes')
ORDER BY id DESC LIMIT 25
```

User sees real data → trust visibly restored.

### 7. **3-round retro mandatory + explicit "trust restored?" question**

Round 1: full analysis, action items
Round 2: verify fixes, look for NEW issues
Round 3: final clean OR another iteration

In Round 1 retro, scrum-master agent MUST ask:
- "Was trust restored? Evidence?"
- "Residual risks?"
- "Process change that would have prevented original event?"

### 8. **Process changes ENFORCED in future**

After playbook execution, expect rule changes like (per G2-E10-S23):
- Tighter hotfix threshold (≤1 file)
- New audit gates in pipeline
- New reviewer agent type
- New mandatory verification step

These rules should land in CLAUDE.md `НИКОГДА` section + corresponding skill/agent files.

---

## Lessons consolidated

| # | Pattern | Why critical |
|---|---|---|
| 1 | Honest acknowledgment without excuse | Foundation of recovery — user needs to see you don't dodge |
| 2 | Inc-prob entry BEFORE code | Memory survives session compact; ensures future sessions know |
| 3 | Separate process commit from feature commit | Rule changes ≠ feature, both deserve own commit |
| 4 | Full pipeline no shortcuts | Trust isn't restored by speed, by THOROUGHNESS |
| 5 | Dual reviewer (Sonnet + Opus) | Convergence catches what single reviewer misses (G2-E10-S20 + S23 both validated) |
| 6 | Explicit verification queries | Vague "check it works" lets next trust event in |
| 7 | 3-round retro + "trust restored?" | Forces explicit answer, not assumption |
| 8 | Rule changes go to CLAUDE.md НИКОГДА | Make recurrence harder mechanically |

---

## When NOT to use this playbook

- Regular bugs (use `/write-code` или standard pipeline)
- Performance issues (use standard incident response)
- User reports something not working that **does** work — verify first; if not a real bug, no trust event

Trust event ≠ "user is annoyed". Trust event = "user discovers Claude said X but X is false".

---

## Related

- `obrep-bot-inc-prob.md` — incident log including trust events
- `CLAUDE.md` НИКОГДА section — mechanical rule enforcement
- `.claude/skills/scrum-master/SKILL.md` — 3-round retro process
- `feedback_signature_change_callers.md` — G2-E10-S20 lesson
- `feedback_buttons_not_logged_lesson.md` (TBD) — G2-E10-S23 lesson
