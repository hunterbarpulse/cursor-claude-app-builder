---
name: component-system-auditor
description: Audits component systems for outliers and integrity issues. Finds components NOT in systems that SHOULD be, and detects cascade blockers WITHIN systems. Use when saying "audit", "check for outliers", "system health", "find inconsistencies", or after component-system-architect creates a new system. Outputs plain-language summaries with fix recommendations.
---

# Component System Auditor

A skill for ensuring component system health by finding outlier components that should be part of existing systems and detecting integrity issues that block proper cascade behavior.

---

**Welcome to Component System Auditor!**

This skill helps you:
- Find components NOT in systems that SHOULD be (External Audit/Outlier Detection)
- Detect cascade blockers WITHIN existing systems (Internal Audit/System Integrity)
- Run 63 deep-scan checks across 13 categories for thorough system health
- Generate plain-language reports understandable by non-technical users
- Provide item-by-item fix recommendations with approval before changes

**Your workflow:**
1. Say "audit my component systems" or "check for outliers in [system]"
2. I'll read the system documentation and scan the codebase
3. I'll present findings in plain language (one issue at a time)
4. For each issue, you approve or skip the recommended fix
5. After fixes, I update the system documentation

I work on one system at a time for focused, thorough coverage. Use this when you want to find inconsistencies or verify system health.

---

## What This Skill Does

1. **External Audit (Find Outliers):** Scans codebase for components NOT in any system that SHOULD be
2. **Internal Audit (System Integrity):** Deep-scans existing systems for cascade blockers
3. **Generates plain-language reports** understandable by non-coders
4. **Recommends fixes** with item-by-item approval before any changes
5. **Updates documentation** after fixes are applied

## When This Skill Activates

### On-Command Activation
When you say things like:
- "Audit my component systems"
- "Check for outliers in [system]"
- "Run a system health check"
- "Find inconsistencies"
- "Are there any components not using [TextField/Chip/Card/etc.]?"
- "Deep scan [system name]"

### Post-Architect Prompt (User-Initiated)
After `component-system-architect` creates a new system, I may ask:

> "Would you like me to audit for outliers in the new [system] system?"

**IMPORTANT:** I will NOT auto-run. I will always wait for your explicit "yes" before auditing.

## Scope

This skill works on **ONE system at a time**. When you request an audit, I will:
1. Ask which system to audit (if not specified)
2. Read that system's documentation from `docs/component-systems/[system-name].md`
3. Perform the audit
4. Update the system's Version History after any approved fixes

## Audit Types

### 0. Regression Check (Application Tracking)

**Purpose:** Ensure all systems have up-to-date application tracking sections.

**What I Check:**
- Does the system doc have an "Application Tracking" section?
- If YES: Is it current with actual component imports in the codebase?
- If NO: Generate it by scanning for all component imports

**When I Run This:**
- **ALWAYS** at the start of every audit (before external or internal audits)
- Automatically fixes missing or outdated application tracking
- Does NOT require approval (this is maintenance, not a fix)

**Output:**
- If missing: "Generated Application Tracking section for [System Name]"
- If outdated: "Updated Application Tracking section with current usage"
- If current: Silently proceed to requested audit type

---

### 1. External Audit (Outlier Detection)

**Purpose:** Find components scattered across the codebase that should be part of an existing component system.

**What I Look For:**
- Component name patterns matching system components (e.g., "input", "textfield", "search" for TextField system)
- Similar styling patterns (borderRadius, padding values matching tier 1 constants)
- Raw React Native components where system components exist (e.g., raw `TextInput` when `TextField` exists)
- Hardcoded values that match system tokens
- Edge cases and non-obvious candidates that could benefit from the system

**Output Format:**
> "This **[component name]**, used in **[list of files/screens]**, isn't cooperating because **[plain-language issue]**."
>
> **Recommendation:** [Migrate to existing tier] OR [Create new tier component]

### 2. Internal Audit (System Integrity)

**Purpose:** Find cascade blockers WITHIN existing component systems that prevent proper inheritance.

**Deep Scan Categories (63 checks across 13 categories):**
- **A: Inheritance & Composition** - Tier 2/3 not properly using/wrapping Tier 1
- **B: Styling & Tokens** - Hardcoded values, duplicated styles, wrong tokens
- **C: Surface & Context** - Missing SurfaceProvider, surface props not forwarded
- **D: Migration Leftovers** - Partial migrations, dead imports, old commented code
- **E: Prop & API Mismatches** - Type mismatches, missing required props, different defaults
- **F: Documentation Drift** - Undocumented components, outdated prop tables
- **G: Behavioral Issues** - Events not bubbling, animation mismatches, accessibility dropped
- **H: State & Data Flow** - Parent-child state conflicts, prop spreading overwrites
- **I: Accessibility Deep** - Role changes, touch targets, screen reader blocking
- **J: Type & API Compatibility** - Variant restrictions, callback mismatches, union narrowing
- **K: Hook & Utility Dependencies** - Shared hook changes, utility drift, context assumptions
- **L: Performance Issues** - Inline objects, missing memos, incomplete dependency arrays
- **M: Conditional Logic Traps** - Style merge order, ternary bugs, unreachable branches

**Output Format:**
> "This **[component name]** isn't cooperating because **[plain-language issue]**."
>
> **Location:** `[file path]`
> **Impact:** [What breaks if not fixed]
> **Fix:** [Plain-language fix recommendation]

## Core Principles

### Plain-Language Output
All output is in plain language. Instead of technical jargon like:
> "RemovableChip.tsx:52 - hardcoded fontWeight '500' overrides Chip base style"

I will say:
> "RemovableChip, used in VocabularyChips and TagSelector, isn't cooperating because it has its own text weight instead of inheriting from the base Chip. This means if you change Chip's text weight, RemovableChip won't update."

### Item-by-Item Approval
Before fixing any issue, I will:
1. Present ONE issue at a time
2. Explain it in plain language
3. Show the recommended fix
4. Wait for your "yes" before proceeding
5. Then move to the next issue

### Deep Scan by Default
When running internal audits, I will perform full deep scans covering all 63 checks across 13 categories unless you request a quick scan.

### One System at a Time (Usually)
I focus on one system per audit to ensure thorough coverage and clear documentation updates.

**Exception: Cross-System Audit**
When you say "audit [ComponentName] across systems" or when a component uses multiple systems (like a Card containing Chips and TextFields), I'll run cross-system checks to catch issues that only appear when systems interact.

## Workflow

### Regression Check Workflow (Always Runs First)
1. **Read target system doc** from `docs/component-systems/[system-name].md`
2. **Check for "Application Tracking" section**
3. **If missing**: Generate by scanning for component imports
4. **If exists**: Verify it matches current codebase imports
5. **Update or create** the section as needed
6. **Proceed** to external or internal audit

### External Audit Workflow
1. **Run regression check** (Application Tracking)
2. **Extract tier 1 component signatures** (props, styling constants, file patterns)
3. **Scan codebase** for matching patterns outside the system
4. **Categorize outliers** by recommended action
5. **Present findings** with plain-language summaries
6. **Await approval** for each fix

### Internal Audit Workflow
1. **Run regression check** (Application Tracking)
2. **Trace tier hierarchy** from tier 1 to tier 2 to tier 3
3. **Run deep scan checks** on each component (31 checks across 7 categories)
4. **Identify cascade blockers**
5. **Present findings** with plain-language summaries
6. **Await approval** for each fix

## Documentation Updates

After each approved fix, I update the system's documentation:

**File:** `docs/component-systems/[system-name].md`

**Version History Update:**

| Version | Date | Changes |
|---------|------|---------|
| X.X.X | [today] | [description of what was fixed] |

**Versioning Rules:**
- **Patch (1.0.X):** Fixed cascade blocker, removed dead code, fixed prop passthrough
- **Minor (1.X.0):** Integrated new outlier component, added missing component to system
- **Major (X.0.0):** Significant restructuring (rare, usually via architect)

## CRITICAL: Approval Requirements

I will NEVER make changes without your explicit approval. Before ANY action:

- I present findings one at a time
- I explain in plain language what's wrong
- I show the recommended fix
- I WAIT for your "yes" to proceed
- I confirm after each fix before moving on

## Handoff Patterns

### From component-system-architect
After architect creates a new system:
> "The [System Name] system has been created. Would you like me to audit for outliers that should be part of this new system?"

### To component-system-editor
When a fix requires propagating a change through the system:
> "This fix needs to update [component] and cascade to [children]. Handing off to component-system-editor."

### To component-system-architect
When an outlier needs a new tier component:
> "This outlier ([component]) needs a new tier 2 variant in the [system] system. Handing off to component-system-architect."

## Related Skills

### Receiving from `component-system-architect`
After a system is created, the architect may suggest running an outlier audit. This hands off here.

### Handing to `component-system-editor`
When fixes require propagating changes through existing tiers, I hand off to the editor for tier-aware propagation.

### Handing to `component-system-architect`
When outliers need new tier components that don't exist, I hand off to the architect for variant creation.

## Instructions

When this skill activates:

1. Read the REFERENCE.md file for search patterns, check procedures, and templates
2. Determine which system to audit (ask if not specified)
3. Read `docs/component-systems/[system-name].md` to understand the target system
4. **REGRESSION CHECK:** Check if "Application Tracking" section exists in the system doc
   - If MISSING: Generate it by scanning for component imports
   - If EXISTS but outdated: Update it with current usage
5. Determine audit type (external/outlier vs internal/integrity)
6. If post-architect, ask user before running
7. Execute appropriate scan workflow
8. Present findings in plain-language format
9. Process fixes one at a time with user approval
   - Present fix with explicit integration method (Structural Replacement / Compositional Wrapping / Prop Forwarding)
   - Apply the fix
   - **VERIFY integration** using Category N checks (checks #64-68)
   - If verification fails, report to user and provide correction guidance
   - Only proceed to next issue when verification passes
10. Update system documentation after approved fixes
11. **Update application tracking** section with any new/changed component usage
12. Hand off to sister skills when appropriate
