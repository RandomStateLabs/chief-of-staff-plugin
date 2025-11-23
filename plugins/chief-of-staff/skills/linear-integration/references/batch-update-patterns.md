# Batch Update Patterns for Linear

This reference provides templates and examples for safely proposing and executing batch updates to Linear issues.

## The Batch Approval Pattern

**Golden Rule**: NEVER write to Linear without explicit user approval.

### Standard Flow

```
1. ANALYZE → Compare actual work (from notes) vs Linear status
2. PROPOSE → Present changes in clear checklist format
3. WAIT → Explicitly ask for confirmation
4. CONFIRM → User says "yes" / "approve" / "go ahead"
5. EXECUTE → Call Linear write operations
6. VERIFY → Confirm successful updates
```

## Template: Standard Evening Sync

```markdown
## 📝 Proposed Linear Updates

I've analyzed your day's activity in Obsidian and suggest these updates:

### Tasks to Mark Complete ✅
- [ ] [PROJ-123] "Implement authentication service"
      Evidence: "Auth Service Implementation.md" shows completion, testing complete
      Current: In Progress → Proposed: Done

- [ ] [PROJ-124] "Review API design document"
      Evidence: "API Design Review.md" finalized with sign-off
      Current: In Progress → Proposed: Done

### Tasks to Mark Blocked 🚧
- [ ] [PROJ-125] "Deploy to staging environment"
      Evidence: Note mentions "blocked by infrastructure team"
      Current: In Progress → Proposed: Blocked

### New Tasks to Create ➕
- [ ] "Fix auth token expiry bug"
      Priority: High
      Source: "Testing Session.md" - Critical bug discovered
      Labels: bug, auth-service
      Description: Token expires prematurely causing auth failures in production

- [ ] "Update API documentation for new endpoints"
      Priority: Normal
      Source: Today's journal - API changes need documentation
      Labels: documentation, api

**Review these 5 proposed changes. Would you like me to apply them to Linear?**
```

## Evidence Requirements

Only propose updates when evidence is clear:

### ✅ GOOD Evidence for "Mark Complete"
- Note titled "X Implementation Complete" with completion checklist
- Testing/review note with "all tests passing" or "approved"
- User explicitly wrote "finished" or "done" in journal
- Documentation shows feature shipped/deployed

### ❌ WEAK Evidence (Don't Propose)
- Note just mentions the task
- Planning notes (not execution)
- Work in progress but not done
- Ambiguous status

### ✅ GOOD Evidence for "Mark Blocked"
- Note explicitly says "blocked by X"
- User wrote "waiting on Y team"
- Clear external dependency mentioned
- User expressed frustration about obstacle

### ❌ WEAK Evidence (Don't Propose)
- Just seems slow
- No explicit blocker mentioned
- Minor delay (not true blocker)

### ✅ GOOD Evidence for "Create New Task"
- Clear bug discovered and documented
- User wrote "TODO: X" or "Need to: Y"
- Gap identified between systems
- User explicitly requested task creation

### ❌ WEAK Evidence (Don't Propose)
- Vague idea mentioned
- Already covered by existing task
- Minor note (not task-worthy)
- Duplicate of existing issue

## Execution Templates

### After User Approves - Execute Updates

```python
# Mark tasks complete
for issue_id in completed_issues:
    mcp__linear__update_issue(
        id=issue_id,
        state="Done"
    )

# Mark tasks blocked
for issue_id in blocked_issues:
    mcp__linear__update_issue(
        id=issue_id,
        state="Blocked"
    )

# Create new tasks
for new_task in new_tasks:
    mcp__linear__create_issue(
        team=new_task["team"],
        title=new_task["title"],
        description=new_task["description"],
        labels=new_task["labels"],
        priority=new_task["priority"]
    )
```

### Confirmation Message After Success

```markdown
✅ **Linear Updates Applied Successfully**

**Completed**:
- PROJ-123: Implement authentication service → Done
- PROJ-124: Review API design document → Done

**Blocked**:
- PROJ-125: Deploy to staging environment → Blocked

**Created**:
- PROJ-456: Fix auth token expiry bug (High priority)
- PROJ-457: Update API documentation for new endpoints

All updates have been applied to Linear. Your workspace is now in sync with today's work!
```

## Edge Cases

### User Wants Partial Approval

If user says "only do the first two":

```markdown
Got it! I'll apply only the first two updates:

✅ Applying:
- PROJ-123 → Done
- PROJ-124 → Done

⏭️ Skipping:
- PROJ-125 (not marking as blocked)
- New task creation

[Execute only approved items]
```

### User Wants Modifications

If user says "mark PROJ-123 as done but change the description":

```markdown
Understood! I'll:
1. Mark PROJ-123 as Done
2. Update its description to: "[user's new description]"

Proceeding with modified update...

[Execute with modifications]
```

### Write Operation Fails

If Linear API returns error:

```markdown
❌ **Update Failed**

I tried to update PROJ-123 but got an error:
"Issue not found or you don't have permission"

Possible causes:
- Issue ID is incorrect
- You don't have write access to this team
- Issue was deleted or moved

Would you like me to:
1. Try the other updates (skip this one)
2. Retry with correct issue ID
3. Cancel all updates
```

## Anti-Patterns (DON'T DO THIS)

### ❌ Auto-Updating Without Approval

```markdown
I noticed you completed 3 tasks. I've updated them in Linear for you!
✅ PROJ-123 → Done
```

**Why wrong**: No user approval, automatic write operation

### ❌ Vague Proposals

```markdown
Should I update some tasks in Linear?
```

**Why wrong**: Not showing what will change, no evidence

### ❌ One-by-One Approval

```markdown
Task PROJ-123 looks done. Update it? [waits]
Task PROJ-124 looks done. Update it? [waits]
```

**Why wrong**: Inefficient, should batch together

### ❌ Proposing Without Evidence

```markdown
These tasks haven't been updated in a while. Mark them as blocked?
```

**Why wrong**: No evidence from notes, just guessing

## Best Practices

1. **Always Show Evidence**: Reference specific notes, quotes, or observations
2. **Be Conservative**: When in doubt, don't propose
3. **Batch Together**: Group all proposed changes into one approval request
4. **Clear Format**: Use checkboxes, clear current → proposed states
5. **Explicit Ask**: End with clear question asking for approval
6. **Verify Success**: Always confirm what was applied
7. **Handle Errors Gracefully**: Don't retry automatically, ask user
8. **Respect Decisions**: If user says no, don't push back

## Safety Checklist

Before executing any Linear write operation, verify:

- [ ] Proposed changes were shown to user
- [ ] Evidence was provided for each change
- [ ] User explicitly approved (said yes/approve/go ahead)
- [ ] Changes are backed by note content
- [ ] Not updating more than 10-15 items at once (avoid overwhelming)
- [ ] Have verified issue IDs are correct
- [ ] Ready to handle potential API errors

## Integration with Evening Sync

The evening-sync command should:

1. Get today's notes from Obsidian
2. Get in-progress Linear issues
3. Compare and identify discrepancies
4. Propose updates using this template
5. Wait for approval
6. Execute approved changes
7. Store summary in Graphiti for future context

Remember: The evening sync is a **killer feature** because it automates the tedious task of updating Linear based on actual work done. But it only works if users trust it - which requires the batch approval pattern.
