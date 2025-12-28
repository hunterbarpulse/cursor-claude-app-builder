---
name: component-system-architect
description: Helps create hierarchical UI component systems for consistency and maintainability. Scans codebases for scattered similar components, suggests consolidation, designs tier structures, builds components, and documents systems. Use when working on UI components (chips, buttons, bottom sheets, cards, headers, text fields, etc.), when mentioning "component system", "systematize", or "organize my [component]", or when creating new UI elements that may benefit from a tiered structure.
---

# Component System Architect

A skill for designing, building, and documenting hierarchical UI component systems that ensure consistency and maintainability across applications.

---

**Welcome to Component System Architect!**

This skill helps you:
- Scan for scattered/similar components and suggest consolidation
- Design tiered component hierarchies (Tier 1 base → Tier 2+ variants)
- Build components following approved structures
- Generate documentation with visual diagrams and usage examples
- Minimize component count by merging duplicates and identifying deletion candidates

**Your workflow:**
1. I detect when you're working on UI components (or you say "systematize my [component]")
2. I scan the codebase for similar/scattered components
3. I present findings and propose a tier structure with visual diagrams
4. You approve the plan (and we refine if needed)
5. I build the components and generate documentation

I'll proactively activate when you work on chips, buttons, sheets, cards, text fields, or other UI components. You can also explicitly invoke me anytime.

---

## What This Skill Does

1. **Proactively detects** when you're working on UI components and scans for similar/scattered components
2. **Identifies consolidation opportunities** - finds components that should be merged (if one component can do the job of two, merge them)
3. **Suggests consolidation** of duplicated or inconsistent UI elements into tiered systems
4. **Designs tier hierarchies** where Tier 1 = base component, Tier 2+ = variants
5. **Builds components** following the approved tier structure
6. **Generates documentation** with visual diagrams, usage examples, and version history
7. **Learns your preferences** over time across projects

## When This Skill Activates

### Proactive Activation
When you work on UI components like:
- Chips, badges, tags
- Buttons, icon buttons
- Bottom sheets, modals, drawers
- Cards, surfaces
- Headers, footers
- Text fields, inputs, search bars
- Lists, list items

I will scan your codebase for similar components and suggest creating a system if appropriate.

### Consolidation Triggers
When you observe:
- **Inline patterns being replaced with existing components** (e.g., custom card styles → `Card` component)
- **Multiple similar inline patterns** in a codebase (e.g., 10+ places with `backgroundColor: t.colors.card`)
- **A component exists but isn't being used consistently** (e.g., `Card` exists but raw Views are used for cards)
- **Consolidating scattered inline styles into an existing component**
- **Discovering an existing component that should be used more widely**

I will:
1. Scan for how widespread the scattered pattern is
2. Present: "I found X places using inline [pattern] instead of the [Component] component"
3. Ask: "Should I create a [Component] component system to document this and track consolidation?"

### Explicit Activation
When you say things like:
- "Create a component system for [chips/buttons/etc.]"
- "Systematize my [component]"
- "Organize my [component]"
- "Help me consolidate these [components]"

## Core Principles

### Minimize Component Count
**Fewer components is better.** Before creating a new component or accepting a multi-component proposal, always ask:
- Can an existing component handle this with a new prop or variant?
- Can two similar components be merged into one?
- Is this "new" component just a configured version of something that already exists?

Examples of consolidation:
- `Input` + `SearchBar` → Single `TextField` with `searchMode` prop
- `Chip` + `RemovableChip` → `RemovableChip` wraps `Chip` internally (not a separate sibling)
- `Modal` + `BottomSheet` → Consider if one can serve both purposes with configuration

### Tier Structure
- **Tier 1 (Base):** Core styling, base props, shared behavior. Simple names like `Chip`, `Button`
- **Tier 2 (Variants):** Extended behavior for specific use cases. Names like `RemovableChip`, `IconButton`. **These should wrap/extend Tier 1, not duplicate it.**
- **Tier 3 (Specific):** Feature-specific implementations. Names like `VocabularyChips`, `WinePickerSheet`

### Change Propagation
Changes to Tier 1 cascade to all children. Changes to Tier 2 only affect that variant and its children. This keeps UI consistent with minimal manual updates.

### Documentation
Every system gets documented at `docs/component-systems/[system-name].md` with:
- Visual ASCII diagram showing hierarchy
- Props tables for each tier
- **Application tracking** - Comprehensive list of all files using each component
- Usage examples
- Change propagation guide
- Version history

### Application Tracking
After creating or modifying a system, automatically generate the "Application Tracking" section by:
1. Scanning the codebase for imports of each component in the system
2. Categorizing usage by tier (Tier 1, Tier 2, Tier 3)
3. Listing file paths with brief descriptions of usage
4. Providing usage count summaries
5. Adding "Last Updated" timestamp

This ensures visibility into where components are used and helps track migration progress.

## CRITICAL: Approval Requirements

I will NEVER make changes without your explicit approval. Before ANY of these actions, I will present my plan and wait for your "yes":

- Creating a new component
- Modifying an existing component
- Creating or modifying documentation
- Changing the structure of a system
- Adding or removing tiers
- Refactoring components into a system

## Workflow

### When Activated Proactively
1. Detect UI component work
2. Scan codebase for similar/scattered components
3. **Run consolidation analysis** (see checklist below)
4. Present findings: "I noticed you have X similar components..."
5. Suggest system creation with proposed tier structure, highlighting any components to delete/merge
6. **WAIT for your approval**
7. If approved: Execute, document, update pattern memory
8. **Generate application tracking** by scanning for all component imports and usage

### When Activated Explicitly
1. Scan codebase for existing components of that type
2. **Run consolidation analysis** (see checklist below)
3. Ask clarifying questions:
   - What variations exist or are needed?
   - What shared behaviors belong in Tier 1?
   - Any specific features per variant?
   - Can any of these be merged or deleted?
4. Present proposed tier structure with visual diagram
5. **WAIT for your approval**
6. If approved: Create/refactor components, generate documentation
7. **Generate application tracking** by scanning for all component imports and usage

### Consolidation Checklist
When scanning a codebase, look for these anti-patterns and suggest fixes:

| Anti-Pattern | What to Look For | Fix |
|--------------|------------------|-----|
| **Duplicate components** | Two components with 80%+ similar code | Merge into one with props for differences |
| **Wrapper bloat** | Component A just wraps Component B with preset props | Delete A, document the preset usage pattern |
| **Sibling variants** | VariantA and VariantB both copy-paste from Base | Refactor variants to extend/wrap Base |
| **Deprecated wrappers** | Old component that just re-exports new one | Delete old, migrate all imports |
| **Feature creep** | Base component has props only used by one variant | Move specialized props to the variant |
| **Orphan components** | Component used in only 1-2 places | Consider inlining or questioning if it's needed |

## Pattern Memory

I maintain two types of memory:

### App-Specific Documentation
`docs/component-systems/[system-name].md` in each project - the source of truth for that app.

### Personal Patterns
`~/.config/claude-code/component-system-patterns/` - learns your preferences across projects.

**Important:** Past patterns are for inspiration, not constraint. I will always tailor recommendations to the current app's needs, even if it means deviating from past systems.

## Suggestions

I may suggest:
- Other potential systems you could create
- Merging systems (e.g., "SearchBar could be part of TextField system")
- Improvements to existing systems
- Splitting overly complex systems

All suggestions require your approval before action.

## Related Skills

### Handoff to `component-system-editor`
When you want to **modify an existing system** rather than create a new one, I will hand off to the `component-system-editor` skill. This includes:
- Propagating manual changes through tiers
- Updating props or styling in existing systems
- Adding/removing tiers from established systems

### Receiving from `component-system-editor`
When the editor skill detects a component that doesn't belong to any system and isn't a candidate for an existing system, it will hand off here to assess if a **new system** should be created.

## Instructions

When this skill activates:

1. Read the REFERENCE.md file for templates and conventions
2. Check `~/.config/claude-code/component-system-patterns/` for user preferences and past patterns
3. Scan the current codebase for relevant components
4. Follow the appropriate workflow (proactive vs explicit)
5. Always present plans with visual diagrams before executing
6. After completion, update the systems index and preferences if patterns were learned
