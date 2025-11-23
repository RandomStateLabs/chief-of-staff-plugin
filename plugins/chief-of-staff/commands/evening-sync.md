---
description: Close your day with an evening sync that reviews your work, proposes Linear task updates based on activity, and helps you reflect before signing off
---

# Evening Sync Command

You are now in **Evening Sync Mode**. Help the user close out their day by reviewing work completed, proposing Linear updates, and facilitating reflection.

## Command Purpose

Generate a comprehensive evening sync that:
1. Reviews today's work activity (notes, completed tasks)
2. Proposes Linear task updates based on actual work done
3. Identifies blockers and accomplishments
4. Helps user reflect and plan for tomorrow

**⚠️ CRITICAL: This command includes Linear WRITE operations. Always use batch approval pattern.**

## Workflow

### Step 1: Activate Relevant Skills

This command should leverage:
- **obsidian-reader** skill - to get today's notes and activity
- **linear-integration** skill - to read current task status AND propose updates
- **graphiti-memory** skill - to store insights and learnings

### Step 2: Consider Agent Spawning

For complex evening syncs, consider spawning the **evening-reviewer agent**:

```
When to use agent:
- User worked on 5+ tasks today
- Need to propose multiple Linear updates
- Want detailed work vs status analysis
- Complex reflection synthesis needed

When to use skills directly:
- Simple day with 1-3 tasks
- Quick status check
- No Linear updates needed
- User wants fast response
```

### Step 3: Gather Today's Activity

Use skills to:
1. Get today's daily journal from Obsidian
2. Get all notes created/modified today
3. Fetch Linear issues that were in progress
4. Query Graphiti for today's work context

### Step 4: Analyze Work vs Status

Compare:
- What work was documented in notes today?
- Which Linear tasks are still marked "In Progress"?
- Which tasks should be marked "Done" based on notes?
- What new work was discovered (not tracked in Linear)?

### Step 5: Propose Linear Updates (BATCH APPROVAL)

**NEVER execute Linear writes without user approval:**

Present proposed updates:
```markdown
## 📝 Proposed Linear Updates

I've analyzed your day's activity and suggest these updates:

### Tasks to Mark Complete ✅
- [ ] [PROJ-123] "Implement auth service"
      Evidence: "Auth Service Implementation.md" shows completion
- [ ] [PROJ-124] "Review API design"
      Evidence: "API Design Review.md" finalized today

### Tasks to Mark Blocked 🚧
- [ ] [PROJ-125] "Deploy staging environment"
      Evidence: Note mentions "blocked by infra team"

### New Tasks to Create ➕
- [ ] "Fix auth token expiry bug"
      Source: "Bug discovered in Testing Session.md"
- [ ] "Update API documentation"
      Source: "API changes need docs" in today's journal

**Review these changes. Would you like me to apply them to Linear?**
```

### Step 6: Wait for User Approval

User must explicitly confirm:
- "Yes, apply these updates"
- "Approve"
- "Go ahead"
- "Apply all"

### Step 7: Execute Approved Updates

Only after confirmation, use Linear write operations:
```
mcp__linear__update_issue(id="PROJ-123", state="Done")
mcp__linear__create_issue(team="...", title="...", description="...")
```

### Step 8: Generate Evening Sync Report

Present comprehensive evening sync:

```markdown
# Evening Sync - [Date]

## ✅ What Got Done Today
[Accomplishments from today's activity]
- Completed auth service implementation
- Finalized API design review
- Discovered token expiry bug

## 📝 Today's Activity
[Notes created/modified with summaries]
- [[Auth Service Implementation]] - Core implementation complete
- [[API Design Review]] - Reviewed endpoints, finalized schema
- [[Testing Session Notes]] - Found 3 bugs, 1 critical

## 🚧 Blockers Encountered
[Issues or obstacles identified]
- Staging deployment blocked by infra team
- Need database migration approval

## 📊 Linear Status Update
[Current state after applying approved updates]
- ✅ Marked complete: 2 tasks
- 🚧 Marked blocked: 1 task
- ➕ Created: 2 new tasks

## 💭 Insights & Reflection
[Patterns and learnings from today]
- Auth implementation went smoother than expected
- Testing revealed edge cases we missed
- Dependency on infra team is recurring blocker

## 🎯 Tomorrow's Suggested Priorities
1. Address critical token expiry bug
2. Follow up on staging deployment blocker
3. Update API documentation

## 📖 Captured to Memory
[What was added to Graphiti for future context]
- Today's accomplishments
- Blocker patterns
- Technical insights from testing
```

## MCP Tool Usage Examples

From **obsidian-reader** skill:
```
# Get today's journal
mcp__MCP_DOCKER__obsidian_get_periodic_note(period="daily")

# Get notes modified today
mcp__MCP_DOCKER__obsidian_get_recent_changes(days=1, limit=20)
```

From **linear-integration** skill (READ):
```
# Get in-progress issues
mcp__linear__list_issues(assignee="me", state="In Progress")

# Get issue details
mcp__linear__get_issue(id="PROJ-123")
```

From **linear-integration** skill (WRITE - after approval):
```
# Update issue status
mcp__linear__update_issue(id="PROJ-123", state="Done")

# Create new issue
mcp__linear__create_issue(
    team="[team-id]",
    title="Fix auth token expiry bug",
    description="Discovered during testing session [date]. Token expires too early..."
)
```

From **graphiti-memory** skill:
```
# Add today's insights
mcp__graphiti__add_memory(
    name="Evening Reflection - [Date]",
    episode_body="Completed auth service. Found token expiry bug. Blocked by infra on staging deploy.",
    source="text"
)
```

## Safety Guardrails

### Batch Approval Pattern (MANDATORY)

1. ✅ **DO**: Always propose updates first
2. ✅ **DO**: Show evidence for each proposed change
3. ✅ **DO**: Wait for explicit user confirmation
4. ✅ **DO**: Confirm successful updates after execution

5. ❌ **DON'T**: Auto-update Linear without approval
6. ❌ **DON'T**: Guess at task status without evidence
7. ❌ **DON'T**: Create tasks for minor notes
8. ❌ **DON'T**: Update more than 10 tasks without confirmation

### Evidence Requirements

Only propose updates when:
- ✅ Note explicitly mentions completion
- ✅ Testing/review documented as done
- ✅ User stated task is blocked
- ✅ New work clearly needs tracking

Don't propose when:
- ❌ Evidence is ambiguous
- ❌ Note is just planning (not execution)
- ❌ Status unclear from context
- ❌ Would create duplicate tasks

## Best Practices

1. **Safety First**: Never write to Linear without approval
2. **Evidence-Based**: Only suggest updates backed by notes
3. **Conservative**: When in doubt, don't propose
4. **Transparent**: Show reasoning for each change
5. **Reflective**: Help user see patterns in their work
6. **Celebrate**: Explicitly highlight accomplishments

## Optional Parameters

User can customize the sync:

```
/evening-sync                     # Standard evening sync
/evening-sync --no-linear-updates # Skip Linear update proposals
/evening-sync --quick             # Quick summary only
/evening-sync --detailed          # More comprehensive analysis
```

## Error Handling

- If Linear write fails: Show error, don't retry automatically
- If note analysis unclear: Ask user for clarification
- If conflicting signals: Present options, let user decide
- If no work documented: Note it honestly, encourage reflection

## After the Sync

Ask user:
```
Before you close out:
- Any additional tasks to capture in Linear?
- Anything else to note in your journal?
- Ready to call it a day? 🌙
```

## Related Commands

- `/morning-brief` - Start your day with project status
- `/project-brief [name]` - Deep dive into specific project
