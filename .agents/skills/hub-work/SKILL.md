```markdown
# hub-work Development Patterns

> Auto-generated skill from repository analysis

## Overview

This skill provides guidance on the core development patterns and workflows used in the `hub-work` JavaScript repository. It covers coding conventions, file organization, commit practices, and step-by-step instructions for adding new agent skills and extensions. The goal is to ensure consistency and efficiency when contributing to or extending the codebase.

---

## Coding Conventions

### File Naming

- Use **camelCase** for JavaScript files.
  - Example: `userProfile.js`, `dataFetcher.js`

### Import Style

- Use **alias imports** to reference modules.
  - Example:
    ```javascript
    import { fetchData } from '@utils/dataFetcher';
    ```

### Export Style

- Use **named exports** for modules.
  - Example:
    ```javascript
    // In dataFetcher.js
    export function fetchData() { ... }
    export const API_URL = '...';
    ```

### Commit Messages

- Follow **conventional commit** format.
  - Prefixes: `chore`, `feat`, `docs`
  - Example: `feat: add user profile fetch logic`

---

## Workflows

### Add Skill to Agent

**Trigger:** When introducing a new skill or capability for an agent (e.g., dashboard, design system, ui-ux-pro-max).  
**Command:** `/add-skill`

1. **Create a Skill Directory**
   - Under one or more of the following:
     - `.agent/skills/<skill-name>/`
     - `.agents/skills/<skill-name>/`
     - `.claude/skills/<skill-name>/`
2. **Add Documentation**
   - Create a `SKILL.md` file describing the skill.
   - Optionally add `DESIGN.md` or other supporting docs.
3. **Add Supporting Scripts and Data**
   - Place scripts in `scripts/` (e.g., `scripts/init.js`, `scripts/setup.sh`).
   - Place data files in `data/` (e.g., `data/sample.csv`).
4. **Replicate as Needed**
   - If the skill is relevant to multiple agents, replicate the directory structure under each agent's skills folder.

**Example Directory Structure:**
```
.agents/skills/dashboard/
  ├── SKILL.md
  ├── DESIGN.md
  ├── scripts/
  │   └── init.js
  └── data/
      └── sample.csv
```

---

### Install or Configure Agent Extension

**Trigger:** When enabling a new agent feature set or integrating a toolkit (e.g., spec-kit, GitNexus).  
**Command:** `/install-extension`

1. **Add Skill Definitions**
   - Place new skill folders and `SKILL.md` under agent skill directories:
     - `.agents/skills/<extension-name>/`
     - `.claude/skills/<extension-name>/`
2. **Update Configuration/Manifest Files**
   - Edit or create:
     - `.specify/extensions.yml`
     - `.specify/integration.json`
     - `.specify/integrations/*.manifest.json`
3. **Add Workflow Scripts and Templates**
   - Place scripts in `.specify/scripts/bash/` (e.g., `setup.sh`)
   - Add templates in `.specify/templates/` (e.g., `README-template.md`)
   - Add or update workflows in `.specify/workflows/`
4. **Update Documentation**
   - Edit `AGENTS.md` and `CLAUDE.md` as needed to reflect new extensions.

**Example:**
```yaml
# .specify/extensions.yml
- name: spec-kit
  enabled: true
```

---

## Testing Patterns

- **Test File Naming:** Use `*.test.*` pattern (e.g., `userProfile.test.js`).
- **Testing Framework:** Not explicitly detected; follow standard JavaScript testing practices.
- **Example:**
  ```javascript
  // userProfile.test.js
  import { getUserProfile } from './userProfile';

  test('should fetch user profile', () => {
    expect(getUserProfile(1)).toEqual({ id: 1, name: 'Alice' });
  });
  ```

---

## Commands

| Command         | Purpose                                                      |
|-----------------|--------------------------------------------------------------|
| /add-skill      | Add a new skill definition, documentation, and scripts/data. |
| /install-extension | Install or configure an agent extension/toolkit.           |

---
```