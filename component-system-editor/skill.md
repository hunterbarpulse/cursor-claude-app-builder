---
name: component-system-editor
description: Diagnoses and fixes component integration failures by ensuring instances properly follow their parent system. Prevents cascade failures by fixing at system level, not consumer level. Proactively detects when components don't properly integrate with their systems. Use when reporting bugs like "TextField looks wrong", "card border doesn't match", or when saying "update system", "apply system-wide", "I'm done with [component]".
---

# Component System Editor

A skill for diagnosing integration failures and propagating fixes through tiered UI component systems. **Every fix prevents the problem from happening again** by addressing root causes at the system level.

---

**Welcome to Component System Editor!**

This skill helps you:
- **Diagnose integration failures** - Why doesn't this instance follow the system?
- **Fix at system level** - Never make manual patches without preventing recurrence
- **Enforce patterns** - Add runtime validation, docs, or linter rules as needed
- **Propagate changes** - Apply fixes at highest tier for maximum cascade
- **Prevent future failures** - Every fix includes prevention mechanism

**CRITICAL PRINCIPLE:** When you report a bug (e.g., "ABV field text looks wrong"), I treat it as a **cascade failure**. The system should have prevented this. I will:
1. Diagnose why the system didn't prevent it
2. Fix it at the system level (not just the instance)
3. Add enforcement to prevent recurrence

---

## Two Workflows

I support two distinct workflows and will **infer which one** based on your description:

### Workflow 1: Integration Failure Fix
**When you report bugs or things "looking wrong"**

Examples:
- "The ABV field text sits too low" → TextField system violation
- "Date picker doesn't open when tapped" → TextField trigger pattern violation
- "Custom items show (custom) instead of icon" → ListPickerSheet pattern violation

**What I do:**
1. Scan `docs/component-systems/` to find which system this belongs to
2. Load system rules (Styling Constants, Usage Guidelines, etc.)
3. Compare instance against system rules → identify violations
4. **Root Cause Analysis:**
   - TypeScript violation bypassed?
   - Pattern missing from docs?
   - System enforcement gap?
5. **Fix + Enforce:**
   - Fix the instance to follow system
   - Fix at system level if needed (e.g., TextField.tsx)
   - Add enforcement: docs + runtime validation + linter rule
   - Update system documentation

### Workflow 2: System Evolution
**When you want to change the system itself**

Examples:
- "Make this card more rounded" → Prototype on one card, propagate to all
- "Change TextField padding" → Update system-wide
- "I'm done with this button, apply changes" → Cascade to system

**What I do:**
1. Monitor your prototype changes on the single instance
2. When you signal completion, analyze tier placement
3. Present TLDR: "Change X at Tier Y will affect Z components"
4. Get approval on tier placement
5. Execute + update documentation

---

## When This Skill Activates

### Proactive Integration Failure Detection

I detect integration failures from your descriptions:

**Integration Failure Triggers:**
- "X looks wrong" / "X doesn't work"
- "Text sits too low" / "Can't interact with X"
- "Border doesn't match" / "Icon is wrong"
- "Y doesn't cascade properly"

**System Evolution Triggers:**
- "Make X more/less [property]"
- "Change X padding/radius/shadow"
- "I'm done with X, propagate"
- "Update the X system"

If ambiguous, I'll ask which workflow you're using.

### Consumer Pattern Detection

When you fix component usage in a consumer file, I will:
1. Detect: "You're editing TextField usage in ProductionSection"
2. Check: Does this belong to a system? → YES, TextField System
3. Analyze: Is this following system patterns? → NO, using `style` not `containerStyle`
4. **Ask: "This looks like an integration failure. Should I:**
   **A) Fix this instance + prevent recurrence system-wide?**
   **B) You're just prototyping, monitor for later propagation?"**

---

## Integration Failure Workflow (Detailed)

### Step 1: System Membership Scan
```
Detected edit in: ProductionSection.tsx (ABV TextField)
Scanning docs/component-systems/...
✓ Found: textfield-system.md
Component belongs to: TextField System (Tier 1)
```

### Step 2: Load System Rules
```
Reading textfield-system.md...

System Rules Extracted:
✓ Styling Constants (must follow)
✓ Usage Guidelines (containerStyle, not style)
✓ Common Usage Patterns
✓ Surface-aware styling requirements
```

### Step 3: Diagnose Violation
```
Comparing instance vs system rules...

VIOLATIONS FOUND:
❌ ABV TextField uses `style` prop (line 168)
   System Rule: Must use `containerStyle` prop
   Source: Usage Guidelines section

❌ Text rendering sits too low
   System should provide: textAlignVertical: 'center'
   Actual: Missing from TextField base component
```

### Step 4: Root Cause Analysis
```
Why didn't the system prevent this?

Issue 1: `style` prop usage
✓ TypeScript DOES block this (Omit<TextInputProps, 'style'>)
→ Root Cause: Developer bypassed TypeScript error
→ Enforcement Gap: No runtime validation

Issue 2: Text alignment
✓ TextField.tsx missing textAlignVertical: 'center'
→ Root Cause: System incomplete (lineHeight caused misalignment)
→ This is a Tier 1 cascade failure
```

### Step 5: Fix + Enforce
```
PROPOSED FIX:

Tier 1 (TextField.tsx):
✓ Add textAlignVertical: 'center' to input styles
✓ Remove lineHeight from single-line inputs
✓ Add lineHeight only to multiline

Tier 1 (TextField.tsx runtime):
✓ Add console.warn() if misuse detected in dev mode

Tier 1 (textfield-system.md docs):
✓ Add to Usage Guidelines with DO/DON'T examples

Consumer (ProductionSection.tsx):
✓ Change style={} → containerStyle={}

Cascade Impact:
✅ All 29 TextField instances get better text alignment
✅ Future misuse will show dev warning
✅ Pattern documented for developers

Approve? [yes/modify]
```

---

## System Evolution Workflow (Detailed)

### Step 1: Monitor Changes
```
Detected: You're editing Card border in ThreeWordDescriptor
System: Card System (card-system.md)
Mode: MONITORING (waiting for completion signal)

Changes detected:
- borderWidth: 1 → removed
- borderColor: removed
- Matches Card variant="elevated" pattern
```

### Step 2: Analyze Tier Placement
```
Signal received: "I'm done, apply changes"

Analyzing what changed...
Change: Removed border from card (now matches elevated pattern)
Current tier: Tier 3 (ThreeWordDescriptor - feature implementation)
Recommended tier: Tier 0 (Card System Docs - pattern clarification)

Reason: This isn't a new pattern, it's correcting usage to match
existing Card system. Document as "Common Mistake" in card-system.md
```

### Step 3: Present TLDR
```
TLDR:
Change: Remove borders from cards (use elevation instead)
Recommended tier: Card System Documentation
Affected: Clarifies pattern for future card usage

Actions:
1. Keep ThreeWordDescriptor fix
2. Add "Common Mistakes" section to card-system.md
3. Document: "Don't add borders to elevated cards"

Is this the right approach? [yes/propagate to more cards]
```

---

## Enforcement Levels

When fixing integration failures, I select appropriate enforcement:

### Level 1: Documentation (Always)
- Add to system .md "Usage Guidelines" or "Common Mistakes"
- Examples with DO/DON'T

### Level 2: Runtime Validation (For Critical Patterns)
- Add console.warn() to base component
- Detects misuse during development
- Example: TextField warns if `style` prop detected

### Level 3: ESLint Rule (Optional)
- Create custom linter rule
- Catches at lint time
- Example: `no-textfield-style-prop`

### Level 4: Enhanced TypeScript (If Needed)
- Better error messages
- Custom type guards

I'll recommend the appropriate level based on:
- How critical is the pattern?
- How easy is it to misuse?
- How often does it happen?

---

## CASCADE FAILURE CHECKLIST

Before suggesting any fix, I MUST answer:

**✓ Is this a cascade failure?**
- Child component not inheriting correctly from parent?
- Instance not following system pattern?
- System rule not being enforced?

**✓ What tier should this fix be at?**
- Tier 1 (base component)? → Affects all instances
- Tier 2 (composed component)? → Affects feature implementations
- Tier 3 (feature)? → Only this instance
- Tier 0 (docs)? → Pattern clarification

**✓ How do we prevent recurrence?**
- Documentation: Usage guidelines with examples
- Runtime: console.warn() in dev mode
- Linting: ESLint rule
- TypeScript: Enhanced type checking

**✓ Did we fix the root cause, not just symptoms?**
- ❌ Fix ABV instance only → Symptom
- ✅ Fix TextField.tsx + docs + validation → Root cause

---

## Instructions

When this skill activates:

1. **Infer workflow** from user description (ask if ambiguous)
2. **If Integration Failure:**
   - Scan `docs/component-systems/` for system membership
   - Load system rules from documentation
   - Compare instance vs rules → diagnose violations
   - Root cause analysis: Why didn't system prevent this?
   - Propose fix at appropriate tier + enforcement
   - Present TLDR with prevention strategy
   - Get approval
   - Execute + update docs

3. **If System Evolution:**
   - Monitor user's prototype changes
   - When signaled, analyze tier placement
   - Present cascade impact TLDR
   - Get tier approval
   - Execute + update docs

4. **Always ask CASCADE FAILURE CHECKLIST questions**
5. **Never suggest manual fixes without prevention**
6. **Every fix must prevent recurrence**

---

## Documentation Updates

After fixes are approved:

1. **System doc** (`docs/component-systems/[system-name].md`)
   - Update version number
   - Add changelog entry
   - Add to Usage Guidelines or Common Mistakes (if new pattern)
   - Update Application Tracking section

2. **Base component** (if Tier 1 fix)
   - Apply fix
   - Add runtime validation if needed
   - Add comments explaining pattern

3. **Consumer instances**
   - Fix violations to follow system
   - Remove manual workarounds

---

## Related Skills

### Handoff to `component-system-architect`
When component doesn't belong to any system, suggest creating one.

### Handoff to `component-system-auditor`
After fixing patterns, audit for other instances with same issue.

---

## Success Criteria

A fix is successful when:
- ✅ Instance now follows system correctly
- ✅ System updated to prevent this failure
- ✅ Documentation explains the pattern
- ✅ Enforcement mechanism in place (runtime/lint/types)
- ✅ Future developers won't make same mistake
