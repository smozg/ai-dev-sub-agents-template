# AI Dev Sub-Agents Template

Template sub-agent prompts for use with **Claude Code Agent tool**. Each sub-agent is a specialized role that runs independently during the development lifecycle.

## Sub-Agents

| Agent | File | Model | When | Purpose |
|-------|------|-------|------|---------|
| **Tester (STAGE)** | `tst-stage.md` | Sonnet | After deploy to STAGE | E2E testing via CLI, DB verification |
| **Tester (PROD)** | `tst-prod.md` | Sonnet | After deploy to PROD | Same checks on production |
| **PM Reviewer (STAGE)** | `pm-stage.md` | Opus | After STAGE tests pass | CJM review, text/UX quality, ideas |
| **PM Reviewer (PROD)** | `pm-prod.md` | Opus | After PROD tests pass | Final CJM pass on production |
| **PM Reviewer (Broadcast)** | `pm-review.md` | Opus | Before broadcast/notification | Copy quality, all scenarios, tone |
| **UX Reviewer (Broadcast)** | `ux-review.md` | Opus | Before broadcast/notification | Style guide compliance, visual consistency |
| **Site Verifier** | `tst-site.md` | Sonnet | After site update (S5b) | Bundle verification, regression check |
| **Marketing** | `marketing.md` | Opus | Start & end of each epic | Positioning, sales scripts, site updates |

## Deploy Flow

```
Code ready → Deploy to STAGE
  → Agent: tst-stage.md (E2E test)
  → Agent: pm-stage.md (UX review)
  → User: smoke test
Deploy to PROD
  → Agent: tst-prod.md (E2E test)
  → Agent: pm-prod.md (UX review)
  → User: smoke test
```

## S5 Marketing Story Flow

Each marketing story (S5) consists of five steps:

| Step | What | Who |
|------|------|-----|
| **S5a** | Marketing materials (positioning, scripts, ideas) | Agent: `marketing.md` |
| **S5b** | Website update (data + build + deploy + verify) | Developer + Agent: `tst-site.md` |
| **S5c** | Broadcast to users (3-gate review + send) | See "3-Gate Broadcast" below |
| **S5d** | "Talking head" script (30-sec video text) | Agent: `marketing.md` |
| **S5e** | Distribution (trainees + sales team) | User |

### 3-Gate Broadcast Review (S5c)

Before any broadcast reaches users, it passes through three independent review gates:

```
Draft message
  → Gate 1: Agent pm-review.md (copy, scenarios, tone)
  → Gate 2: Agent ux-review.md (style guide, visual consistency)
  → Gate 3: Agent tst-stage.md (deploy, buttons work, callbacks)
  → Preview to owner (--preview flag)
  → User approval
  → Production broadcast
```

Each gate agent works independently with a clean context. If any gate finds issues, fix and re-run that gate.

## Iterative Scrum Master Review

After completing an epic, run up to 4 rounds of scrum master review:

1. Review documentation (CLAUDE.md, memory files, templates) against checklist
2. **Show findings to user — mandatory approval** before fixing anything
3. Fix only approved findings
4. Re-review — verify fixes + look for new issues
5. Repeat until 0 findings or 4 rounds

**Rule:** The scrum master finds problems. The user decides what to fix.

### Scrum Master Checklist

**Documentation quality:**
- [ ] No garbage or outdated info?
- [ ] No contradictions?
- [ ] All file/doc references valid?

**Memory files:**
- [ ] All entries current and accurate?
- [ ] No stale claims about code behavior?

**Templates:**
- [ ] Cover the current process?
- [ ] Examples up to date?

**Process:**
- [ ] Completed task reflected in docs?
- [ ] Problems from this cycle documented?
- [ ] 1-3 improvements identified?

## Usage with Claude Code

1. Copy the `sub-agents/` directory into your project's `docs/` folder
2. Customize the templates:
   - Replace `{{PLACEHOLDER}}` values with your project's specifics
   - Update checklists to match your product's needs
3. Reference them in your CLAUDE.md deploy process
4. Launch via Claude Code's Agent tool:

```
Agent(prompt="Run E2E test on STAGE. Brief: docs/briefs/my-feature.md. E2E plan: [steps]. Use docs/sub-agents/tst-stage.md as your instructions.", model="sonnet")
```

## Customization Points

Each template has `{{PLACEHOLDER}}` sections to fill in:

| Placeholder | Description |
|-------------|-------------|
| `{{SSH_COMMAND}}` | How to connect to your server |
| `{{CLI_COMMAND}}` | How to run your test CLI |
| `{{DB_CHECK}}` | How to query your database |
| `{{MEMORY_PATH}}` | Path to your project's memory files |
| `{{STYLE_GUIDE}}` | Path to your style guide |
| `{{TEXTS_FILE}}` | Path to your texts/copy file |
| `{{WEBSITE_URL}}` | Your project's website URL |
| `{{WEBSITE_DIST}}` | Path to built website bundle |
| `{{WEBSITE_SRC}}` | Path to website source data |

## Template Structure

```
sub-agents/
  tst-stage.md    — Tester for staging environment
  tst-prod.md     — Tester for production environment
  tst-site.md     — Site/website verifier
  pm-stage.md     — PM reviewer for staging (CJM walk-through)
  pm-prod.md      — PM reviewer for production
  pm-review.md    — PM reviewer for broadcast/notifications
  ux-review.md    — UX reviewer for broadcast/notifications
  marketing.md    — Marketing strategist
```

## Philosophy

- **Sub-agents don't fix code** — they find and report issues
- **Clean context** — each agent starts fresh, simulating a real team member
- **Structured output** — standardized report formats for easy parsing
- **Model matching** — Sonnet for fast testing, Opus for nuanced PM/marketing/UX work
- **Mandatory approval** — scrum master findings require user sign-off before changes

## Related

- [ai-dev-template](https://github.com/SMozg/ai-dev-template) — CLAUDE.md and project infrastructure template
