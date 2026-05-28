---
name: marketing
description: |
  Marketing sub-agent instructions and reference materials.
  GTM execution is now part of the unified deploy-pipeline (steps 14-18).

  Use when: "marketing materials", "positioning", "marketing brief",
  "sales scripts", "client materials"
---

# Marketing — Reference Skill

## Overview

GTM (Go To Market) is now part of the unified **deploy-pipeline** (steps 14-18).
This skill contains reference materials for the marketing sub-agent.

**For the full GTM process** → see `.claude/skills/deploy-pipeline/SKILL.md` steps 14-18.

## Marketing Sub-agent

The marketing sub-agent (`docs/sub-agents/marketing.md`) is launched at step 14 of the deploy pipeline.

### What it produces:
- Positioning (one sentence)
- For trainees/clients (assignment + how to explain)
- For sales team (selling argument)
- For website (what block to add)
- Conversion to next level
- Draft broadcast message
- Talking head script (draft)

### Output: `docs/marketing/<epic>.md`

## References

- [broadcast-guide.md](references/broadcast-guide.md) — broadcast.py usage, flags, personalization (also in deploy-pipeline/references/)
- `docs/sub-agents/marketing.md` — full sub-agent instructions
