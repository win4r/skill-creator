---
name: skill-creator
description: "Create or optimize Claude Code skills with best-practice structure, bundled resources, and prompt engineering patterns."
license: Complete terms in LICENSE.txt
---

# Skill Creator

Create effective Claude Code skills by following the process below. For background on skill anatomy and bundled resources, read `references/skill-anatomy.md`.

## Quick Reference

| Step | What | When to Skip |
|------|------|-------------|
| 1. Understand | Gather concrete usage examples | Usage patterns already clear |
| 2. Plan | Identify reusable scripts, references, assets | Simple skill with no resources |
| 3. Initialize | Run `init_skill.py` to scaffold | Skill already exists |
| 4. Edit | Write SKILL.md + bundled resources | Never — always needed |
| 5. Package | Validate and zip for distribution | Skill is local-only |
| 6. Iterate | Test on real tasks, refine | Never — always valuable |

After creating or editing a skill, apply the **Optimization Principles** below to ensure quality.

---

## Skill Creation Process

### Step 1: Understanding the Skill with Concrete Examples

Clearly understand concrete examples of how the skill will be used. This understanding can come from either direct user examples or generated examples validated with user feedback.

For example, when building an image-editor skill, relevant questions include:

- "What functionality should the image-editor skill support?"
- "Can you give some examples of how this skill would be used?"
- "What would a user say that should trigger this skill?"

Start with the most important questions and follow up as needed. Conclude when there is a clear sense of the functionality the skill should support.

### Step 2: Planning the Reusable Skill Contents

Analyze each concrete example by:

1. Considering how to execute on the example from scratch
2. Identifying what scripts, references, and assets would be helpful when executing these workflows repeatedly

For guidance on when to use `scripts/`, `references/`, and `assets/`, read `references/skill-anatomy.md`.

### Step 3: Initializing the Skill

When creating a new skill from scratch, run the init script:

```bash
python ~/.claude/skills/skill-creator/scripts/init_skill.py <skill-name> --path <output-directory>
```

The script creates the skill directory with a SKILL.md template, proper frontmatter, and example `scripts/`, `references/`, and `assets/` directories. After initialization, customize or remove the generated example files.

Skip this step if the skill already exists and iteration or packaging is needed.

### Step 4: Edit the Skill

The skill is being created for another instance of Claude to use. Focus on information that would be beneficial and non-obvious.

#### Start with Reusable Skill Contents

Implement the `scripts/`, `references/`, and `assets/` files identified in Step 2. This may require user input (e.g., brand assets, API documentation). Delete any example files not needed.

#### Write SKILL.md

**Writing Style:** Use **imperative/infinitive form** (verb-first instructions), not second person. Use objective, instructional language (e.g., "To accomplish X, do Y" rather than "You should do X").

Answer these questions in SKILL.md:

1. What is the purpose of the skill, in a few sentences?
2. When should the skill be used?
3. How should Claude use it? Reference all bundled resources so Claude knows they exist.

### Step 5: Packaging a Skill

Package into a distributable zip:

```bash
python ~/.claude/skills/skill-creator/scripts/package_skill.py <path/to/skill-folder>
```

The script validates (frontmatter, naming, structure, description quality) then packages if validation passes. Fix any errors and re-run if validation fails.

### Step 6: Iterate

**Iteration workflow:**
1. Use the skill on real tasks
2. Notice struggles or inefficiencies
3. Identify how SKILL.md or bundled resources should be updated
4. Implement changes and test again

---

## Optimization Principles

Apply these when creating non-trivial skills or iterating on existing ones. These patterns are derived from production skill optimization and directly improve LLM behavior.

### 1. Information Architecture: Position for LLM Attention

An LLM reads SKILL.md sequentially and weights earlier content more heavily:

- **Rules and constraints before reference material.** Behavioral rules must appear before the content they constrain. If workflows come first and rules come last, the LLM may forget the rules by execution time.
- **Front-load what's needed every invocation.** Classification, mode selection, and global rules appear early. Conditional content can appear later or in separate files.
- **Move reference material to `references/`.** Background information (anatomy, schemas, examples) that isn't needed on every invocation should be in reference files, not in SKILL.md.

### 2. The Router Pattern: Modular Skills

When a skill has 3+ distinct code paths and exceeds ~400 lines, split into a router + per-path files:

```
skill-name/
├── SKILL.md              ← Router: classification + shared rules (~200 lines)
└── workflows/
    ├── path-a.md         ← Loaded on demand after classification
    ├── path-b.md
    └── path-c.md
```

The router pattern reduces initial context by 40-70%. Trade-off: one extra `Read` tool call per invocation (~1 second). Add a "File" column to routing tables so the LLM has an explicit path to `Read`.

### 3. Classification Design: Priority + Disambiguation

When a skill classifies user input into categories:

1. **Define a strict priority ordering** to resolve multi-match deterministically.
2. **Add disambiguation rules** for patterns that look like one type but are actually another.
3. **Test with 20-30 adversarial inputs** including clear cases, ambiguous multi-match, and false-positive traps.

### 4. Inline References Beat Cross-References

When a rule defined in one section must be applied elsewhere, add an inline reminder at the point of application. Do not rely on the LLM to cross-reference across hundreds of lines. Redundant text is reliable behavior.

### 5. Prompt Engineering Checklist

Apply to every skill before shipping:

- [ ] **Frontmatter description under 120 characters** — long descriptions hurt discovery
- [ ] **Quick Reference table** for routing or multi-step processes — tables are O(1) for LLMs
- [ ] **Explicit "Do NOT" prohibitions** in high-stakes situations — prevents well-intentioned destructive behavior
- [ ] **Optional vs. required dependencies categorized** — optional skills skip on failure; required skills retry then ask user
- [ ] **Background/reference material in `references/`** not inline — keeps SKILL.md lean
- [ ] **Absolute paths for script references** — the LLM needs the full path, not a relative one
