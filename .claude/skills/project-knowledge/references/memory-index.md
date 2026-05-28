# Memory File Index — Complete Mapping

<!-- 
CUSTOMIZE: Replace with your project's memory file structure.
Base path: <MEMORY_PATH>
-->

Base path: `<MEMORY_PATH>`

## Quick Reference

| Question | File | Key Section |
|----------|------|-------------|
| What is this project? | `<project>.md` | Lines 1-50: overview, TA, business model |
| What levels/features exist? | `<project>-cjm.md` | Full file: user journey with screens and callbacks |
| How does level progression work? | `<project>-pm.md` | "Level progression formula" section |
| What's the code structure? | `<project>-architecture.md` | Directory tree + key functions |
| What DB tables exist? | `<project>.md` | "Database tables" section: all tables with columns |
| How to deploy? | `<project>.md` | "Deploy rules" section: scp, restart, hardlink rules |
| What services run? | `<project>.md` | "Systemd services" section: all units |
| What features are done? | `<project>.md` | "Feature checklist" section |
| What's the rating formula? | `<project>-rating-formulas.md` | Full file |
| What incidents happened? | `<project>-inc-prob.md` | Full file: known bugs, workarounds |
| What tone should the bot use? | `<project>-pm.md` | "Tone and style" section |
| What feedback rules exist? | `feedback_plan_before_action.md` | Must-follow rules |

## File Sizes (for context budget planning)

| File | Size | Read Strategy |
|------|------|---------------|
| `<project>.md` | Large | Section-by-section (use line ranges) |
| `<project>-cjm.md` | Medium | Section-by-section (by level) |
| `<project>-pm.md` | Small | Can read fully |
| `<project>-architecture.md` | Small | Can read fully |
| `<project>-rating-formulas.md` | Small | Read fully |
| `<project>-inc-prob.md` | Small | Can read fully |

## Update Rules

1. **Ask before updating** — show diff to user, wait for "yes"
2. **One file per update** — don't batch across memory files
3. **Git after update:**
   ```bash
   cd <MEMORY_PATH> && git add . && git commit -m "update: what changed" && git push
   ```
4. **Don't update during active development** — update at phase boundaries (after deploy, after /done)
