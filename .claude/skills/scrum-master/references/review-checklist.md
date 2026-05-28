# Scrum Master Review Checklist

## 1. CLAUDE.md — Quality

- [ ] No garbage or irrelevant content?
- [ ] No contradictions between sections?
- [ ] No confusing or ambiguous information?
- [ ] Commands table matches actual `.claude/commands/` files?
- [ ] Code rules still accurate?
- [ ] Infrastructure table up to date (IPs, paths)?

## 2. CLAUDE.md — Integrity

- [ ] All file/doc references point to existing files?
- [ ] CLAUDE.md readable in <30 seconds (minimal section)?
- [ ] A new Claude session will understand everything?
- [ ] Legacy sections still needed or can be removed?

## 3. Memory Files

Path: `<MEMORY_PATH>`

For each project memory file:
- [ ] All information current and accurate?
- [ ] No stale claims about code behavior?
- [ ] No references to deleted/renamed files?
- [ ] No contradictions with current codebase?

Key files to verify:
- Main project file — feature checklist matches reality?
- CJM file — all levels/features described? New features included?
- PM decisions file — product decisions still valid?
- Architecture file — code structure matches reality?

## 4. Skills and Agents

For each `.claude/skills/*/SKILL.md`:
- [ ] Instructions still match the current process?
- [ ] Referenced files/paths exist?
- [ ] No outdated examples?

For each `.claude/agents/*.md`:
- [ ] Agent instructions accurate?
- [ ] Codebase paths correct?

## 5. Templates

- `docs/templates/user-spec.md.template` — covers current discovery process?
- `docs/templates/task.md.template` — fields match current workflow?
- Plan template — still relevant alongside tech-spec template?

## 6. Sub-agents

For each `docs/sub-agents/*.md`:
- [ ] SSH commands correct?
- [ ] CLI commands correct?
- [ ] DB query examples valid?
- [ ] Checklist items relevant?

## 7. Process

- [ ] Completed task reflected in `docs/BACKLOG.md`?
- [ ] `docs/TODO.md` cleaned up (no stale items)?
- [ ] Were there problems this cycle? → Document the lesson
- [ ] 1-3 improvements identified:

| Problem | Solution | Where to Document |
|---------|----------|-------------------|
| {what went wrong} | {how to prevent} | {which file to update} |

## 8. Conversation Log

- [ ] Key decisions logged in conversation DB?
- [ ] Phase transitions logged?
- [ ] No gaps in critical decision trail?

## 9. Retrospective (after epic completion)

- [ ] Conv_log read and analyzed?
- [ ] User messages classified (CORRECTION / FEEDBACK / APPROVAL)?
- [ ] Comparison table built (process / fact / reminder / improvement)?
- [ ] Action items identified and added to skills/templates?
- [ ] User approved changes?

## 10. Lessons Learned (mandatory at each retro)

Update the "Lessons Learned" section in each touched agent/sub-agent:

- [ ] `docs/sub-agents/tst-stage.md` — new bugs missed by tester?
- [ ] `docs/sub-agents/tst-prod.md` — PROD-specific bugs?
- [ ] `docs/sub-agents/pm-stage.md` — what PM missed?
- [ ] `docs/sub-agents/pm-prod.md` — PROD PM findings?
- [ ] `docs/sub-agents/pm-review.md` — text review: what was missed?
- [ ] `docs/sub-agents/ux-review.md` — UX review: style, tone, emoji?
- [ ] `docs/sub-agents/marketing.md` — marketing: what worked/didn't?
- [ ] `.claude/agents/code-reviewer.md` — code review: new patterns?
- [ ] `.claude/agents/skeptic.md` — planning: what was wrong?
- [ ] `.claude/agents/completeness-validator.md` — coverage: what was missed?

Entry format: `- **{EPIC}:** {what happened}. **Lesson:** {what to do differently}.`
