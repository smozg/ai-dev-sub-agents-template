# AI Dev Sub-Agents Template

A complete Claude Code workflow system for production-quality software development. Battle-tested on a real Telegram bot product across 10+ epics and 6+ months of continuous development.

This template provides 7 specialized sub-agents, 8 workflow skills, 7 slash commands, and a 19-step deploy + GTM pipeline — all ready to adapt to any project.

## What's Included

```
.claude/
├── agents/                     # 7 specialized sub-agents
│   ├── code-reviewer.md        # Architecture + sync + security review (Sonnet)
│   ├── completeness-validator.md # User-spec ↔ tech-spec traceability (Sonnet)
│   ├── deploy-agent.md         # scp + pycache + restart + verify (Sonnet)
│   ├── financial-reviewer.md   # Payment/billing/audit review (Opus)
│   ├── scrum-master-agent.md   # Post-epic retrospective (Opus)
│   ├── skeptic.md              # Plan hallucination catcher (Sonnet)
│   └── webdevops.md            # Website deploy + live verify (Sonnet)
├── skills/                     # 8 workflow skills (loaded by Claude on demand)
│   ├── code-reviewing/SKILL.md + ...
│   ├── deploy-pipeline/SKILL.md + references/
│   ├── development/SKILL.md + references/
│   ├── discovery/SKILL.md + references/
│   ├── marketing/SKILL.md + references/
│   ├── planning/SKILL.md + references/
│   ├── project-knowledge/SKILL.md + references/
│   └── scrum-master/SKILL.md + references/
└── commands/                   # 7 slash commands
    ├── /new-feature            # Discovery → Planning pipeline
    ├── /deploy                 # 19-step Deploy + GTM pipeline
    ├── /done                   # Retrospective + archive
    ├── /write-code             # Quick code task (bug fix, small feature)
    ├── /broadcast              # Marketing broadcast (3-gate + 3-step)
    ├── /scrum-master-weekly    # Weekly process retrospective
    └── /scrum-master-monthly   # Monthly process retrospective
docs/
├── playbooks/
│   └── trust-event-recovery.md # When Claude misrepresents a capability
└── sub-agents/                 # 8 prompt-ready sub-agent instructions
    ├── tst-stage.md / tst-prod.md / tst-site.md (deprecated → webdevops)
    ├── pm-stage.md / pm-prod.md / pm-review.md
    ├── ux-review.md
    └── marketing.md
```

## The Deploy Pipeline (19 Steps)

The core of the system. No feature ships without going through all 19 steps:

```
Phase 1 — DEV
  Step 1: Code + Tests (all pass required)

Phase 2 — STAGE
  Step 2: Deploy to STAGE (via deploy-agent)
  Step 3: Start parser (if applicable)
  Step 4: E2E Test (tst-stage agent, Sonnet)
  Step 5: PM Review (pm-stage agent, Opus)
  Step 6: UX Review (ux-review agent, Opus)
  Step 7: User Smoke Test ← COMPOUND GATE: 4+5+6 all pass

Phase 3 — PROD
  Step 8: Deploy to PROD (via deploy-agent)
  Step 9: Start parser (if applicable)
  Step 10: E2E Test (tst-prod agent, Sonnet)
  Step 11: PM Review (pm-prod agent, Opus)
  Step 12: UX Review (ux-review agent, Opus)
  Step 13: User Smoke Test ← COMPOUND GATE: 10+11+12 all pass

Phase 4 — Go To Market
  Step 14: Marketing Materials (marketing agent, Opus)
  Step 15: Website Update (webdevops agent, Sonnet)
  Step 16: Broadcast 3-Gate Review (PM → UX → Tester)
  Step 17: Broadcast 3-Step Send (STAGE preview → PROD preview → full send)
  Step 18: Talking Head + Distribution

Phase 5 — Close
  Step 19: Scrum Master Review (3 rounds, Opus)
```

Plus 5 conv_log audit gates (0a, 0b, 7.5, 13.5, 19.5) that enforce conversation logging discipline, tracked in `pipeline-state.yml`.

**Key principle:** Sub-agents verify BEFORE user smoke-tests. Never ask user to test until automated checks pass.

## The New Feature Pipeline

```
/new-feature "description"
  → Phase 0: Initialize (work dir, interview.yml)
  → Phase 1.5: PM Ideation (5 value ideas + hybrid, Opus)
  → Phase 1-3: 3-Cycle Interview (Business → UX/CJM → Technical)
      YAML scoring, 85% threshold, session-resumable
  → Phase 6: User-Spec creation + approval
  → Phase 1-3 Planning: Tech-Spec with validator agents:
      - Skeptic: verify all file/function claims actually exist
      - Completeness Validator: bidirectional user-spec ↔ tech-spec traceability
      - Architect Review: 5 code rules check
  → Phase 6: User approval → /deploy
```

## The Development Skill (TDD + Dual Review)

For code changes, the system enforces:

1. **Plan first** — show plan to user, wait for explicit "yes"
2. **TDD for core/** — write failing test → implement → pass
3. **Sync all frontends** — TG + MAX + CLI always together
4. **Dual reviewer**:
   - code-reviewer (Sonnet): architecture, sync, security
   - financial-reviewer (Opus): payment/billing/audit code (on demand)
5. **Convergence pattern** — if both reviewers find same issue = high confidence

## Trust Event Recovery

When Claude misrepresents a capability (says X is available but X isn't), the `docs/playbooks/trust-event-recovery.md` defines an 8-step recovery sequence:

1. Honest acknowledgment (name the failure explicitly)
2. Read user directives carefully (no shortcuts)
3. Incident entry in memory BEFORE code changes
4. Separate process commit from feature commit
5. Full pipeline, no shortcuts
6. Explicit verification queries in PROD smoke
7. 3-round retro + explicit "trust restored?" question
8. Rule changes go to CLAUDE.md NEVER section

## Quickstart for New Projects

### 1. Copy the structure

```bash
cp -r .claude/ your-project/.claude/
cp -r docs/ your-project/docs/
```

### 2. Customize configuration

Replace placeholders throughout:

| Placeholder | Replace with |
|---|---|
| `<PROJECT>` | Your project name |
| `<PROD_PROJECT>` | Production project variant name |
| `<SITE_DOMAIN>` | Your website domain |
| `$PROJECT_DIR` | Absolute path to your project |
| `<CODE_SERVER>` | Your development server IP |
| `<RUN_SERVER>` | Your production server IP |
| `<STAGE_PATH>` | STAGE environment path on Run server |
| `<PROD_PATH>` | PROD environment path on Run server |
| `<ADMIN_USER_ID>` | Your Telegram/bot admin user ID |
| `<TEST_USER_ID>` | Test user ID for E2E testing |

Key files to customize:
- `.claude/skills/deploy-pipeline/references/deploy-commands.md` — server IPs, paths, service names
- `.claude/skills/development/references/code-rules.md` — your 5 architecture invariants
- `docs/sub-agents/*.md` — update `{{SSH_COMMAND}}`, `{{CLI_COMMAND}}`, `{{DB_CHECK}}`, `{{MEMORY_PATH}}` placeholders

### 3. Set up CLAUDE.md

Reference the skills in your CLAUDE.md:

```markdown
## Commands
| /new-feature | New feature | Discovery → Planning |
| /deploy | Deploy + GTM | 19-step pipeline |
| /done | Close task | Scrum master review |
| /write-code | Quick task | Bug fix, small feature |
| /broadcast | Broadcast | 3-gate + 3-step send |
```

### 4. Set up conv_log (optional but recommended)

The system uses a conversation log database for retrospectives:

```bash
# scripts/conv_log.py logs to data/conversation.db
# Query: SELECT epic, role, type, content FROM messages WHERE epic LIKE 'G2%'
```

### 5. Launch via Claude Code Agent tool

```
Agent(
  prompt="You are a tester. Read your instructions from docs/sub-agents/tst-stage.md.
  Brief: {path to brief}. E2E plan: {steps from tech-spec}.
  SSH to <RUN_SERVER>, run project CLI, test the feature, check DB.
  Read 'Lessons Learned' section and add 'Applied lessons' table to report.",
  model="sonnet"
)
```

## Lessons Learned (from real project)

All "Lessons Learned" sections across agents contain examples from a real project. They explain WHY each rule exists. Examples use format "Example from project (G2-E8): ..." to distinguish illustrative examples from rules.

**Key lessons that shaped this system:**

| Problem | Rule Created | Where |
|---|---|---|
| 7 bugs shipped to PROD because PM/UX review skipped | Compound gates (Steps 7 and 13) require ALL 3 preceding sub-agents | deploy-pipeline/SKILL.md |
| Broadcast sent to PROD without STAGE preview | Mandatory 3-step send: STAGE preview → PROD preview → full send | Step 17 |
| Worktree agent missed 6 INSERT sites for audit column | Signature Drift Check: grep all callers, not just tech-spec list | code-reviewer.md, development/SKILL.md |
| Site deploy marked "PASS" but scp never reached Run | webdevops mandatory Step 5 + live URL verify via WebFetch | webdevops.md |
| Claude promised data that wasn't there | trust-event-recovery.md playbook | docs/playbooks/ |
| Conv_log entries fell below expected minimum | 5 audit gates in pipeline-state.yml | deploy-pipeline references |

## Philosophy

- **Sub-agents don't fix code** — they find and report. Orchestrator decides.
- **Clean context** — each agent starts fresh, simulating a real team member
- **Structured output** — standardized report formats for easy parsing
- **Model matching** — Sonnet for fast testing, Opus for nuanced PM/marketing/UX/financial work
- **Mandatory approval** — scrum master and code review findings require user sign-off before changes
- **Pipeline state = source of truth** — pipeline-state.yml is the only protection against skipped steps
- **Lessons are first-class** — every failure creates a lesson in the agent that should have caught it

## Related

- [ai-dev-template](https://github.com/SMozg/ai-dev-template) — CLAUDE.md and project infrastructure template
