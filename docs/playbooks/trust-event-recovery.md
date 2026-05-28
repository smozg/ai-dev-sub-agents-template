# Trust Event Recovery Playbook

**Trigger:** user discovers Claude misrepresented a capability — "you said you could X, but really you can't / data isn't there / table is empty".

**Definition:** Trust event = "user discovers Claude said X but X is false". NOT: regular bugs, performance issues, or user annoyance.

**Examples:**
- A table was schema-only for weeks. Claude several times promised "I'll look at what the bot sent to user X" without checking that the table was empty.
- Example from project: G2-E10-S9 (2026-05-28): `message_log` was schema-only for weeks. Claude promised data access without verifying.

**Severity:** ALWAYS treat as P0. Trust is harder to rebuild than to maintain.

---

## Sequence (proven on multiple trust events)

### 1. Immediately acknowledge honestly (text, not code)

Don't:
- "I didn't mean to"
- "That was a long time ago"
- "We'll fix it"

Do:
- Explicitly name the failure: "I misrepresented capability — table was empty, I wasn't checking, was saying data exists"
- Trace the chain: schema → no implementation → multiple sessions assumed it works
- Apologize without excuse

### 2. Read user's directives carefully

After trust event, user typically asks for one or more of:
- Full process (no shortcuts)
- Additional rule changes
- Specific verification gates
- Quality bar higher than usual

**Don't propose shortcuts.** Honour exact scope.

### 3. Incident entry in memory FIRST (before code changes)

Add entry to incident log with:
- Severity tag "TRUST EVENT"
- Root cause (multi-layered: schema-only, no validation, false claim)
- Impact (what was misled, for how long)
- Process changes triggered

### 4. Process changes commit BEFORE feature commit

Pattern: separate the "rule change" from the "feature fix":

```
commit1  process(G2-E10-S23): trust event response — 9-item rule changes
commit2  feat(G2-E10-S23): full audit implementation
commit3  fix(G2-E10-S23): close reviewer findings
```

Why: future contributors can see "we changed rules because X" independently from "we shipped feature Y". Process changes outlive features.

### 5. Full pipeline — no shortcuts

Even if scope seems clear:
- Discovery 3-cycle YAML interview (even if pre-fill 80%, ASK explicit questions)
- Planning + skeptic + completeness + architect review
- Worktree agent for code
- TWO reviewers (Sonnet code + Opus financial/data) — convergence pattern
- Multiple commit iterations if needed
- PROD smoke with EXPLICIT verification queries (not vague "check DB")

### 6. PROD smoke uses EXPLICIT verification queries

Don't: "verify it works"

Do:
- Predefine SQL queries / commands
- Run them after smoke
- Show user the actual output
- Map to user-spec acceptance criteria

Example:
```sql
SELECT id, user_id, direction, substr(text,1,50), context, created_at
FROM message_log
WHERE created_at >= datetime('now','-15 minutes')
ORDER BY id DESC LIMIT 25
```

User sees real data → trust visibly restored.

### 7. 3-round retro mandatory + explicit "trust restored?" question

Round 1: full analysis, action items
Round 2: verify fixes, look for NEW issues
Round 3: final clean OR another iteration

In Round 1 retro, scrum-master agent MUST ask:
- "Was trust restored? Evidence?"
- "Residual risks?"
- "Process change that would have prevented original event?"

### 8. Process changes ENFORCED in future

After playbook execution, expect rule changes like:
- Tighter hotfix threshold
- New audit gates in pipeline
- New reviewer agent type
- New mandatory verification step

These rules should land in CLAUDE.md "NEVER" section + corresponding skill/agent files.

---

## Lessons consolidated

| # | Pattern | Why critical |
|---|---|---|
| 1 | Honest acknowledgment without excuse | Foundation of recovery — user needs to see you don't dodge |
| 2 | Incident entry BEFORE code | Memory survives session compact; ensures future sessions know |
| 3 | Separate process commit from feature commit | Rule changes ≠ feature, both deserve own commit |
| 4 | Full pipeline no shortcuts | Trust isn't restored by speed, by THOROUGHNESS |
| 5 | Dual reviewer (Sonnet + Opus) | Convergence catches what single reviewer misses |
| 6 | Explicit verification queries | Vague "check it works" lets next trust event in |
| 7 | 3-round retro + "trust restored?" | Forces explicit answer, not assumption |
| 8 | Rule changes go to CLAUDE.md NEVER section | Make recurrence harder mechanically |

---

## When NOT to use this playbook

- Regular bugs (use `/write-code` or standard pipeline)
- Performance issues (use standard incident response)
- User reports something not working that **does** work — verify first; if not a real bug, no trust event

---

## Related

- Incident log (project memory) — including trust events
- CLAUDE.md "NEVER" section — mechanical rule enforcement
- `.claude/skills/scrum-master/SKILL.md` — 3-round retro process
