<!-- Powered by BMAD™ Core -->
# manage-follow-ups

View and manage follow-up items tracked in the QA system.

## Purpose

Display and optionally update the follow-up items tracker (`docs/qa/follow-ups.yml`) which tracks low-severity issues that need follow-up attention.

## Inputs

```yaml
optional:
  - action: 'list' | 'update' | 'status' # Default: 'list'
  - status_filter: 'pending' | 'all' | 'resolved' # For list action
  - item_id: '{id}' # For status update (e.g., 'SEC-001')
  - new_status: 'pending' | 'in_progress' | 'resolved' | 'waived' # For status update
```

## Process

### List Follow-ups (Default)

1. Read `docs/qa/follow-ups.yml` from `qa.qaLocation` (from core-config.yaml)
2. Filter by status if provided
3. Display in readable format:
   - Item ID, title, story
   - Category, severity, status
   - Description and suggested action
   - Source (gate/story) and creation date

### Update Follow-ups

1. Run the update script: `pnpm qa:update-follow-ups`
2. This regenerates the tracker from all gate files and story files
3. Display summary of items found

### Update Item Status

1. Read `docs/qa/follow-ups.yml`
2. Find item by ID
3. Update status field
4. Write back to file
5. Confirm update

## Output

### List Output

```markdown
📋 Follow-up Items (X pending)

1. [SEC-001] RLS enforcement verification needed
   Story: 1.3 - Implement Authentication & Session Management
   Category: security | Severity: low | Status: pending
   Description: [description]
   Action: [suggested_action]
   Source: gate | Created: [date]
   Gate: docs/qa/gates/1.3-*.yml
```

### Update Output

```markdown
✓ Updated follow-ups tracker with X items
  - Pending: Y
  - In Progress: Z
  - Resolved: W
```

## Safety Guarantees

- **Read-only by default** - Only reads follow-ups.yml unless explicitly updating status
- **No story file modifications** - Never touches story files
- **No gate file modifications** - Never modifies existing gate files
- **Only updates follow-ups.yml** - Single file write, isolated from BMAD core
- **Non-blocking** - Doesn't affect story status or workflow

## Usage Examples

- `follow-ups` or `follow-ups list` - Show all pending items
- `follow-ups list --status all` - Show all items including resolved
- `follow-ups update` - Regenerate tracker from gate/story files
- `follow-ups status SEC-001 resolved` - Mark item as resolved

## Key Principles

- Utility command for tracking, not core workflow
- Safe to use anytime - doesn't interfere with reviews
- Helps ensure nothing gets forgotten
- Complements existing QA processes

