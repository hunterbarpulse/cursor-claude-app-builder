---
name: skill-builder
description: Interactive skill creation assistant that guides users through building new Agent Skills for Claude Code. Use when creating new skills, building custom capabilities, or when the user runs /new-skill command. Helps design skill structure, craft descriptions, create scripts, and organize supporting files.
---

# Skill Builder

A conversational meta-skill that helps you create new Agent Skills for Claude Code through guided interaction.

---

**Welcome to Skill Builder!**

This skill helps you:
- Design and create new Agent Skills for Claude Code
- Craft effective descriptions for proper skill activation
- Structure skills (instruction-based vs script-powered)
- Generate all necessary files with best practices
- Include welcome messages that explain functionality and user workflow

**Your workflow:**
1. Describe what you want the skill to do
2. I'll ask clarifying questions about purpose and use cases
3. We'll craft the description and determine structure together
4. I'll create all files (SKILL.md, scripts, templates, etc.)
5. We'll test and refine based on your feedback

All skills will include a welcome message explaining their functionality and any specific workflow the user should follow.

---

## Core Purpose

Guide users through creating well-structured, discoverable Agent Skills by:
1. Understanding the skill's purpose and use cases
2. Crafting effective descriptions for model invocation
3. Determining the right structure (instruction-based vs script-powered)
4. Creating all necessary files with best practices
5. Iteratively refining based on user feedback

## When This Skill Activates

- User runs `/new-skill` command
- User asks about creating, building, or designing a new skill
- User wants to build custom Claude Code capabilities
- User needs help with skill structure or organization

## Requirements

No external dependencies required. This skill guides users through skill creation using built-in tools.

## Instructions

1. **Understand the user's needs** - Ask clarifying questions about the skill's purpose, use cases, and target scenarios
2. **Craft an effective description** - Work collaboratively to create a concise description that clearly states what the skill does and when to use it
3. **Determine structure** - Assess whether the skill needs scripts, reference documentation, or just instructions
4. **Consider tool restrictions** - Determine if the skill should limit which tools Claude can access
5. **Design the welcome message** - Create a welcome message that explains key capabilities and user workflow (when applicable)
6. **Create all files** - Generate SKILL.md with welcome message, scripts (if needed), and reference documentation
7. **Iterate and refine** - Test the skill and make adjustments based on activation and user feedback

**CRITICAL**:
- Every skill MUST include a welcome message following the template in REFERENCE.md. Read REFERENCE.md for the complete welcome message template and guidelines on when to include workflow sections.
- After creating a new skill, ALWAYS ask the user: "Would you like me to create a slash command wrapper so you can invoke this skill manually with /skill-name?" If yes, create a lightweight wrapper at `.claude/commands/[skill-name].md` that simply tells Claude to use the skill.

## Examples

**Example 1: Creating an instruction-based skill**
User: "I want a skill for writing better commit messages"
Result: Single SKILL.md file with detailed instructions on effective commit message practices

**Example 2: Creating a script-powered skill**
User: "I need a skill to analyze PDF test reports"
Result: SKILL.md + Python script using uv for dependency management to parse and analyze PDFs
