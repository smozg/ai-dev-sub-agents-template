---
name: discovery
description: |
  Conducts a 3-cycle adaptive interview to deeply understand a feature before any planning or coding.
  Creates a user-spec (brief) with validated requirements, edge cases, and success metrics.
  Uses YAML scoring (85% threshold) to prevent premature transition to planning.

  Use when: "new feature", "start feature", "discovery", "interview",
  "/new-feature", "user spec", "brief"
---

# Discovery — 3-Cycle Adaptive Interview

## Overview

Discovery is the first step of any feature. It ensures we deeply understand WHAT we're building and WHY before writing a single line of code. The interview has 3 cycles with a YAML-tracked scoring system.

**Rule:** Do NOT proceed to planning until all required fields score ≥ 85%.

## Phase 0: Initialize

1. Determine feature name from user's input (or ask)
2. Determine epic ID from `docs/BACKLOG.md` (or create new)
3. Create work directory:
   ```bash
   mkdir -p work/{feature-name}
   ```
4. Copy interview template:
   ```bash
   cp .claude/skills/discovery/references/interview-template.yml work/{feature-name}/interview.yml
   ```
5. Set `interview_metadata.started`, `status: in_progress`
6. Log to conv_log:
   ```bash
   cd $PROJECT_DIR && uv run python scripts/conv_log.py {EPIC} claude phase "discovery started: {feature-name}"
   ```

## Phase 1: Load Context

Before asking questions, understand the project:

1. Read `docs/BACKLOG.md` — what's already done, what's planned
2. Read relevant memory file (use [memory-index.md](../project-knowledge/references/memory-index.md)):
   - For new level/feature: read CJM file (existing levels) + PM decisions file
   - For infrastructure: read architecture file
   - For parser work: read data/schema file
3. Scan codebase if needed: `src/<project>/core/flows.py` for existing features

## Phase 1.5: PM-Ideation (value generation)

**Before** the interview — PM-agent helps think about value.

Launch PM-ideation agent:
```
Agent(
  prompt="You are a product strategist for <PROJECT> (<brief project description>).

  Read:
  - Memory files: CJM, PM decisions, BACKLOG
  
  User wants to build: {feature description from user}

  Execute 3 steps:

  ### 1. Emotional user portrait for this level/feature
  - Primary fear (what prevents buying/using?)
  - Primary desire (what do they want to get?)
  - Hidden tension (what they don't say out loud?)
  - Trust trigger / rejection trigger
  - Path: BEFORE contact with feature → AFTER

  ### 2. 5 value ideas
  For each idea:
  - Name (2-4 words)
  - Essence (1 sentence)
  - Mechanics (how it works in the product)
  - Why it works (what pain it addresses)
  - Strength estimate (1-10)

  ### 3. Critique + best hybrid
  For each idea: strength, weakness.
  Choose best idea and best hybrid (combination of two).",
  model="opus"
)
```

**Result:** Show user 5 ideas + hybrid. Discuss. Best ideas → input for YAML interview.

## Phase 2: Initial Scoring

After PM-ideation discussion, user describes the feature (free-form or building on PM ideas):

1. Read the [scoring-guide.md](references/scoring-guide.md)
2. Score ALL 12 fields based on what user mentioned:
   - Directly addressed with detail → 80-95%
   - Mentioned briefly → 50-70%
   - Implied but not stated → 20-40%
   - Not mentioned → 0%
3. Update `work/{feature}/interview.yml` with scores and values
4. Show the user a summary: "Here's what I understood, here are the gaps"

## Phase 3: Cycle 1 — Business

**Scope:** `cycle1_business` fields: feature_name, pain, success_metric, what_if_not, target_users

**Process:**
1. Find required fields with score < 85%
2. Ask **3-4 questions per batch** about different gaps
3. **Propose solutions** based on project knowledge — don't just ask questions
4. After user answers: update scores, values, gaps in interview.yml
5. Repeat until all required fields ≥ 85% (or 3 batches, then ask "enough?")
6. Mark `progress.cycle1_complete: true`

**Key questions:**
- "What pain do existing features NOT solve?"
- "How will the customer know the feature works? What specific action?"
- "What happens if we DON'T build this? Customer leaves? Won't buy?"

## Phase 4: Cycle 2 — UX + CJM

**Scope:** `cycle2_ux_cjm` fields: cjm_step, affected_screens, user_scenarios, edge_cases

**Process:** Same as Cycle 1 but with UX focus.

**Key questions:**
- "After what command/level does this appear in the journey?"
- "Does the user see this as a new command or part of existing feature?"
- "Walk the path: user presses [what?] → sees [what?] → does [what?]"
- "What if user has 0 data? No competitors? Already at maximum level?"

## Phase 5: Cycle 3 — Technical + Cleanup

**Scope:** `cycle3_technical` fields: data_source, dependencies, risks, frontend_sync
**Also:** cleanup pass on ALL fields still below 85%

**Key questions:**
- "Does the parser already collect the needed data or do we need new fields?"
- "Which DB tables are affected? Is migration needed?"
- "What can change externally — how do we handle missing data?"
- "Are there differences between TG/MAX/CLI for this feature?"

## Phase 6: Create User Spec

After all 3 cycles complete (all required ≥ 85%):

Create `work/{feature}/user-spec.md`:

```markdown
# User Spec: {feature_name}

Epic: {epic}
Created: {date}
Status: draft

## What we're building
{feature_name} — {pain} → {success_metric}

## For whom
{target_users}

## Why now
{what_if_not}

## CJM
- Step: {cjm_step}
- Screens: {affected_screens}

## Usage scenarios
{user_scenarios}

## Edge cases
{edge_cases}

## Technical requirements
- Data: {data_source}
- Dependencies: {dependencies}
- Risks: {risks}
- Frontend sync (TG/MAX/CLI): {frontend_sync}

## Success metric
{success_metric}
```

Show user-spec to user for approval. Wait for explicit "ok" / "yes". Set status to `approved`.

## Phase 7: Handoff to Planning

After user approves user-spec:

1. Log to conv_log:
   ```bash
   cd $PROJECT_DIR && uv run python scripts/conv_log.py {EPIC} claude decision "user-spec approved, moving to planning"
   ```
2. Inform user: "Discovery complete. User-spec approved. Next step: planning."
3. If part of `/new-feature` command — automatically load the `planning` skill

## Resume Protocol

If `interview.yml` exists with `status: in_progress`:
1. Read interview.yml
2. Show conversation_history summary to user
3. Show current scores
4. Resume from current cycle and gaps

## Rules

- **NEVER skip a cycle.** Even if user says "just build it" — ask at least the required questions
- **NEVER proceed to planning with any required field < 85%**
- **Propose, don't just ask.** Use project knowledge to suggest answers
- **3-4 questions per batch.** Not 1, not 12.
- **Save interview.yml after every user response** — enables session recovery
- **Log phase transitions** to conv_log
