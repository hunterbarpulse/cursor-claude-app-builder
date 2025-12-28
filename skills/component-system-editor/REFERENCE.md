# Component System Editor - Reference Document

This document contains logic, templates, and conventions for editing and propagating changes through existing component systems.

---

## Change Detection Patterns

### Identifying System Membership

To determine if a component belongs to an existing system:

1. **Read all system docs** in `docs/component-systems/*.md`
2. **Extract file paths** from each system's tier structure
3. **Match the edited file** against known system files

```
Example matching:
Edited file: app/src/features/insights/components/WinePickerSheet.tsx
System: Bottom Sheets (docs/component-systems/bottom-sheets.md)
Match: Tier 3 component in Bottom Sheets system
```

### Identifying Overrides vs System Defaults

When a component is edited, compare:

| What Changed | System Default Location | Override Indicator |
|--------------|------------------------|-------------------|
| Padding/spacing | Tier 1 base styles | Hardcoded value in Tier 2/3 |
| Colors | Theme tokens via Tier 1 | Direct color value in Tier 2/3 |
| Border radius | Tier 1 base styles | Inline style in Tier 2/3 |
| Typography | Tier 1 text styles | Custom font props in Tier 2/3 |
| Composition usage | Parent tier component | Component doesn't render parent |
| Behavior/callbacks | Tier-specific | N/A (usually intentional) |

**Override detection heuristic:**
- If a style that should come from a parent tier is defined inline → potential propagation candidate
- If a prop value differs from the system doc's documented default → potential update

### Composition Detection (CRITICAL)

Before analyzing any style/prop changes, verify compositional integration:

**Question to answer first:** Does this component render its parent tier component?

**Detection method:**
1. Identify parent tier from system doc
2. Read component file
3. Check for import of parent component
4. Grep return statement for parent component usage

**If parent is NOT rendered:**
```
⚠️  COMPOSITION MISSING

This component is not structurally integrated into the system.
Any style/prop changes you propagate will NOT cascade to this component.

STOP: Do not analyze overrides until composition is fixed.
ACTION: Hand off to component-system-auditor to fix composition.
```

**Only after confirming composition**, proceed with override detection.

---

## Tier Placement Logic

### Goal: Place Changes as High as Possible

The higher the tier, the more components inherit the change automatically.

### Decision Tree

```
1. Does this change apply to ALL components in the system?
   → YES: Place in Tier 1 (base)
   → NO: Continue to 2

2. Does this change apply to a specific variant AND all its children?
   → YES: Place in that Tier 2 variant
   → NO: Continue to 3

3. Is this change specific to one feature implementation?
   → YES: Keep in Tier 3 (feature-specific)
   → NO: Re-evaluate - might be a new variant needed
```

### Examples

| Change | Recommended Tier | Reasoning |
|--------|-----------------|-----------|
| Update base padding from 16 to 20 | Tier 1 | Affects all sheets |
| Add close button to removable sheets | Tier 2 (if variant exists) | Only affects removable variant |
| Custom header for wine picker | Tier 3 | Feature-specific |
| New border radius across all | Tier 1 | Universal change |
| New animation for search sheets | Tier 2 (SearchSheet) | Variant-specific |

---

## Impact Analysis Template

When presenting changes to the user:

```markdown
## TLDR: Proposed System Update

**Change:** [Concise description of what's being changed]

**Recommended tier:** Tier [X]: [ComponentName]
**File:** `[path/to/file.tsx]`

**This change will affect:**
- [Component A] (`path/to/A.tsx`) - Tier 2
- [Component B] (`path/to/B.tsx`) - Tier 3
- [Component C] (`path/to/C.tsx`) - Tier 3

**What stays the same:**
- [Components not affected, if relevant]

---

**Is Tier [X] the right place?**
- Yes, apply to Tier [X]
- No, I want it in a different tier: [specify]
```

---

## Cascade Readiness Verification

Before propagating ANY change to a higher tier, verify the component will actually receive the cascaded change.

### Three-Gate Verification System

**Gate 1: Structural Integration Check**
```
Question: Does the component render the parent tier component?

How to verify:
1. Read the component file
2. Check imports for parent component
3. Grep return statement for `<ParentComponent`

PASS: Component contains `return (...<ParentComponent>...)`
FAIL: Component builds its own JSX without using parent

If FAIL → DO NOT PROPAGATE
         → Report: "Component is not structurally integrated"
         → Hand off to auditor to fix composition first
```

**Gate 2: Prop Forwarding Check**
```
Question: Will the changed prop/style actually reach the parent component?

How to verify:
1. Identify what's being changed (e.g., `padding`, `variant`, `size`)
2. Check if component forwards this to parent
3. Look for `{...props}` or explicit prop drilling

PASS: Prop is forwarded to parent OR component doesn't override it
FAIL: Component accepts prop but doesn't pass it through

If FAIL → DO NOT PROPAGATE
         → Report: "Component accepts [prop] but doesn't forward it"
         → Recommend: Add prop forwarding first
```

**Gate 3: Style Override Check**
```
Question: Are there inline style overrides blocking the cascade?

How to verify:
1. Check for hardcoded styles matching what you want to change
2. Look for style objects/arrays that override parent styles
3. Use Category B checks from auditor (hardcoded values, wrong tokens)

PASS: No overrides, or overrides are intentional additions
FAIL: Component hardcodes the value you're trying to cascade

If FAIL → DO NOT PROPAGATE
         → Report: "Component has hardcoded [style] override"
         → Recommend: Remove override to allow cascade
```

**Override Classification for Gate 3:**

**ALLOWED (Intentional Addition):**
- Child adds NEW style properties not handled by parent
- Child applies conditional styling based on its own state
- Example: `<Chip style={{ marginTop: selected ? 8 : 0 }}>` ✅

**BLOCKING (Prevents Cascade):**
- Child overrides style property parent already defines
- Child hardcodes value that should come from parent
- Example: `<Chip style={{ padding: 16 }}>` when Chip has padding ❌

**Detection heuristic:**
If parent's baseStyles contains property X, and child also defines property X → BLOCKING

### Verification Workflow

```
User signals: "Apply this change system-wide"
     │
     ├─ Analyze what changed
     ├─ Propose tier placement
     │
     ├─ FOR EACH affected component:
     │   ├─ Run Gate 1: Structural Integration
     │   ├─ Run Gate 2: Prop Forwarding
     │   ├─ Run Gate 3: Style Override
     │   │
     │   ├─ ALL GATES PASS? → Component ready for cascade
     │   └─ ANY GATE FAILS? → Flag component
     │
     ├─ Present TLDR with:
     │   ├─ ✅ Components ready for cascade
     │   └─ ⚠️  Components NOT ready (with reasons)
     │
     └─ Ask user:
         ├─ "Fix flagged components first?" → Hand to auditor
         └─ "Proceed with ready components only?" → Continue
```

---

## Cascade Readiness Report Template

When presenting changes, include cascade readiness status:

```markdown
## TLDR: Proposed System Update

**Change:** [description]
**Recommended tier:** Tier [X]: [ComponentName]

### Cascade Impact Analysis

**✅ Ready for Cascade (will inherit changes):**
- [Component A] (`path/to/A.tsx`) - Tier 2
  - Renders <ParentComponent>
  - Forwards all relevant props
  - No blocking overrides

**⚠️ NOT Ready for Cascade (will NOT inherit):**
- [Component B] (`path/to/B.tsx`) - Tier 3
  - ❌ **Issue:** Doesn't render <ParentComponent>, builds own JSX
  - **Fix needed:** Replace JSX with <ParentComponent>

- [Component C] (`path/to/C.tsx`) - Tier 3
  - ❌ **Issue:** Hardcoded padding override
  - **Fix needed:** Remove hardcoded padding

**Recommendation:**
- Option 1: Fix flagged components first (hand off to auditor)
- Option 2: Proceed with [Component A] only, fix others later

**Is Tier [X] the right place?**
[yes/different tier/fix components first]
```

---

## Documentation Update Templates

### System Doc Update (docs/component-systems/[system].md)

After a change is applied, update:

1. **Props tables** if props changed
2. **Visual diagram** if structure changed
3. **Version history** (always)

Version history entry format:
```markdown
| [new version] | YYYY-MM-DD | [Description of change] |
```

### Version Bump Guidelines

| Change Type | Version Bump | Examples |
|-------------|--------------|----------|
| **Patch** (1.0.X) | Bug fixes, small styling tweaks | Fixed padding inconsistency, adjusted color |
| **Minor** (1.X.0) | New props, new variants, additive changes | Added `size` prop, new IconChip variant |
| **Major** (X.0.0) | Breaking changes, restructured tiers | Renamed props, removed variant, changed interface |

### Cascade Verification Documentation

When a change is propagated, document the cascade result:

```markdown
#### Cascade Verification Log

**Change:** [What was changed in Tier X]
**Date:** [YYYY-MM-DD]

**Successfully Cascaded To:**
- Component A - No changes needed, inherited automatically
- Component B - No changes needed, inherited automatically

**Required Composition Fixes:**
- Component C - Fixed: Now renders <ParentComponent>
- Component D - Fixed: Removed hardcoded padding override

**Still Not Integrated:**
- Component E - Feature-specific variation, intentionally separate
```

This log goes in the system doc after the version history table.

**Log Retention Policy:**
- Keep last 10 cascade verification entries in main log
- Archive older entries to end of document under "## Historical Cascade Logs (Archive)"
- Include link in main log: "See full history in Historical Cascade Logs section"
- Archive format: Same as main log, but collapsed/condensed

### Systems Index Update (if significant)

Only update `~/.config/claude-code/component-system-patterns/systems-index.md` when:

- A new pattern emerges worth remembering
- A notable architectural decision was made
- The change represents a learning for future projects

Most routine updates (padding, colors) do NOT need systems index updates.

---

## Handoff Protocols

### To `component-system-architect`

When a component doesn't belong to any system:

```
This component ([name]) doesn't appear to belong to any existing system.

Options:
1. Should it be added to an existing system? [list candidates]
2. Should we check if a new system is needed?

[If option 2]: I'll hand this to the component-system-architect skill to assess.
```

### From `component-system-architect`

When receiving a newly created system:
- Read the new system doc
- Add it to the known systems list for future proactive detection
- Be ready to handle propagation requests for this new system

---

## Common Scenarios

### Scenario: User edited a Tier 3 component with changes that should propagate

1. Detect the edit is in a Tier 3 file
2. Identify which system it belongs to
3. Analyze what changed vs. parent tier defaults
4. Propose moving the change to appropriate tier
5. Present TLDR with impact
6. Wait for tier approval
7. Execute and update docs

### Scenario: User wants to update "all bottom sheets"

1. Clarify what specific change they want
2. Identify the appropriate tier (likely Tier 1)
3. Show what components will be affected
4. Get approval
5. Make the change in Tier 1
6. Update system doc

### Scenario: Change only makes sense for some components

1. Explain why Tier 1 isn't appropriate
2. Suggest creating a new Tier 2 variant, OR
3. Suggest keeping it in Tier 3 if truly specific
4. Let user decide

---

## Integration vs Mimicking - Glossary

### ✅ TRUE INTEGRATION

**Definition:** Component achieves desired behavior by rendering and composing parent tier components.

**Characteristics:**
- Imports parent component from system
- Returns JSX containing `<ParentComponent>`
- Passes props through to parent
- Minimal/no style overrides
- Changes to parent automatically cascade

**Example:**
```tsx
// RemovableChip - INTEGRATED
import { Chip } from '../primitives/Chip';

export function RemovableChip({ label, onRemove, ...chipProps }) {
  return (
    <Chip {...chipProps}> {/* ✅ Renders parent, forwards props */}
      <Text>{label}</Text>
      <Pressable onPress={onRemove}>
        <Icon name="x" />
      </Pressable>
    </Chip>
  );
}
```

**Test:** If you change Chip's padding tomorrow, RemovableChip updates automatically. ✅

---

### ❌ MIMICKING

**Definition:** Component achieves desired behavior by copying styles/patterns from parent tier without using it.

**Characteristics:**
- May or may not import parent (often just for types)
- Returns JSX using raw React Native primitives
- Duplicates parent's styling constants
- Uses same tokens but applies them locally
- Changes to parent do NOT cascade

**Example:**
```tsx
// RemovableChip - MIMICKING
import { Pressable, View, Text } from 'react-native';
import { useThemedStyles } from '../theme';

export function RemovableChip({ label, onRemove }) {
  const t = useThemedStyles();

  return (
    <Pressable style={{
      borderRadius: t.radii.full,  // ❌ Copied from Chip
      paddingHorizontal: t.spacing.md, // ❌ Copied from Chip
      backgroundColor: t.colors.muted, // ❌ Copied from Chip
    }}>
      <Text>{label}</Text>
      <Pressable onPress={onRemove}>
        <Icon name="x" />
      </Pressable>
    </Pressable>
  );
}
```

**Test:** If you change Chip's padding tomorrow, RemovableChip stays the same. ❌

---

### ⚠️ PARTIAL INTEGRATION

**Definition:** Component renders parent but has blocking overrides preventing full cascade.

**Characteristics:**
- Imports and renders parent component
- BUT: Hardcodes some styles that override parent
- OR: Doesn't forward all relevant props
- Cascade works for some changes but not others

**Example:**
```tsx
// RemovableChip - PARTIAL
import { Chip } from '../primitives/Chip';

export function RemovableChip({ label, onRemove, size }) {
  return (
    <Chip
      // ⚠️ Forgot to pass 'size' prop through
      style={{ paddingHorizontal: 20 }} // ⚠️ Hardcoded override
    >
      <Text>{label}</Text>
      <Pressable onPress={onRemove}>
        <Icon name="x" />
      </Pressable>
    </Chip>
  );
}
```

**Test:** If you change Chip's color, RemovableChip updates. ✅
         If you change Chip's padding, RemovableChip stays 20px. ❌

---

### Decision Rule

**When evaluating a component:**

```
Does it render <ParentComponent>?
├─ NO → ❌ MIMICKING (even if styles match perfectly)
└─ YES → Continue
    │
    Are there hardcoded overrides of parent's base styles?
    ├─ YES → ⚠️ PARTIAL INTEGRATION
    └─ NO → Continue
        │
        Are all relevant props forwarded?
        ├─ NO → ⚠️ PARTIAL INTEGRATION
        └─ YES → ✅ TRUE INTEGRATION
```

**Goal:** Every component should be ✅ TRUE INTEGRATION.
