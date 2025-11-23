---
name: linear-integration
description: Access and manage Linear tasks including reading issue status, updating tasks, and creating new issues. Use when you need to interact with the user's Linear workspace for task management and project tracking. CRITICAL - Always use batch approval pattern for write operations.
---

# Linear Integration Skill

This skill helps you interact with the user's Linear workspace using the Linear MCP server. Supports both READ operations (safe, no approval needed) and WRITE operations (requires batch approval pattern).

## When to Use This Skill

Activate this skill when:
- User asks about their Linear tasks or issues
- Commands like `/morning-brief` or `/evening-sync` need task status
- User wants to update task status or create new issues
- You need to get project information from Linear
- Cross-referencing tasks with Obsidian notes

## ⚠️ CRITICAL: Safety Pattern for Write Operations

**NEVER update or create Linear issues without explicit user approval.**

### Batch Approval Pattern (MANDATORY for writes)

1. ✅ **Analyze**: Review data and determine what should change
2. ✅ **Propose**: Present proposed changes in clear checklist format
3. ✅ **Wait**: Explicitly ask for user confirmation
4. ✅ **Confirm**: User must say "yes" / "approve" / "go ahead"
5. ✅ **Execute**: Only then call Linear write operations
6. ✅ **Verify**: Confirm successful updates

**Example**:
```markdown
## 📝 Proposed Linear Updates

### Tasks to Mark Complete
- [ ] [PROJ-123] "Implement auth" - Evidence: completion note
- [ ] [PROJ-124] "API review" - Evidence: review doc finalized

**Review these changes. Would you like me to apply them?**
```

## Core Workflow

### 1. Determine Operation Type

**READ Operations** (safe, no approval needed):
- Get user's tasks
- Get project details
- Search for issues
- Get issue details
- List teams, labels, projects

**WRITE Operations** (require batch approval):
- Update issue status
- Create new issue
- Update issue fields
- Create comments

### 2. Choose the Right Tool

**Quick Reference** (most common operations):

| Need | Tool | Requires Approval |
|------|------|-------------------|
| Get my tasks | `list_issues` | ❌ No |
| Get project issues | `list_issues` | ❌ No |
| Get issue details | `get_issue` | ❌ No |
| Search issues | `list_issues` + filters | ❌ No |
| Update issue | `update_issue` | ✅ YES |
| Create issue | `create_issue` | ✅ YES |
| Add comment | `create_comment` | ✅ YES |

## Quick Patterns (READ Operations)

These patterns are safe and don't require approval:

### Pattern 1: Get My In-Progress Tasks

```
# Get issues assigned to me that are in progress
mcp__linear__list_issues(
    assignee="me",
    state="In Progress"
)

# Returns: List of in-progress issues with details
# Use for: Morning briefs, checking active work
```

### Pattern 2: Get All My Tasks

```
# Get all issues assigned to me (any state)
mcp__linear__list_issues(
    assignee="me"
)

# Returns: All your issues across all states
# Use for: Overall workload view
```

### Pattern 3: Get Project Issues

```
# Get all issues for a specific project
mcp__linear__list_issues(
    project="Auth Service"
)

# Or by label
mcp__linear__list_issues(
    label="auth-service"
)

# Returns: All project issues
# Use for: Project briefs, status reviews
```

### Pattern 4: Get Issue Details

```
# Get full details of a specific issue
mcp__linear__get_issue(
    id="PROJ-123"
)

# Returns: Complete issue details including:
# - Title, description, status
# - Assignee, priority, labels
# - Comments, attachments
# - Git branch name (if connected)

# Use for: Deep dives, understanding task context
```

### Pattern 5: Get Project Details

```
# Get project information
mcp__linear__get_project(
    query="Auth Service"
)

# Returns: Project details including:
# - Name, description, status
# - Lead, members, timeline
# - Progress metrics

# Use for: Project briefs, status reports
```

### Pattern 6: List Teams

```
# Get all teams in workspace
mcp__linear__list_teams()

# Returns: List of teams with names and IDs
# Use for: Understanding workspace structure
```

### Pattern 7: List Issue Labels

```
# Get available issue labels
mcp__linear__list_issue_labels()

# Or for specific team
mcp__linear__list_issue_labels(
    team="Engineering"
)

# Returns: Available labels
# Use for: Understanding categorization
```

## WRITE Operations (Require Approval)

These operations modify Linear and MUST use batch approval pattern:

### Pattern: Update Issue Status

**Step 1 - Analyze & Propose**:
```markdown
## 📝 Proposed Linear Updates

I found evidence that these tasks should be marked complete:

### Tasks to Mark Complete
- [ ] [PROJ-123] "Implement auth service"
      Evidence: "Auth Service Implementation.md" shows completion
      Current status: In Progress → Proposed: Done

**Review this change. Would you like me to apply it?**
```

**Step 2 - Wait for User Approval**

User must say: "yes" / "approve" / "apply it" / "go ahead"

**Step 3 - Execute After Approval**:
```
# Only after user confirms
mcp__linear__update_issue(
    id="PROJ-123",
    state="Done"
)

# Confirm success
"✅ Updated PROJ-123 to Done"
```

### Pattern: Create New Issue

**Step 1 - Analyze & Propose**:
```markdown
## 📝 Proposed Linear Updates

I found work in your notes that's not tracked:

### New Issues to Create
- [ ] "Fix auth token expiry bug"
      Source: "Testing Session.md" - bug discovered today
      Team: Engineering
      Labels: bug, auth-service
      Priority: High

**Review this. Would you like me to create this issue?**
```

**Step 2 - Wait for Approval**

**Step 3 - Execute After Approval**:
```
# Only after user confirms
mcp__linear__create_issue(
    team="Engineering",
    title="Fix auth token expiry bug",
    description="Bug discovered during testing session on [date]. Token expires too early causing auth failures...",
    labels=["bug", "auth-service"],
    priority=1  # Urgent
)

# Confirm success with issue ID
"✅ Created issue PROJ-456: Fix auth token expiry bug"
```

### Pattern: Mark Issue Blocked

**Step 1 - Analyze & Propose**:
```markdown
## 📝 Proposed Linear Updates

### Tasks to Mark Blocked
- [ ] [PROJ-125] "Deploy staging environment"
      Evidence: Note mentions "blocked by infra team"
      Current status: In Progress → Proposed: Blocked

**Apply this update?**
```

**Step 2-3 - Wait & Execute**:
```
# After approval
mcp__linear__update_issue(
    id="PROJ-125",
    state="Blocked"
)
```

## MCP Tools Available

### Read Tools (Safe)

**`list_issues`**
- Purpose: Get issues with filtering
- Key Parameters:
  - `assignee`: "me" | user email | user name
  - `state`: "In Progress" | "Done" | "Blocked" | etc.
  - `project`: Project name or ID
  - `label`: Label name or ID
  - `team`: Team name or ID
  - `query`: Text search in title/description
  - `limit`: Max results (default 50, max 250)
- Returns: List of issues with metadata
- Common filters: assignee="me", state="In Progress"

**`get_issue`**
- Purpose: Get full details of specific issue
- Parameters:
  - `id`: Issue ID (e.g., "PROJ-123")
- Returns: Complete issue object with:
  - Title, description, status, assignee
  - Priority, labels, comments
  - Git branch name, attachments
- Use for: Deep dives, understanding context

**`get_project`**
- Purpose: Get project details
- Parameters:
  - `query`: Project name or ID
- Returns: Project information, progress, team
- Use for: Project briefs, status reports

**`list_projects`**
- Purpose: Get all projects in workspace
- Parameters:
  - `team`: Filter by team (optional)
  - `state`: Filter by state (optional)
  - `member`: Filter by member (optional)
- Returns: List of projects
- Use for: Discovering projects, workspace overview

**`list_teams`**
- Purpose: Get all teams
- Parameters: None (or filtering options)
- Returns: List of teams with names and IDs
- Use for: Understanding workspace structure

**`list_issue_labels`**
- Purpose: Get available issue labels
- Parameters:
  - `team`: Filter by team (optional)
- Returns: Available labels
- Use for: Understanding categorization

**`list_issue_statuses`**
- Purpose: Get available status options for a team
- Parameters:
  - `team`: Team name or ID
- Returns: List of valid status names
- Use for: Knowing what states you can set

**`list_comments`**
- Purpose: Get comments on an issue
- Parameters:
  - `issueId`: Issue ID
- Returns: List of comments with content and authors
- Use for: Understanding discussion history

### Write Tools (Require Approval)

**`update_issue`**
- Purpose: Update issue fields
- Parameters:
  - `id`: Issue ID (required)
  - `state`: New status (optional)
  - `title`: New title (optional)
  - `description`: New description (optional)
  - `assignee`: New assignee (optional)
  - `priority`: New priority 0-4 (optional)
  - `labels`: New labels array (optional)
- ⚠️ **REQUIRES**: Batch approval pattern
- Use for: Status updates, field changes

**`create_issue`**
- Purpose: Create new issue
- Parameters:
  - `team`: Team name or ID (required)
  - `title`: Issue title (required)
  - `description`: Issue description (optional)
  - `assignee`: User to assign (optional)
  - `priority`: Priority 0-4 (optional)
  - `labels`: Array of label names/IDs (optional)
  - `project`: Project name or ID (optional)
  - `state`: Initial state (optional)
- ⚠️ **REQUIRES**: Batch approval pattern
- Use for: Creating new tasks from notes

**`create_comment`**
- Purpose: Add comment to issue
- Parameters:
  - `issueId`: Issue ID
  - `body`: Comment text (markdown)
- ⚠️ **REQUIRES**: Batch approval pattern
- Use for: Adding context, updates

## Common Workflows

### Morning Brief Workflow

1. Get in-progress tasks:
   ```
   mcp__linear__list_issues(assignee="me", state="In Progress")
   ```

2. Process: Include in morning brief, cross-reference with Obsidian notes

### Evening Sync Workflow (with Batch Approval)

1. Get in-progress tasks:
   ```
   mcp__linear__list_issues(assignee="me", state="In Progress")
   ```

2. **Analyze** against today's notes from Obsidian

3. **Propose** updates in batch format:
   ```markdown
   ## 📝 Proposed Linear Updates
   [List of proposed changes with evidence]

   **Apply these updates?**
   ```

4. **Wait** for user confirmation

5. **Execute** approved updates:
   ```
   mcp__linear__update_issue(id="...", state="Done")
   mcp__linear__create_issue(team="...", title="...")
   ```

6. **Confirm** successful updates

### Project Brief Workflow

1. Get project details:
   ```
   mcp__linear__get_project(query="Auth Service")
   ```

2. Get all project issues:
   ```
   mcp__linear__list_issues(project="Auth Service")
   ```

3. Process: Analyze status distribution, identify blockers, assess velocity

## Best Practices

### For Read Operations
1. **Filter Early**: Use assignee, state, project filters to reduce results
2. **Pagination**: Use limit parameter (default 50 is usually good)
3. **Specific Queries**: Get exact issue when ID known
4. **Batch Context**: Get full issue details for relevant issues only

### For Write Operations
1. **Always Propose First**: Never write without showing user
2. **Evidence-Based**: Only suggest updates backed by clear evidence
3. **Conservative**: When in doubt, don't auto-suggest
4. **Transparent**: Show reasoning for each proposed change
5. **Batch Together**: Propose multiple changes at once, not one-by-one
6. **Confirm Success**: Always verify write operations succeeded

### Safety Guardrails

**DO**:
- ✅ Always use batch approval pattern for writes
- ✅ Show evidence for proposed changes
- ✅ Wait for explicit user confirmation
- ✅ Confirm successful updates

**DON'T**:
- ❌ Auto-update Linear without approval
- ❌ Guess at task status without evidence
- ❌ Create tasks for minor notes
- ❌ Update more than 10 tasks without checking
- ❌ Retry failed writes automatically

## Error Handling

- **Issue not found**: Verify issue ID, may be from different workspace
- **Permission denied**: User may not have write access to that team/project
- **Invalid state**: Check valid states with `list_issue_statuses`
- **API rate limit**: Batch operations, add delays between writes
- **Team/Project not found**: List available options, ask user to clarify

## Advanced Features

For complex requirements, see:
- `references/linear-api-guide.md` - Complete API reference and examples
- `references/batch-update-patterns.md` - Templates for multi-task updates
- `references/project-workflows.md` - Advanced project management patterns

## Integration with Other Skills

This skill commonly works with:
- **obsidian-reader** skill - match Linear tasks to Obsidian documentation
- **graphiti-memory** skill - store task context and decision history
- Commands that need task data (`/morning-brief`, `/evening-sync`, `/project-brief`)

## Token Optimization

- Use filters to reduce result size (assignee, state, project)
- Don't fetch all issues when specific ones known
- Use `limit` parameter appropriately (default 50 usually sufficient)
- Batch write operations when multiple updates needed
