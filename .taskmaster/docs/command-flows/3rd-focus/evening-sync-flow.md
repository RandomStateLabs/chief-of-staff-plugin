# /evening-sync Command Flow Trace

> **Purpose**: End-of-day workflow that reviews your work, proposes Linear task updates based on evidence found in notes and git activity, and helps you close out your day with reflection.

---

## Overview

| Property | Value |
|----------|-------|
| **Command** | `/evening-sync` |
| **Spawns Agent** | Yes - `evening-reviewer` |
| **Complexity** | High (evidence gathering, batch approval for writes) |
| **Write Operations** | Linear updates (with batch approval), Graphiti memory |

| Feature                  | Obsidian Read | Obsidian Write | Linear Read | Linear Write | Graphiti Read | Graphiti Write | GitHub Read | GitHub Write | Git Read |
| -------------------------- | --------------- | ---------------- | ------------- | -------------- | --------------- | ---------------- | ------------- | -------------- | ---------- |
| `/evening-sync`          | ✅            |                | ✅          | ✅           | ✅            | ✅             |             |              | ✅       |

---

## Flow Diagram

```
User: /evening-sync
         │
         ▼
┌─────────────────────────────────────────┐
│  COMMAND: evening-sync.md               │
│                                         │
│  1. Load skill: project-context         │
│  2. Detect current project (if in dir)  │
│  3. Spawn agent: evening-reviewer       │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  AGENT: evening-reviewer                │
│  (200K context)                         │
│                                         │
│  Activated Skills:                      │
│  - obsidian-reader                      │
│  - linear-integration (read + write)    │
│  - graphiti-memory (read + write)       │
│  - git-operations (read only)           │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 1: Gather Today's Activity       │
│                                         │
│  OBSIDIAN:                              │
│  Tool: obsidian_get_periodic_note       │
│  Params: {period: "daily"}              │
│  → Get today's journal                  │
│                                         │
│  Tool: obsidian_get_recent_changes      │
│  Params: {days: 1, limit: 30}           │
│  → Get notes modified today             │
│                                         │
│  For each relevant note:                │
│  Tool: obsidian_get_file_contents       │
│  → Read full content for evidence       │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 2: Gather Git Activity           │
│                                         │
│  Tool: Bash(git log:*)                  │
│  Params: --since="midnight" --oneline   │
│  → Today's commits                      │
│                                         │
│  Tool: Bash(git diff:*)                 │
│  Params: --stat HEAD~5                  │
│  → Files changed today                  │
│                                         │
│  Tool: Bash(git status:*)               │
│  → Uncommitted work                     │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 3: Gather Linear Current State   │
│                                         │
│  Tool: mcp__linear__list_issues         │
│  Params: {                              │
│    team: "RS42",                        │
│    assignee: "me",                      │
│    status: "in-progress"                │
│  }                                      │
│  → My in-progress tasks                 │
│                                         │
│  Tool: mcp__linear__list_issues         │
│  Params: {                              │
│    team: "RS42",                        │
│    assignee: "me",                      │
│    project: "[current-project]"         │
│  }                                      │
│  → All my tasks in current project      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 4: Cross-Reference Evidence      │
│                                         │
│  For each Linear task:                  │
│  - Search notes for task mentions       │
│  - Check git commits for related code   │
│  - Check git diff for file patterns     │
│                                         │
│  Build evidence map:                    │
│  {                                      │
│    "RS4-123": {                         │
│      notes: ["completed auth flow"],    │
│      commits: ["abc123: fix auth"],     │
│      confidence: "high"                 │
│    }                                    │
│  }                                      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 5: Generate Proposals            │
│  ═══════════════════════════════════════│
│  BATCH APPROVAL PATTERN STARTS HERE     │
│  ═══════════════════════════════════════│
│                                         │
│  ## Proposed Linear Updates             │
│                                         │
│  Based on today's evidence:             │
│                                         │
│  - [ ] RS4-123: "Auth flow" → Done      │
│        Evidence: Note "completed auth", │
│        commit "fix auth edge case"      │
│        Confidence: High                 │
│                                         │
│  - [ ] RS4-124: "API docs" → In Progress│
│        Evidence: commit "add API docs"  │
│        Confidence: Medium               │
│                                         │
│  - [ ] RS4-125: (no change suggested)   │
│        No evidence found today          │
│                                         │
│  **Please review and confirm which      │
│  updates to apply (y/n/edit):**         │
│                                         │
│  ⏸️  WAITING FOR USER CONFIRMATION      │
└────────────────┬────────────────────────┘
                 │
        User confirms
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 6: Execute Approved Updates      │
│                                         │
│  For each approved update:              │
│                                         │
│  Tool: mcp__linear__update_issue        │
│  Params: {                              │
│    id: "RS4-123",                       │
│    state: "Done"                        │
│  }                                      │
│  → Update Linear task status            │
│                                         │
│  Tool: mcp__linear__create_comment      │
│  Params: {                              │
│    issueId: "RS4-123",                  │
│    body: "Completed via evening sync.   │
│           Evidence: [notes, commits]"   │
│  }                                      │
│  → Add context to Linear task           │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 7: Store to Graphiti             │
│                                         │
│  Tool: mcp__graphiti__add_memory        │
│  Params: {                              │
│    name: "Evening Sync - Dec 4",        │
│    episode_body: "Completed: RS4-123.   │
│      In progress: RS4-124. Blocked:     │
│      RS4-125 (waiting on API spec).",   │
│    group_id: "work:current-project",    │
│    source: "text",                      │
│    source_description: "evening-sync"   │
│  }                                      │
│  → Store day's progress to memory       │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 8: Daily Reflection              │
│                                         │
│  ## Day Summary                         │
│                                         │
│  **Completed Today:**                   │
│  - RS4-123: Auth flow implementation    │
│                                         │
│  **Progressed:**                        │
│  - RS4-124: API documentation           │
│                                         │
│  **Carried Over:**                      │
│  - RS4-125: Still blocked on API spec   │
│                                         │
│  **Commits:** 5                         │
│  **Notes Created:** 2                   │
│                                         │
│  **Reflection Questions:**              │
│  - What went well today?                │
│  - What blocked you?                    │
│  - What should you focus on tomorrow?   │
└─────────────────────────────────────────┘
```

---

## Tool Calls Summary

### Read Operations (Auto-Allowed)

| Tool | Purpose | Parameters |
|------|---------|------------|
| `mcp__MCP_DOCKER__obsidian_get_periodic_note` | Today's journal | `{period: "daily"}` |
| `mcp__MCP_DOCKER__obsidian_get_recent_changes` | Today's notes | `{days: 1, limit: 30}` |
| `mcp__MCP_DOCKER__obsidian_get_file_contents` | Read note content | `{filepath: "..."}` |
| `Bash(git log:*)` | Today's commits | `--since="midnight"` |
| `Bash(git diff:*)` | Files changed | `--stat` |
| `Bash(git status:*)` | Uncommitted work | - |
| `mcp__linear__list_issues` | My tasks | `{assignee: "me"}` |

### Write Operations (Require Batch Approval)

| Tool | Purpose | Approval |
|------|---------|----------|
| `mcp__linear__update_issue` | Update task status | **BATCH APPROVAL** |
| `mcp__linear__create_comment` | Add completion context | **BATCH APPROVAL** |
| `mcp__graphiti__add_memory` | Store day summary | Auto (low risk) |

---

## Batch Approval Pattern Detail

```markdown
## Proposed Linear Updates

I found evidence for the following task updates:

### Ready to Mark Complete (High Confidence)

- [ ] **RS4-123**: "Implement auth flow" → Done
  - **Note evidence**: "Completed the auth flow, all tests passing"
  - **Git evidence**: Commits abc123, def456 in auth/
  - **Confidence**: High (explicit completion mention)

### Suggest Status Change (Medium Confidence)

- [ ] **RS4-124**: "Write API documentation" → In Progress
  - **Git evidence**: New files in docs/api/
  - **Confidence**: Medium (activity but no explicit status)

### No Changes Suggested

- **RS4-125**: "External API integration" - No evidence found
- **RS4-126**: "Bug fix" - Already marked Done

---

**Please confirm:**
1. Type `yes` to apply all suggested updates
2. Type `no` to cancel all updates
3. Type `edit` to modify individual items
4. Type the task ID to toggle individual items (e.g., `RS4-123`)

Your response:
```

---

## Skills Required

| Skill | Why Needed |
|-------|------------|
| `obsidian-reader` | Read journal and today's notes |
| `linear-integration` | Read tasks, write updates (with approval) |
| `graphiti-memory` | Store day summary |
| `git-operations` | Check today's commits and changes |
| `project-context` | Detect current project |

---

## Error Handling

| Scenario | Handling |
|----------|----------|
| No evidence found | Report "no changes suggested", skip to reflection |
| Linear update fails | Report error, continue with other updates |
| User rejects all | Skip to reflection, store "no updates made" |
| No journal today | Continue with other evidence sources |
| Git not available | Continue with Obsidian + Linear only |

---

## Safety Guarantees

1. **Never auto-updates Linear** - Always shows proposals first
2. **Shows evidence** - User can verify confidence level
3. **Allows selective approval** - Can accept/reject individual items
4. **Audit trail** - Adds comments to Linear with evidence
5. **Memory logging** - Stores sync event to Graphiti for history

---

## Example Output

```markdown
## Evening Sync - December 4, 2025

### Evidence Gathered

**Today's Notes (3):**
- "Auth implementation complete" - mentions RS4-123 as done
- "API docs progress" - WIP content
- Daily journal - general reflections

**Today's Commits (5):**
- abc123: "Complete auth flow handler"
- def456: "Add auth tests"
- ghi789: "Start API documentation"
- jkl012: "Fix typo in readme"
- mno345: "Add more API examples"

---

### Proposed Linear Updates

- [x] **RS4-123**: "Auth flow" → Done
      Evidence: Note explicitly says "complete", 2 commits in auth/
      Confidence: **High**

- [ ] **RS4-124**: "API docs" → In Progress
      Evidence: 2 commits adding docs, note mentions "WIP"
      Confidence: **Medium**

**Apply these updates? (yes/no/edit):** yes

---

### Updates Applied

✅ RS4-123 marked as Done
   - Added comment: "Completed via evening sync. Evidence: auth note, commits abc123, def456"

✅ RS4-124 status unchanged (already in-progress)

### Stored to Memory
Evening sync logged to Graphiti group "work:pe-eval"

---

### Day Summary

**Completed:** 1 task
**In Progress:** 2 tasks
**Blocked:** 1 task (RS4-125 - waiting on external API)

**Tomorrow's Focus:**
1. Follow up on external API spec
2. Continue API documentation
3. Start RS4-126 if unblocked
```
