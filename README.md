# /skill-creator — Claude Code Skill Design Guide

A Claude Code skill for creating and optimizing other Claude Code skills. Provides a 6-step creation process, bundled scaffolding scripts, and optimization principles for building production-quality skills.

## Usage

```
/skill-creator
```

Then describe the skill you want to build. The skill walks you through understanding, planning, initializing, editing, packaging, and iterating.

## What's Included

```
skill-creator/
├── SKILL.md              ← Lean router: creation process + optimization principles
├── references/
│   └── skill-anatomy.md  ← Background: skill structure, bundled resources, progressive disclosure
├── scripts/
│   ├── init_skill.py     ← Scaffold a new skill directory
│   ├── package_skill.py  ← Validate and package a skill as .zip
│   └── quick_validate.py ← Fast validation of skill structure
└── LICENSE.txt
```

## Key Improvements Over the Original

This is a fork of Anthropic's official `skill-creator` with the following upgrades:

| Area | Before | After |
|------|--------|-------|
| **Context cost** | 263 lines (~4,000 tokens) loaded every invocation, including 75 lines of background material rarely needed | 147 lines (~1,800 tokens) — background extracted to `references/skill-anatomy.md`, loaded on demand (**-56%**) |
| **Quick Reference** | None — had to read 200+ lines to understand the process | Table at the top with all 6 steps, purpose, and skip conditions |
| **Script paths** | Relative (`scripts/init_skill.py`) — LLM had to guess absolute path | Absolute (`~/.claude/skills/skill-creator/scripts/init_skill.py`) — copy-paste ready |
| **Name validation** | `init_skill.py` accepted invalid names (spaces, uppercase) — created skills that immediately failed validation | Validates before creating: regex, hyphen boundaries, length. Provides a suggested fix |
| **Optimization Principles** | Not present | 5 numbered principles + a pre-ship checklist, derived from production optimization of [win4r/dev-skill](https://github.com/win4r/dev-skill) |
| **Self-consistency** | Violated its own best practices (description over 120 chars, no quick reference table) | Follows all of its own Optimization Principles |

## Skill Creation Process

| Step | What | Output |
|------|------|--------|
| 1. Understand | Gather concrete usage examples | Clear functionality scope |
| 2. Plan | Identify reusable scripts, references, assets | Resource list |
| 3. Initialize | Run `scripts/init_skill.py` | Skill directory with template |
| 4. Edit | Write SKILL.md + bundled resources | Working skill |
| 5. Package | Run `scripts/package_skill.py` | Distributable .zip |
| 6. Iterate | Test on real tasks, refine | Polished skill |

## Optimization Principles

The skill includes an **Optimization Principles** section derived from a thorough optimization pass on [win4r/dev-skill](https://github.com/win4r/dev-skill). These are reusable design patterns for building high-quality skills:

### Information Architecture
Position behavioral rules **before** reference material in SKILL.md. LLMs weight earlier content more heavily — rules buried at the bottom get forgotten.

### The Router Pattern
When a skill has 3+ code paths and exceeds ~400 lines, split into a router (SKILL.md) + per-path files loaded on demand. Reduces initial context by 40-70%.

```
skill-name/
├── SKILL.md              ← Router: classification + shared rules
└── workflows/
    ├── path-a.md         ← Loaded via Read tool after routing
    └── path-b.md
```

### Classification Design
For skills that classify user input: define a strict **priority ordering** (resolves multi-match) and **disambiguation rules** (resolves false positives). Test with 20-30 adversarial inputs.

### Inline References
When a rule must be applied in a different section, add a reminder at the point of application. Don't rely on the LLM to cross-reference across hundreds of lines.

### Prompt Engineering Heuristics
- Keep frontmatter `description` under 120 characters
- Use tables for routing/lookup (O(1) for LLMs)
- Explicit "Do NOT" prohibitions in high-stakes situations
- Categorize dependencies as optional vs. required for error handling

## Scaffolding Scripts

### Initialize a new skill

```bash
python scripts/init_skill.py my-skill --path ~/.claude/skills
```

Creates `~/.claude/skills/my-skill/` with a SKILL.md template and example `scripts/`, `references/`, `assets/` directories.

### Validate a skill

```bash
python scripts/quick_validate.py path/to/skill-folder
```

Checks frontmatter, naming conventions, directory structure, and description quality.

### Package for distribution

```bash
python scripts/package_skill.py path/to/skill-folder
```

Validates first, then creates a `.zip` file ready for sharing.

## Installation

```bash
git clone https://github.com/win4r/skill-creator.git ~/.claude/skills/skill-creator
```

## Prerequisites

- [Claude Code](https://claude.ai/code)
- Python 3.8+ (for scaffolding scripts)

## License

See [LICENSE.txt](LICENSE.txt).
