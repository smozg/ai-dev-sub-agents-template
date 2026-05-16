# AI Dev Sub-Agents Template

Template sub-agent prompts for use with **Claude Code Agent tool**. Each sub-agent is a specialized role that runs independently during the development lifecycle.

## Sub-Agents

| Agent | Model | When | Purpose |
|-------|-------|------|---------|
| **Tester (STAGE)** | Sonnet | After deploy to STAGE | E2E testing via CLI, DB verification |
| **Tester (PROD)** | Sonnet | After deploy to PROD | Same checks on production |
| **PM Reviewer (STAGE)** | Opus | After STAGE tests pass | CJM review, text/UX quality, ideas |
| **PM Reviewer (PROD)** | Opus | After PROD tests pass | Final CJM pass on production |
| **Marketing** | Opus | Start & end of each epic | Positioning, sales scripts, site updates |

## How It Works

```
Code ready → Deploy to STAGE
  → Agent: tst-stage.md (E2E test)
  → Agent: pm-stage.md (UX review)
  → User: smoke test
Deploy to PROD
  → Agent: tst-prod.md (E2E test)
  → Agent: pm-prod.md (UX review)
  → User: smoke test
  → Agent: marketing.md (site + materials)
```

## Usage with Claude Code

1. Copy the `sub-agents/` directory into your project's `docs/` folder
2. Customize the templates:
   - Replace SSH/CLI commands with your project's access methods
   - Update memory file paths to match your project structure
   - Adjust checklists to your product's needs
3. Reference them in your CLAUDE.md deploy process
4. Launch via Claude Code's Agent tool:

```
Agent(prompt="Run E2E test on STAGE. Brief: docs/briefs/my-feature.md. E2E plan: [steps]. Use docs/sub-agents/tst-stage.md as your instructions.", model="sonnet")
```

## Customization Points

Each template has `{{PLACEHOLDER}}` sections to fill in:

- `{{SSH_COMMAND}}` — how to connect to your server
- `{{CLI_COMMAND}}` — how to run your test CLI
- `{{DB_CHECK}}` — how to query your database
- `{{MEMORY_PATH}}` — path to your project's memory files
- `{{FINDINGS_PATH}}` — where to write findings

## Template Structure

```
sub-agents/
  tst-stage.md    — Tester for staging environment
  tst-prod.md     — Tester for production environment
  pm-stage.md     — PM reviewer for staging
  pm-prod.md      — PM reviewer for production
  marketing.md    — Marketing strategist
```

## Philosophy

- **Sub-agents don't fix code** — they find and report issues
- **Clean context** — each agent starts fresh, simulating a real team member
- **Structured output** — standardized report formats for easy parsing
- **Model matching** — Sonnet for fast testing, Opus for nuanced PM/marketing work

## Related

- [ai-dev-template](https://github.com/SMozg/ai-dev-template) — CLAUDE.md and project infrastructure template
