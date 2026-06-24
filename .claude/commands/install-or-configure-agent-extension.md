---
name: install-or-configure-agent-extension
description: Workflow command scaffold for install-or-configure-agent-extension in hub-work.
allowed_tools: ["Bash", "Read", "Write", "Grep", "Glob"]
---

# /install-or-configure-agent-extension

Use this workflow when working on **install-or-configure-agent-extension** in `hub-work`.

## Goal

Installs or configures an agent extension or toolkit, including updating registry/configuration files and adding workflow scripts/templates.

## Common Files

- `.agents/skills/*/SKILL.md`
- `.claude/skills/*/SKILL.md`
- `.specify/extensions.yml`
- `.specify/extensions/*`
- `.specify/integration.json`
- `.specify/integrations/*.manifest.json`

## Suggested Sequence

1. Understand the current state and failure mode before editing.
2. Make the smallest coherent change that satisfies the workflow goal.
3. Run the most relevant verification for touched files.
4. Summarize what changed and what still needs review.

## Typical Commit Signals

- Add new skill definitions under agent skill directories (e.g., .agents/skills/speckit-*/SKILL.md, .claude/skills/gitnexus/*).
- Add or update configuration/manifest files (e.g., .specify/extensions.yml, .specify/integration.json, .specify/integrations/*.manifest.json).
- Add workflow scripts and templates (e.g., .specify/scripts/bash/*.sh, .specify/templates/*.md, .specify/workflows/*).
- Update documentation files (e.g., AGENTS.md, CLAUDE.md).

## Notes

- Treat this as a scaffold, not a hard-coded script.
- Update the command if the workflow evolves materially.