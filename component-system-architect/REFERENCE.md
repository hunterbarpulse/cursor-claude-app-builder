# Component System Architect - Reference Document

This document contains templates, conventions, and formats for creating component system documentation.

---

## Documentation Template

When creating a component system, generate the following document at `docs/component-systems/[system-name].md`:

```markdown
# [System Name] System

> **Created:** YYYY-MM-DD | **Last Updated:** YYYY-MM-DD | **Version:** 1.0.0

## Overview

[Brief 1-2 sentence description of what this system handles and why it exists]

---

## Visual Diagram

[Use flow format diagrams - arrows show inheritance/composition relationships]

```
ComponentName (base)
    │
    ├──► VariantA (adds feature X)
    │        │
    │        ├──► FeatureSpecificA (integrates with Y)
    │        └──► FeatureSpecificB (integrates with Z)
    │
    ├──► VariantB (adds feature W)
    │
    └──► VariantC (adds feature V)
```

---

## Tier Structure

### Tier 1: [ComponentName] (Base)

**File:** `components/primitives/[ComponentName].tsx`

**Responsibilities:**
- [Core responsibility 1]
- [Core responsibility 2]
- [Core responsibility 3]

**Base Props:**

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `ReactNode` | - | Content to display |
| `variant` | `'default' \| 'primary'` | `'default'` | Visual variant |
| `size` | `'sm' \| 'md' \| 'lg'` | `'md'` | Size preset |
| `disabled` | `boolean` | `false` | Disabled state |

---

### Tier 2: Variants

| Component | File | Purpose | Added Props |
|-----------|------|---------|-------------|
| VariantA | `components/primitives/VariantA.tsx` | [Brief purpose] | `onRemove`, `isSelected` |
| VariantB | `components/primitives/VariantB.tsx` | [Brief purpose] | `icon`, `iconPosition` |

#### VariantA Details

**Added Props:**

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `onRemove` | `() => void` | - | Callback when remove is triggered |
| `isSelected` | `boolean` | `false` | Selected state |

---

### Tier 3: Specific Implementations

| Component | File | Purpose |
|-----------|------|---------|
| SpecificImpl | `features/[feature]/components/SpecificImpl.tsx` | [Feature-specific purpose] |

---

## Application Tracking

This section tracks all files in the codebase that use components from this system. It is automatically updated by the component-system-architect, component-system-editor, and component-system-auditor skills.

### Tier 1: [ComponentName] Applications

**Direct Usage:** (files importing and using ComponentName directly)
- `features/example/SomeScreen.tsx` - Example screen with basic usage
- `features/another/OtherScreen.tsx` - Another example
- `screens/Dashboard.tsx` - Dashboard implementation

**Total files using Tier 1:** 3

---

### Tier 2: Variant Applications

#### VariantA Applications
**Direct Usage:**
- `features/example/MultiSelectScreen.tsx` - Multi-select with removable items
- `features/filters/FilterSheet.tsx` - Filter chips

**Total files using VariantA:** 2

#### VariantB Applications
**Direct Usage:**
- `features/navigation/TabBar.tsx` - Icon-based navigation

**Total files using VariantB:** 1

---

### Tier 3: Specific Implementation Applications

**Note:** Tier 3 components are feature-specific and typically used only within their feature domain.

#### SpecificImpl Applications
- `features/[feature]/FeatureScreen.tsx` - Main feature implementation
- `features/[feature]/components/FeatureSection.tsx` - Section within feature

**Total files using SpecificImpl:** 2

---

### System-Wide Usage Summary

| Tier Level | Component | Usage Count |
|------------|-----------|-------------|
| Tier 1 | ComponentName | 3 |
| Tier 2 | VariantA | 2 |
| Tier 2 | VariantB | 1 |
| Tier 3 | SpecificImpl | 2 |
| **Total** | - | **8** |

**Last Updated:** YYYY-MM-DD

---

## Usage Examples

### Tier 1: Basic Usage

```tsx
import { ComponentName } from '@/components/primitives/ComponentName';

// Default usage
<ComponentName>
  Label
</ComponentName>

// With variant
<ComponentName variant="primary" size="lg">
  Primary Label
</ComponentName>
```

### Tier 2: Variant Usage

```tsx
import { VariantA } from '@/components/primitives/VariantA';

<VariantA
  isSelected={true}
  onRemove={() => handleRemove(item.id)}
>
  Removable Item
</VariantA>
```

### Tier 3: Feature-Specific Usage

```tsx
import { SpecificImpl } from '@/features/[feature]/components/SpecificImpl';

<SpecificImpl
  data={items}
  onSelect={handleSelect}
  customProp={value}
/>
```

---

## Change Propagation Guide

| If you change... | It affects... |
|------------------|---------------|
| Tier 1 base styling (padding, colors) | ALL components in this system |
| Tier 1 base props interface | ALL components (may require updates) |
| Tier 2 VariantA styling | VariantA and any Tier 3 children of VariantA |
| Tier 2 VariantB behavior | VariantB and any Tier 3 children of VariantB |
| Tier 3 SpecificImpl | Only SpecificImpl |

### Common Update Scenarios

**Scenario: Update padding across all chips**
→ Edit Tier 1 `ComponentName.tsx` base styles
→ All Tier 2 and Tier 3 components automatically inherit the change

**Scenario: Add new prop to a variant**
→ Edit the specific Tier 2 component
→ Only that variant and its children are affected

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | YYYY-MM-DD | Initial system created |

---

## Related Systems

- [Other System] - [Relationship description, e.g., "Uses this system's base component"]
```

---

## Consolidation Analysis

Before proposing a new system, analyze the codebase for consolidation opportunities. Present findings in this format:

### Consolidation Report Template

```markdown
## Consolidation Analysis for [Component Type]

### Components Found
| Component | Location | Usage Count | Similar To |
|-----------|----------|-------------|------------|
| Input | primitives/Input.tsx | 15 | TextField |
| SearchBar | primitives/SearchBar.tsx | 8 | TextField |
| TextField | primitives/TextField.tsx | 3 | - |

### Recommended Actions

#### 1. DELETE: Input.tsx
- **Reason:** Wrapper that only re-exports TextField
- **Migration:** Change `import { Input }` → `import { TextField }`
- **Files affected:** 15

#### 2. MERGE: SearchBar into TextField
- **Reason:** SearchBar is just TextField with `searchMode={true}`
- **Action:** Add `searchMode` prop to TextField, delete SearchBar
- **Files affected:** 8

#### 3. KEEP: TextField.tsx
- **Reason:** Base component with all necessary props
- **Becomes:** Tier 1 of TextField system

### Post-Consolidation Structure
```
TextField (base)
    │
    └──► [No Tier 2 needed - variants handled by props]
```

### Component Count Change
- Before: 3 components
- After: 1 component
- Reduction: 67%
```

### Questions to Ask During Analysis

1. **Are there near-duplicate components?**
   - Look for components with 80%+ similar styling/behavior
   - Check if differences can be handled by props

2. **Are there unnecessary wrappers?**
   - Component that just passes props to another
   - Component that only sets default prop values

3. **Are Tier 2 components truly extending, or copy-pasting?**
   - Tier 2 should import and render Tier 1
   - If Tier 2 duplicates Tier 1 code, refactor to wrap instead

4. **Is every component actually used?**
   - Search for import statements
   - Orphan components should be deleted or questioned

5. **Could one component do multiple jobs?**
   - `editable={false}` can turn input into trigger
   - `variant` prop can handle visual differences
   - Icons/children can be composed rather than specialized

---

## Naming Conventions

### Component Names by Tier

| Tier | Naming Pattern | Examples |
|------|----------------|----------|
| Tier 1 (Base) | Simple, single word | `Chip`, `Button`, `Card`, `Sheet` |
| Tier 2 (Variants) | Descriptive prefix/suffix | `RemovableChip`, `IconButton`, `ListPickerSheet` |
| Tier 3 (Specific) | Feature-specific | `VocabularyChips`, `WinePickerSheet`, `FilterChips` |

### File Organization

```
components/
├── primitives/              # Tier 1 and simple Tier 2
│   ├── Chip.tsx             # Tier 1
│   ├── RemovableChip.tsx    # Tier 2
│   ├── Button.tsx           # Tier 1
│   └── IconButton.tsx       # Tier 2
│
├── sheets/                  # Complex system with its own folder
│   ├── BottomSheetContainer.tsx  # Tier 1
│   ├── ListPickerSheet.tsx       # Tier 2
│   ├── components/               # Internal components
│   │   ├── SearchHeader.tsx
│   │   └── SelectedChipsSection.tsx
│   └── hooks/
│       └── useListItemStyling.ts
│
└── layout/
    └── ...

features/
└── [feature]/
    └── components/
        └── SpecificImpl.tsx  # Tier 3 (feature-specific)
```

---

## Visual Diagram Formats

Always use flow format diagrams. The format shows component inheritance/composition with arrows.

### Two-Tier System

```
ComponentName (base)
    │
    ├──► VariantA (adds feature X)
    ├──► VariantB (adds feature Y)
    └──► VariantC (adds feature Z)
```

### Three-Tier System

```
ComponentName (base)
    │
    ├──► VariantA (adds feature X)
    │        │
    │        ├──► FeatureSpecificA (integrates with Y)
    │        └──► FeatureSpecificB (integrates with Z)
    │
    └──► VariantB (adds feature W)
             │
             └──► FeatureSpecificC (integrates with Q)
```

### System with Shared Hooks

```
ComponentName (base) + useSharedHook
    │
    ├──► VariantA (uses hook for X)
    └──► VariantB (uses hook for Y)
```

---

## Version History Format

Use semantic versioning:
- **Major (X.0.0):** Breaking changes to props or structure
- **Minor (1.X.0):** New variants or features added
- **Patch (1.0.X):** Bug fixes, styling tweaks

Example entries:

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2025-12-25 | Initial system created |
| 1.1.0 | 2025-12-28 | Added RemovableChip variant |
| 1.1.1 | 2025-12-29 | Fixed padding inconsistency in base Chip |
| 2.0.0 | 2026-01-15 | Restructured props interface (breaking) |

---

## Preferences File Format

Location: `~/.config/claude-code/component-system-patterns/preferences.md`

```markdown
# Component System Preferences

> Last updated: YYYY-MM-DD

## Naming Conventions
- [User's preferred naming patterns]

## Documentation Style
- [User's preferred documentation elements]

## Patterns Liked
- [Pattern description] (Project, Date)

## Patterns Rejected
- [Pattern description] (Project, Date, Reason)

## Notes
- [Any other preferences learned]
```

---

## Systems Index Format

Location: `~/.config/claude-code/component-system-patterns/systems-index.md`

```markdown
# Component Systems Index

> Cross-project reference for inspiration (not constraint)

## [Project Name] (Technology)

| System | Tiers | Created | Key Pattern |
|--------|-------|---------|-------------|
| Bottom Sheets | 3 | YYYY-MM | Container → ListPicker → Specific implementations |
| Chips | 2 | YYYY-MM | Base Chip → RemovableChip |

### Notable Decisions
- [Decision and reasoning]

---

## [Another Project] (Technology)

...
```
