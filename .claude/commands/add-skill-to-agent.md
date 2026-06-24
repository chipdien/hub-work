---
name: add-skill-to-agent
description: Workflow command scaffold for add-skill-to-agent in hub-work.
allowed_tools: ["Bash", "Read", "Write", "Grep", "Glob"]
---

# /add-skill-to-agent

Use this workflow when working on **add-skill-to-agent** in `hub-work`.

## Goal

Adds a new skill definition for an agent, including documentation and supporting scripts/data.

## Common Files

- `.agent/skills/*/SKILL.md`
- `.agents/skills/*/SKILL.md`
- `.claude/skills/*/SKILL.md`
- `.agent/skills/*/scripts/*`
- `.agent/skills/*/data/*`
- `.agents/skills/*/scripts/*`

## Suggested Sequence

1. Understand the current state and failure mode before editing.
2. Make the smallest coherent change that satisfies the workflow goal.
3. Run the most relevant verification for touched files.
4. Summarize what changed and what still needs review.

## Typical Commit Signals

- Create a new directory under .agent/skills/, .agents/skills/, or .claude/skills/ with the skill name.
- Add SKILL.md (skill definition) and optionally DESIGN.md or other documentation.
- Add supporting scripts (e.g., scripts/*.js, scripts/*.sh, scripts/*.py) and data files (e.g., data/*.csv) as needed.
- If relevant, replicate the skill under multiple agent directories (e.g., .agent/skills/, .agents/skills/, .claude/skills/).

## Notes

- Treat this as a scaffold, not a hard-coded script.
- Update the command if the workflow evolves materially.