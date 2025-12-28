# Component System Auditor - Reference Document

This document contains search patterns, check procedures, report templates, and decision trees for auditing component systems.

---

## External Audit: Outlier Detection

### Search Strategy by System

#### TextField System
**Tier 1 Signature:**
- File: `components/primitives/TextField.tsx`
- Key props: `searchMode`, `clearable`, `editable`, `surface`
- Styling constants: `borderRadius: t.radii.full`, `minHeight: 48`, `paddingHorizontal: t.spacing.lg`

**Search Patterns:**
- Raw `TextInput` imports from react-native
- Components with pill-shaped styling (borderRadius: 9999 or t.radii.full)
- Input-like Views with minHeight: 48
- Search-related component names not using SearchBar

**Red Flags:**
- Raw `TextInput` not inside TextField.tsx or SearchBar.tsx
- Hardcoded `borderRadius: 9999` on input-like views
- `minHeight: 48` with pill styling (indicates DIY TextField)

---

#### Chip System
**Tier 1 Signature:**
- File: `components/primitives/Chip.tsx`
- Key props: `variant`, `size`, `surface`
- Styling constants: `borderRadius: t.radii.full`, pill shape

**Search Patterns:**
- Custom pill-shaped Pressables with selection state
- Components with `isSelected` and `onSelect` not using Chip
- Badge-like or tag-like components not in the system

**Red Flags:**
- Hardcoded selection colors instead of `variant="selected"`
- Custom pressable pills with background color changes on press
- Components named "tag", "badge", "filter" not using Chip

---

#### Card System
**Tier 1 Signature:**
- File: `components/primitives/Card.tsx`
- Key props: `variant`, `pressable`, `bordered`
- Styling constants: `borderRadius: t.radii.lg`, `backgroundColor: t.colors.card`

**Search Patterns:**
- Raw `View` with `backgroundColor: t.colors.card` + `borderRadius`
- Missing `SurfaceProvider` inside card-like containers
- Hardcoded shadow values instead of `t.elevation.*`

**Red Flags:**
- Views styled to look like cards but not using Card component
- Card-like containers without SurfaceProvider (surface-aware children will break)
- Custom shadow implementations

---

#### Bottom Sheet System
**Tier 1 Signature:**
- File: `components/layout/BottomSheetContainer.tsx`
- Key props: `visible`, `onClose`, `title`, `headerIcon`
- Patterns: backdrop scrim, drag-to-dismiss, keyboard avoidance

**Search Patterns:**
- Custom modals not using BottomSheetContainer
- DIY drag-to-dismiss implementations
- Missing "journal pace" animation timing (350ms)

**Red Flags:**
- Modal components with custom backdrop/scrim
- Animated sheets not following 350ms timing
- Sheets without proper keyboard avoidance

---

### Outlier Classification

When an outlier is found, classify it:

| Classification | Description | Recommended Action |
|----------------|-------------|-------------------|
| **Direct Migrate** | Can use existing tier component as-is | Replace with tier component |
| **Needs Variant** | Needs a prop/variant that doesn't exist | Add prop to tier 1 OR create tier 2 |
| **Needs New Tier** | Unique pattern warranting new tier 2 | Hand off to architect |
| **False Positive** | Intentionally different (document why) | Add to exceptions list |

---

### Outlier Report Template

```
## External Audit Report: [System Name] System

**Scanned:** [date]
**Outliers found:** [count]

---

### Outlier 1: [Component Name]

**Location:** `[file path]`
**Used in:** [list of screens/features]

**Issue (plain language):**
This [component], used in [locations], isn't cooperating because [it uses raw TextInput/View/etc. instead of the [System] component].

**Classification:** [Direct Migrate / Needs Variant / Needs New Tier]

**Recommendation:**
[Plain-language fix description]

**Integration Method Required:**
- [ ] Structural Replacement: Replace entire component JSX with <SystemComponent>
- [ ] Compositional Wrapping: Wrap existing JSX inside <SystemComponent>
- [ ] Prop Forwarding: Component renders <SystemComponent> and forwards props

**Post-Fix Verification Checklist:**
1. [ ] Component imports the tier component
2. [ ] Component renders it in JSX (not just type usage)
3. [ ] No hardcoded style overrides blocking cascade
4. [ ] Relevant props are passed through
5. [ ] Changes to tier component will automatically cascade

**Example Fix (code):**
```tsx
// BEFORE (mimicking):
export function SearchInput() {
  return (
    <TextInput style={{ borderRadius: 9999, padding: 16 }} />
  );
}

// AFTER (integrated):
import { TextField } from '../primitives/TextField';

export function SearchInput(props) {
  return <TextField searchMode {...props} />;
}
```
```

---

## Internal Audit: Deep Scan Checks

### Category A: Inheritance & Composition Issues

| # | Check Name | What Goes Wrong | Plain-Language Example |
|---|------------|-----------------|------------------------|
| 1 | **Wrapper doesn't use base** | Tier 2 imports Tier 1 but renders a raw View/Pressable instead | "RemovableChip imports Chip but never actually uses it - it builds its own chip-like button from scratch" |
| 1b | **Component actually renders parent tier** | Tier 2/3 has matching styles but doesn't render parent component | "RemovableChip has all the same padding and colors as Chip, but it builds its own Pressable from scratch instead of wrapping <Chip>" |
| 1c | **Import is decorative only** | Component imports parent tier but only uses it for types/props, not rendering | "SearchBar imports TextField for the type definition, but the actual return statement uses raw TextInput - it's cosplaying as integrated" |
| 2 | **Prop not passed through** | Tier 1 has a prop (like `size`) that Tier 2 doesn't expose | "You can set `size` on Chip, but RemovableChip doesn't let you change the size - it's stuck on medium" |
| 3 | **Prop passed but ignored** | Tier 2 accepts a prop but doesn't forward it to Tier 1 | "VocabularyChips accepts a `surface` prop but never passes it to the chips inside - they always use the wrong background" |
| 4 | **Prop overridden silently** | Tier 2 always sets a prop to a fixed value, ignoring what you pass | "SearchBar always forces `clearable=true` even if you set it to false" |
| 5 | **Wrong component composed** | Tier 3 uses Tier 1 directly when it should use Tier 2 | "CountrySelector uses Chip directly instead of RemovableChip, so there's no X button to remove selections" |

**Enhanced Detection for Checks #1, #1b, #1c:**

**Check #1 - Wrapper doesn't use base:**
1. Verify component imports parent tier component
2. Grep return statement for `<ParentComponent`
3. If import exists but component name not in return → Flag as Check #1

**Check #1b - Component actually renders parent tier:**
1. Search JSX return for parent component usage (e.g., search for `<Chip`)
2. If not found, check if component has matching style patterns to parent
3. If styles match but no parent render → Flag as Check #1b (mimicking)

**Check #1c - Import is decorative only:**
1. Check if import statement exists
2. Check if it's a type-only import (`import type { ... }`)
3. If type-only OR import exists but component not rendered in JSX → Flag as Check #1c

---

### Category B: Styling & Token Issues

| # | Check Name | What Goes Wrong | Plain-Language Example |
|---|------------|-----------------|------------------------|
| 6 | **Hardcoded value instead of token** | Uses `borderRadius: 24` instead of `t.radii.lg` | "WineCard uses `borderRadius: 24` directly instead of the theme token - if you change the design system's radius, this card won't update" |
| 7 | **Duplicated style definition** | Same styling appears in both Tier 1 and Tier 2 | "The padding is defined in both Chip AND RemovableChip - if you change Chip's padding, RemovableChip still uses its own value" |
| 8 | **Style override blocks cascade** | Tier 2 defines a style that Tier 1 already handles | "DimensionChipList sets its own font size instead of letting Chip control it" |
| 9 | **Inconsistent token usage** | Uses `t.spacing.md` in one place and `12` (same value) in another | "VintagePicker uses `12` for padding while TextField uses `t.spacing.md` - they happen to match now but could drift apart" |
| 10 | **Wrong token used** | Uses a token but it's the wrong one for this context | "SearchHeader uses `t.colors.card` for its background when it should use `t.colors.muted` like TextField" |

---

### Category C: Surface & Context Issues

| # | Check Name | What Goes Wrong | Plain-Language Example |
|---|------------|-----------------|------------------------|
| 11 | **Missing SurfaceProvider** | Surface-aware component inside a View instead of Card | "This TextField is inside a View styled like a card, but there's no SurfaceProvider so it can't detect it's on a card surface" |
| 12 | **Surface prop not forwarded** | Parent accepts `surface` but doesn't pass to children | "TagSelector accepts a surface prop but the chips inside always assume they're on a background" |
| 13 | **Hardcoded surface assumption** | Component always assumes one surface type | "FilterSheet always uses 'background' colors even when placed inside a Card" |
| 14 | **Context provider missing in tests** | Component works in app but tests fail because no context | "Chip tests fail because there's no ThemeProvider wrapping the test render" |

---

### Category D: Migration Leftovers

| # | Check Name | What Goes Wrong | Plain-Language Example |
|---|------------|-----------------|------------------------|
| 15 | **Partial migration** | Some instances migrated, others forgotten | "3 out of 5 search inputs now use TextField, but 2 screens still use the old custom TextInput" |
| 16 | **Dead import** | Old component imported but not used | "WineInfoTab still imports the old `Input` component even though it uses TextField now" |
| 17 | **Commented-out old code** | Migration left behind commented code | "There's 50 lines of commented-out old chip styling in VocabularyChips" |
| 18 | **Duplicate component file** | Old version of component still exists | "There's both `Chip.tsx` and `OldChip.tsx` in the codebase - some files use one, some use the other" |
| 19 | **Inconsistent naming** | Same concept has different names in different places | "It's called `SearchBar` in cellar but `SearchField` in insights - should be unified" |

---

### Category E: Prop & API Mismatches

| # | Check Name | What Goes Wrong | Plain-Language Example |
|---|------------|-----------------|------------------------|
| 20 | **Missing required prop** | Tier 1 requires a prop that Tier 2 doesn't provide | "Card requires `children` but SectionCard can render with no children, causing errors" |
| 21 | **Type mismatch** | Tier 2 accepts different types than Tier 1 | "Chip accepts `variant` as a string union, but FilterChips passes a boolean for `isSelected`" |
| 22 | **Default value mismatch** | Tier 2 has different defaults than documented | "TextField docs say `clearable` defaults to false, but SearchBar sets it to true without documenting this" |
| 23 | **Callback signature mismatch** | `onPress` vs `onSelect` vs `onChange` inconsistency | "Chip uses `onPress` but RemovableChip uses `onRemove` - confusing which to use" |

---

### Category F: Documentation Drift

| # | Check Name | What Goes Wrong | Plain-Language Example |
|---|------------|-----------------|------------------------|
| 24 | **Undocumented component** | A Tier 2/3 component exists but isn't in the system docs | "FilterChips is used in the app but isn't listed in the Chip System documentation" |
| 25 | **Outdated prop table** | Doc lists props that don't exist anymore | "The TextField docs mention a `multiline` prop that was removed 2 versions ago" |
| 26 | **Missing from diagram** | Component exists and works but not in the visual hierarchy | "DimensionChipList is documented in the table but missing from the ASCII diagram" |
| 27 | **Wrong tier assignment** | Component is in wrong tier in docs | "VocabularyChips is listed as Tier 2 but it's actually Tier 3 (feature-specific)" |

---

### Category G: Behavioral Issues

| # | Check Name | What Goes Wrong | Plain-Language Example |
|---|------------|-----------------|------------------------|
| 28 | **Event not bubbling** | Child handles event but parent also needs it | "Pressing a chip in TagSelector selects it but doesn't trigger the parent's `onChange`" |
| 29 | **Animation mismatch** | Different timings/easings across tiers | "Cards use 200ms transitions but ListCard uses 350ms - feels inconsistent" |
| 30 | **Accessibility props dropped** | `accessibilityLabel` not passed through | "You set an accessibility label on WineCard but the inner Card doesn't receive it" |
| 31 | **Loading state not inherited** | Tier 1 has loading but Tier 2 doesn't use it | "Card has a loading skeleton but SectionCard built its own loading spinner" |

---

### Category H: State & Data Flow Issues

| # | Check Name | What Goes Wrong | Plain-Language Example |
|---|------------|-----------------|------------------------|
| 32 | **Parent-child state conflict** | Both parent and child manage the same state | "DimensionChipList tracks `isExpanded` internally, but if a parent also tries to control visibility, they fight each other" |
| 33 | **Prop spreading overwrites** | `...props` spreads override component's own values | "RemovableChip spreads props onto Chip, but if someone passes `variant` in those props, it overrides the component's selection logic" |
| 34 | **Default value not applied** | Tier 2 destructures with default but doesn't pass to Tier 1 | "Tier 2 has `size = 'md'` default, but passes `{...props}` to Tier 1 without including size - Tier 1 never gets the default" |
| 35 | **Conditional prop missing in branch** | Required prop only passed in some code paths | "The `surface` prop is passed when `isSelected` is true, but forgotten in the else branch" |
| 36 | **Null/undefined prop bypass** | Passing `undefined` explicitly bypasses Tier 1 defaults | "Passing `variant={undefined}` is different from not passing variant at all - can cause unexpected behavior" |

---

### Category I: Accessibility Deep Issues

| # | Check Name | What Goes Wrong | Plain-Language Example |
|---|------------|-----------------|------------------------|
| 37 | **Accessibility role changed** | Tier 2 overrides semantic role incorrectly | "Chip says it's a 'button' but RemovableChip makes it a 'checkbox' - screen readers get confused" |
| 38 | **Accessibility hint lost** | `accessibilityHint` not passed through | "Card has a helpful hint for screen readers, but WineCard doesn't pass it along" |
| 39 | **Touch target too small** | Tier 2/3 reduces clickable area below 44px | "The small chip variant ends up 30px tall - hard to tap for users with motor difficulties" |
| 40 | **Accessible false blocks children** | Parent sets `accessible={false}` hiding children from screen readers | "A wrapper View has accessible=false, so all the chips inside are invisible to VoiceOver" |

---

### Category J: Type & API Compatibility Issues

| # | Check Name | What Goes Wrong | Plain-Language Example |
|---|------------|-----------------|------------------------|
| 41 | **Variant subset restriction** | Tier 2 only allows some of Tier 1's variants | "Chip has 4 variants, but FilterChips only ever uses 2 of them - the other 2 are impossible to access" |
| 42 | **Callback argument mismatch** | Callback called with different arguments than expected | "Parent passes `onPress={() => doSomething()}` but component calls `onPress(item)` with an argument that gets ignored" |
| 43 | **Union type narrowed silently** | Tier 2 accepts narrower type without documenting | "Chip accepts `size: 'sm' | 'md' | 'lg'` but RemovableChip only works with 'sm' and 'md' - passing 'lg' breaks silently" |
| 44 | **Optional becomes required** | Tier 1 prop is optional but Tier 2 requires it | "TextField's `placeholder` is optional, but SearchBar crashes without it" |

---

### Category K: Hook & Utility Dependencies

| # | Check Name | What Goes Wrong | Plain-Language Example |
|---|------------|-----------------|------------------------|
| 45 | **Hook dependency missing** | Custom hook changes break components silently | "All chips use `useSurface()` hook - if someone changes that hook, every chip breaks but the auditor won't catch it as a 'chip issue'" |
| 46 | **Utility function drift** | Shared utility changes affect multiple components | "The `withOpacity()` helper is used by Chip, Card, and Badge - changing it breaks all three" |
| 47 | **Theme context assumed** | Component crashes if not wrapped in ThemeProvider | "RemovableChip calls `useTheme()` but doesn't check if theme exists - crashes in tests without provider" |
| 48 | **Cleanup function missing** | useEffect without cleanup causes memory leaks | "Component adds an event listener in useEffect but never removes it when unmounting" |

---

### Category L: Performance Issues

| # | Check Name | What Goes Wrong | Plain-Language Example |
|---|------------|-----------------|------------------------|
| 49 | **Inline style object** | Style object created every render | "Component has `style={{ color: theme.colors.primary }}` inline - creates new object every render, causing unnecessary re-renders" |
| 50 | **Missing React.memo** | Expensive component re-renders unnecessarily | "WineCard is complex but not memoized - parent re-rendering causes all WineCards to re-render even if their props didn't change" |
| 51 | **Inline callback function** | Callback defined inline breaks memoization | "Component has `onPress={() => handlePress()}` - new function every render, breaks child memoization" |
| 52 | **Dependency array incomplete** | useCallback/useMemo dependencies missing | "The debounced callback doesn't list `value` in dependencies - uses stale value" |

---

### Category M: Conditional Logic Traps

| # | Check Name | What Goes Wrong | Plain-Language Example |
|---|------------|-----------------|------------------------|
| 53 | **Style merge order wrong** | User style overrides when it shouldn't (or vice versa) | "Component does `[styles.base, userStyle]` but for disabled state should be `[styles.base, userStyle, styles.disabled]` so disabled always wins" |
| 54 | **Ternary precedence bug** | Ternary operator evaluates incorrectly | "Code has `condition ? styles.a : condition2 && styles.b` - when condition2 is true, it returns `true` not the style object" |
| 55 | **Unreachable variant branch** | Some variants can never be reached | "The if-else chain handles 'primary' twice in different branches - second one never runs" |
| 56 | **Boolean coercion in style array** | False/null in style array causes issues | "`[styles.base, isActive && styles.active]` - when isActive is false, array contains `false` which RN handles but is messy" |

---

### Category N: Composition Verification (POST-FIX)

These checks run AFTER a fix is applied to verify true integration:

| # | Check Name | What to Verify | Plain-Language Example |
|---|------------|----------------|------------------------|
| 64 | **Import verification** | Component file contains import statement | "After fix, RemovableChip.tsx should have `import { Chip } from ...`" |
| 64a | **Import is value not type** | Import is value import, not type-only | "Should be `import { Chip }` not `import type { Chip }` - need actual component, not just types" |
| 65 | **JSX usage verification** | Return statement contains `<ParentComponent` | "The return statement should have `<Chip` not `<Pressable`" |
| 66 | **No style mimicking** | Component doesn't duplicate parent's base styles | "RemovableChip shouldn't have its own borderRadius or padding - let Chip handle that" |
| 67 | **Prop forwarding exists** | Relevant props passed to parent component | "If RemovableChip accepts `size`, it should pass it to Chip" |
| 68 | **Cascade readiness** | Changing parent will affect this component | "If I change Chip's padding tomorrow, RemovableChip should update automatically" |

**Detection Details for Check #66 (No style mimicking):**

1. Extract parent component's base style properties from system doc
2. Check if child component defines the SAME properties inline
3. **IGNORE (allowed):**
   - Additive styles (child adds properties parent doesn't have)
   - Intentional overrides for tier-specific behavior
4. **FLAG (mimicking):**
   - Child hardcodes value that parent provides
   - Child uses token for property parent already handles
   - Example: Parent has `paddingHorizontal: t.spacing.md`, child also defines `paddingHorizontal: t.spacing.md` ❌

**When to Run Category N:**
- ALWAYS after applying an outlier migration fix
- ALWAYS after applying a composition issue fix (Category A)
- Before marking a fix as "completed"

**If Category N Check Fails:**
- DO NOT mark fix as complete
- Report: "Fix applied but verification failed - component is mimicking, not integrating"
- Provide guidance to user on correcting the integration

**If Category N Results in PARTIAL Integration:**
1. Mark verification as "INCOMPLETE"
2. Report to user:
   - "Fix applied but component has partial integration"
   - List specific issues (missing prop forwarding, hardcoded override, etc.)
3. Ask user:
   - "Fix partial integration issues now?" → Continue with additional fixes
   - "Accept as-is for now?" → Log as known partial integration in system doc, proceed with warning

---

## Cross-System Audit Checks

When a component uses multiple systems (e.g., a Card containing Chips and TextFields), run these additional checks:

### Cross-System Issues

| # | Check Name | What Goes Wrong | Plain-Language Example |
|---|------------|-----------------|------------------------|
| 57 | **Token value drift between systems** | Same token means different things | "TextField uses `t.spacing.lg` (16px) for padding, Button uses it too but looks misaligned when side-by-side" |
| 58 | **Surface context not bridging** | Card provides surface but nested system component doesn't use it | "Card sets surface='card', but the TextField inside uses `useSurface()` which doesn't detect it because there's a plain View wrapper in between" |
| 59 | **Animation timing clash** | Different systems animate at different speeds | "BottomSheet opens at 350ms but the Card inside fades in at 200ms - looks janky" |
| 60 | **Typography inconsistency** | Different font handling across systems | "Button uses hardcoded 'Inter_500' but Chip uses `t.typography.bodyMd` - if typography tokens change, Button stays the same" |
| 61 | **Color semantic conflict** | Same color name means different things | "Chip uses 'accent' variant for 'selected', Button uses 'accent' for 'hero action' - combining them is confusing" |
| 62 | **Icon spacing inconsistency** | Icons spaced differently across systems | "TextField's left icon has 8px gap, Button's icon has 12px gap - combined UI looks uneven" |
| 63 | **Z-index layering conflict** | Overlapping components fight for visibility | "BottomSheet is z-index 10, but a Badge inside uses z-index 20 - Badge appears above sheet overlay" |

---

## Additional Search Patterns

### Token Reference Patterns (Catch Inconsistent Usage)

Search for these patterns that indicate token bypass:

| Pattern | Issue |
|---------|-------|
| `theme.colors.X` outside token system | Should use `t.colors.X` for consistency |
| `useTheme().colors.X` inline | Should use `useThemedStyles` pattern |
| Hardcoded hex values `#FFFFFF` | Should use token like `t.colors.background` |
| Hardcoded numbers `borderRadius: 24` | Should use token like `t.radii.lg` |
| `fontSize: 17` | Should use `t.typography.bodyMd.fontSize` |
| `fontFamily: 'Inter'` | Should use `t.typography.fontSans` |

### Prop Spreading Danger Patterns

Look for risky prop spreading:

| Pattern | Risk |
|---------|------|
| `<Tier1 {...props} />` at end | Props can override component's own values |
| `<Tier1 {...props} style={...} />` | User's style prop is lost |
| `{...accessibilityProps}` not included | Accessibility props dropped |
| `const { dangerous, ...rest } = props` | Need to verify `dangerous` props are filtered |

---

## Decision Trees

### Should This Be an Outlier?

```
Is the component using raw RN primitives (View, TextInput, Pressable)?
├── NO → Probably not an outlier (already using some abstraction)
└── YES → Continue
    │
    Does a system component exist for this pattern?
    ├── NO → Not an outlier (no system to join)
    └── YES → Continue
        │
        Is the styling similar to the system component?
        ├── NO → Check if it's intentionally different
        │   ├── YES (intentional) → Add to exceptions, document why
        │   └── NO (accidental) → OUTLIER - recommend migration
        └── YES → OUTLIER - definitely should use system component
```

### Which Tier Should It Join?

```
Is the behavior identical to an existing component?
├── YES → Direct migration to that tier
└── NO → Continue
    │
    Could the behavior be added via a new prop on tier 1?
    ├── YES → Recommend prop addition to tier 1
    └── NO → Continue
        │
        Is it a variant of an existing tier 2?
        ├── YES → Could be tier 3 under that tier 2
        └── NO → Continue
            │
            Is it a general pattern usable across features?
            ├── YES → New tier 2 candidate
            └── NO → Tier 3 (feature-specific)
```

### Is This a Cascade Blocker?

```
Does the issue prevent tier 1 changes from reaching this component?
├── YES → CRITICAL - cascade blocker
└── NO → Continue
    │
    Does the issue cause visual inconsistency?
    ├── YES → MODERATE - styling deviation
    └── NO → Continue
        │
        Does the issue make maintenance harder?
        ├── YES → MINOR - code quality issue
        └── NO → Not an issue (false positive)
```

### Post-Fix Integration Verification

```
Fix has been applied. Is the component truly integrated?
│
├─ Does the component file import the parent tier component?
│  ├── NO → ❌ MIMICKING - Add import and use component
│  └── YES → Continue
│      │
│      ├─ Is it a type-only import (import type { ... })?
│      │  ├── YES → ❌ MIMICKING - Change to value import
│      │  └── NO → Continue
│      │      │
│      │      ├─ Does the return statement render the parent component?
│      │      │  ├── NO → ❌ MIMICKING - Replace JSX to use parent
│      │      │  └── YES → Continue
│      │      │      │
│      │      │      ├─ Are there hardcoded style overrides of parent's base styles?
│      │      │      │  ├── YES → ⚠️  PARTIAL - Remove overrides
│      │      │      │  └── NO → Continue
│      │      │      │      │
│      │      │      │      ├─ Are relevant props forwarded to parent?
│      │      │      │      │  ├── NO → ⚠️  PARTIAL - Add prop forwarding
│      │      │      │      │  └── YES → ✅ INTEGRATED
```

---

## Integrity Report Template

```
## Internal Audit Report: [System Name] System

**Scanned:** [date]
**Components checked:** [count]
**Issues found:** [count] ([critical], [moderate], [minor])

---

### Issue 1: [Component Name] - [Issue Type]

**Severity:** [Critical / Moderate / Minor]
**Category:** [A-G: Category Name]
**Location:** `[file path]`

**Issue (plain language):**
This [component] isn't cooperating because [plain-language explanation].

**Impact:**
If not fixed: [what breaks or won't cascade properly]

**Recommended Fix:**
[Plain-language fix description]

**Handoff:** [None / Editor / Architect]
```

---

## Handoff Protocols

### To component-system-editor

When a fix requires updating a tier component and cascading changes:

```
## Handoff: Auditor → Editor

**Issue:** [description]
**Component:** [name] in [file]
**Fix required:** Update [prop/style] in [tier X]

The fix needs to be applied at [Tier X: ComponentName] and will cascade to:
- [Child 1]
- [Child 2]
- [Child 3]

Handing off to component-system-editor for tier-aware propagation.
```

### To component-system-architect

When an outlier needs a new tier component:

```
## Handoff: Auditor → Architect

**Outlier:** [component name]
**Location:** `[file path]`
**Pattern:** [description of what it does]

This outlier needs a new tier 2 variant in the [System] system because [reason it can't use existing components].

Suggested tier 2 name: [SuggestedName]
Suggested additions to tier 1: [if any]

Handing off to component-system-architect for variant creation.
```

### From component-system-architect

When architect creates a new system:

```
## Post-Creation Audit Prompt

The [System Name] system has been created with:
- Tier 1: [ComponentName] (`[path]`)
- Tier 2: [Variants] (`[paths]`)

Would you like me to audit for outliers that should be part of this new system?

- Reply "yes" to run external audit
- Reply "no" to skip
```

---

## Documentation Update Template

After approved fixes, update the system doc's Version History:

**File:** `docs/component-systems/[system-name].md`

**Add row to Version History table:**

```markdown
| [new version] | [today's date] | [description of fix] |
```

**Example entries:**

```markdown
| 1.3.2 | 2025-12-27 | Fixed VocabularyChips not passing surface prop to chips |
| 1.4.0 | 2025-12-27 | Migrated LocationPickerSheet search to use TextField; added to Tier 3 |
| 1.3.3 | 2025-12-27 | Removed dead Input import from WineInfoTab |
```

**Versioning:**
- Patch (+0.0.1): Bug fixes, cascade blockers, dead code removal
- Minor (+0.1.0): New component added to system, outlier integrated
- Major (+1.0.0): Significant restructuring (rare from auditor)

---

## Exceptions List

Components intentionally outside systems (document the reason):

| Component | System Affinity | Reason for Exception |
|-----------|-----------------|---------------------|
| TokenDisplay | Card | Development-only debug component |
| [Add as discovered] | | |

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
