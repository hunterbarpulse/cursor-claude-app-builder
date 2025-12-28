# Skill Builder - Reference Documentation

This comprehensive guide provides detailed instructions for creating skills using the skill-builder meta-skill framework.

## Core Components

### Required SKILL.md Structure

Every skill must contain a `SKILL.md` file with YAML frontmatter and Markdown content. The frontmatter requires two fields:

- **name**: "Must use lowercase letters, numbers, and hyphens only" with a maximum of 64 characters
- **description**: Limited to 1024 characters and must explain both functionality and appropriate usage contexts

### Welcome Message (Required)

Every skill MUST include a welcome message immediately after the main heading and subtitle. This appears when users invoke the skill with a slash command. The welcome message should:

1. **Appear right after the main title and subtitle** (before any "## What This Skill Does" sections)
2. **Use horizontal rules** (`---`) to separate it from surrounding content
3. **Start with a bold greeting** (e.g., "**Welcome to [Skill Name]!**")
4. **List key capabilities** using bullet points starting with "This skill helps you:"
5. **Include workflow guidance** when applicable using "**Your workflow:**" section
6. **Be concise** - typically 5-10 bullet points maximum

**Template:**

```markdown
# [Skill Name]

[Brief one-line description of the skill]

---

**Welcome to [Skill Name]!**

This skill helps you:
- [Key capability 1]
- [Key capability 2]
- [Key capability 3]
- [Additional capabilities as needed]

**Your workflow:** (include this section if the skill has a specific user workflow)
1. [First step user takes]
2. [What happens next / what I do]
3. [Collaborative steps]
4. [Final outcome]

[Optional: Brief note about activation triggers or when to use]

---
```

**Example (Component System Editor):**

```markdown
# Component System Editor

A skill for propagating manual component changes through existing tiered UI component systems.

---

**Welcome to Component System Editor!**

This skill helps you:
- Proactively detect when you're editing components in existing systems
- Monitor your manual changes without interfering during your edit phase
- Propose optimal tier placement for system-wide propagation
- Summarize impact (which components will be affected)
- Update documentation with version history after approved changes

**Your workflow:**
1. Edit a component file that belongs to a system
2. I'll monitor your changes in the background
3. When you say "I'm done with [component]" or "apply changes"
4. I'll propose tier placement and show impact summary
5. After your approval, I'll propagate changes and update docs

I activate when you edit a component belonging to a system, or when you say "update the [X] system", "apply this change system-wide", or "I'm done with [component]".

---
```

**When to include workflow guidance:**

Include a "**Your workflow:**" section when:
- The skill involves **multi-step user interaction** (user does A, skill does B, user approves C)
- There's a **specific sequence** users should follow
- The skill has **phases** (edit phase, approval phase, execution phase)
- Users need to **signal when they're ready** for the skill to act

Skip the workflow section for skills that are:
- Purely reactive (just answer questions)
- Single-action (user asks, skill responds once)
- Fully automated (no user interaction after invocation)

### Optional Configuration

The `allowed-tools` field restricts Claude's access to specific tools when the skill is active. Common combinations include:

- Read-only analysis: `Read, Grep, Glob`
- File manipulation: `Read, Write, Edit, Glob, Grep`
- Shell operations: `Bash, Read, Glob`

## Description Best Practices

Effective descriptions follow this pattern: "[Action verbs] [capabilities]. Use when [specific triggers]."

**Good example**: "Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction."

**Poor example**: "Helps with documents" (lacks specificity and trigger keywords)

Critical trigger elements include file extensions (`.pdf`, `.xlsx`), technology names (pytest, docker, React), and action verbs (analyze, review, generate, extract).

## File Structure Patterns

Three main organizational approaches exist:

1. **Simple Instruction-Based**: Single `SKILL.md` file for teaching Claude processes
2. **Script-Powered**: `SKILL.md` plus `scripts/` directory for custom tooling
3. **Comprehensive Multi-File**: Includes `SKILL.md`, `REFERENCE.md`, `scripts/`, and `templates/`

## Script Implementation

Python scripts should use `uv` with PEP 723 inline metadata. The template includes:

```python
#!/usr/bin/env -S uv run
# /// script
# requires-python = ">=3.12"
# dependencies = ["package-name>=1.0.0"]
# ///
```

This approach ensures portability without separate requirements files or virtual environments.

Bash scripts are acceptable when necessary but Python with `uv` is preferred for portability and dependency management.

## Testing and Validation

The validation checklist ensures quality:

- Name compliance (lowercase, hyphens, ≤64 characters)
- Description completeness (≤1024 characters, includes what and when)
- Valid YAML frontmatter with proper opening/closing delimiters
- Clear instructions and concrete examples
- Proper documentation of any scripts or dependencies
- Correct file placement (personal vs. project location)

Skills can be tested by extracting trigger keywords from descriptions and verifying activation with those phrases, ensuring proper scope (personal skills work anywhere; project skills only in designated directories).
