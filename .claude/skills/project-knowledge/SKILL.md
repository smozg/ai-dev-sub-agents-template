---
name: project-knowledge
description: |
  Loads project context just-in-time from memory files. Instead of reading all memory files at session start,
  loads only the relevant file based on the current task.

  Use when: "project context", "what is this project", "architecture", "CJM", "customer journey",
  "product decisions", "incidents", "bugs", "rating formulas", "database schema", "deploy rules"
---

# Project Knowledge — Just-in-Time Context Loader

## When This Skill Triggers

Load project context **on demand** — not all at once. Read only the file relevant to your current task.

## Step 1: Determine What You Need

What are you working on right now? Match your task to the table below.

| Task Type | Memory File | What's Inside |
|-----------|-------------|---------------|
| **Any task start** | `MEMORY.md` | Index of all projects and rules |
| **UI, screens, buttons, journey** | `<project>-cjm.md` | Full Customer Journey Map, screens, callbacks |
| **Product decisions, tone, formulas** | `<project>-pm.md` | PM agreements, level progression formula, style rules |
| **Architecture, DB, tables, services** | `<project>.md` | Main project context, deploy rules, DB schema |
| **Code patterns, clean arch** | `<project>-architecture.md` | Directory structure, Action-Response pattern, key functions |
| **Incidents, bugs, support** | `<project>-inc-prob.md` | Known issues, workarounds |
| **Rating calculations, scoring** | `<project>-rating-formulas.md` | Traffic light thresholds, scoring weights |
| **Process rules, feedback** | `feedback_plan_before_action.md` | Must-follow rules from user feedback |

## Step 2: Load the File

All files are at: `<MEMORY_PATH>`

**Read only what you need.** For large files, read specific sections:

### Main project file — Section Guide
| Section | When |
|---------|------|
| Overview + Target Audience | First time context |
| Database tables | DB work |
| Deploy rules | Deployment |
| Systemd services | Infrastructure |
| Feature checklist | Status check |

### CJM file — Section Guide
| Section | When |
|---------|------|
| L1-L2 Onboarding | Onboarding changes |
| L3-LN Features | Feature-specific work |

## Step 3: After Work — Update Memory

After completing a task that changed architecture, CJM, or product decisions:
1. Identify which memory file needs updating
2. **Show the user** what will change
3. Wait for explicit approval ("yes")
4. Update the file
5. Commit: `cd <MEMORY_PATH> && git add . && git commit -m "update: description" && git push`

## Rules

- **Never read all files at once** — wastes context window
- **Always read MEMORY.md at session start** — it's the index
- **Read docs/BACKLOG.md** at session start — it tells you what to work on
- Large files: read the relevant section, not the whole file
- When in doubt, check [memory-index.md](references/memory-index.md) for the full mapping
