# /sync-tasks Command Flow Trace

> **Purpose**: Bidirectional sync between Linear (stakeholder milestones) and TaskMaster (developer subtasks). Detects scope changes in Linear, cascades updates to TaskMaster with approval, and reports parent task completions back to Linear.

---

## Overview

| Property | Value |
|----------|-------|
| **Command** | `/sync-tasks` |
| **Spawns Agent** | Yes - `task-syncer` |
| **Complexity** | High (bidirectional sync, cascade detection) |
| **Write Operations** | Linear updates (batch), TaskMaster updates (batch) |

| Feature                  | Obsidian Read | Obsidian Write | Linear Read | Linear Write | Graphiti Read | Graphiti Write | GitHub Read | GitHub Write | Git Read |
| -------------------------- | --------------- | ---------------- | ------------- | -------------- | --------------- | ---------------- | ------------- | -------------- | ---------- |
| `/sync-tasks`            |               |                | ✅          | ✅           |               |                |             |              |          |

---

## Two-View Task Philosophy

```
┌─────────────────────────────────────────────────────────────┐
│                    LINEAR (Stakeholder View)                 │
│                                                             │
│  High-level milestones, phases, deliverables               │
│  - "Implement Authentication" (RS4-100)                     │
│  - "Launch Beta" (RS4-101)                                  │
│  - Visible to stakeholders, PM-friendly                     │
└────────────────────────────┬────────────────────────────────┘
                             │
                    /sync-tasks
                             │
┌────────────────────────────▼────────────────────────────────┐
│                 TASKMASTER (Developer View)                  │
│                                                             │
│  Implementation subtasks, technical details                 │
│  - Task 1: "Implement Authentication"                       │
│    - 1.1: Set up JWT library                               │
│    - 1.2: Create auth middleware                           │
│    - 1.3: Build login endpoint                             │
│    - 1.4: Add token refresh                                │
│  - Not visible to stakeholders, dev-focused                │
└─────────────────────────────────────────────────────────────┘
```

---

## Sync Rules

### What Syncs (Linear → TaskMaster)

| Linear Change | TaskMaster Action |
|---------------|-------------------|
| Priority changed | Update parent task priority |
| Scope expanded (description) | Flag for subtask review |
| Status → Cancelled | Cancel parent + all subtasks |
| Status → Blocked | Mark parent as blocked |

### What Syncs (TaskMaster → Linear)

| TaskMaster Change | Linear Action |
|-------------------|---------------|
| All subtasks done | Propose marking Linear issue Done |
| Parent marked done | Update Linear status |
| Blocker documented | Add comment to Linear issue |

### What Stays Local

| System | Stays Local |
|--------|-------------|
| TaskMaster | Subtask details, implementation notes, test strategies |
| Linear | Comments, attachments, external links |

---

## Flow Diagram

```
User: /sync-tasks
         │
         ▼
┌─────────────────────────────────────────┐
│  COMMAND: sync-tasks.md                 │
│                                         │
│  1. Load skill: project-context         │
│  2. Detect current project              │
│  3. Spawn agent: task-syncer            │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  AGENT: task-syncer                     │
│  (200K context)                         │
│                                         │
│  Activated Skills:                      │
│  - linear-integration (read + write)    │
│  - taskmaster-mcp                       │
│  - project-context                      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 1: Load Linear State             │
│                                         │
│  Tool: mcp__linear__get_project         │
│  Params: {query: "[project-name]"}      │
│  → Get project ID                       │
│                                         │
│  Tool: mcp__linear__list_issues         │
│  Params: {                              │
│    project: "[project-id]",             │
│    includeArchived: false               │
│  }                                      │
│  → All Linear issues for project        │
│                                         │
│  For each issue:                        │
│  Tool: mcp__linear__get_issue           │
│  → Get full details including history   │
│                                         │
│  Build Linear state map:                │
│  {                                      │
│    "RS4-100": {                         │
│      title: "Implement Auth",           │
│      status: "in-progress",             │
│      priority: "high",                  │
│      description: "...",                │
│      last_updated: "2025-12-03"         │
│    }                                    │
│  }                                      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 2: Load TaskMaster State         │
│                                         │
│  Tool: mcp__taskmaster-ai__get_tasks    │
│  Params: {                              │
│    projectRoot: "[path]",               │
│    withSubtasks: true                   │
│  }                                      │
│  → All TaskMaster tasks with subtasks   │
│                                         │
│  Build TaskMaster state map:            │
│  {                                      │
│    "1": {                               │
│      title: "Implement Auth",           │
│      linear_id: "RS4-100",              │
│      status: "in-progress",             │
│      subtasks: [                        │
│        {id: "1.1", status: "done"},     │
│        {id: "1.2", status: "done"},     │
│        {id: "1.3", status: "in-progress"}│
│        {id: "1.4", status: "pending"}   │
│      ]                                  │
│    }                                    │
│  }                                      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 3: Detect Linear → TM Changes    │
│                                         │
│  For each Linear issue, compare:        │
│                                         │
│  PRIORITY CHANGES:                      │
│  - Linear RS4-100 priority: high        │
│  - TaskMaster 1 priority: medium        │
│  → Flag: Update TM priority to high     │
│                                         │
│  SCOPE CHANGES (description expanded):  │
│  - Linear desc has new requirements     │
│  - TaskMaster subtasks don't cover them │
│  → Flag: Review subtasks needed         │
│                                         │
│  STATUS CHANGES:                        │
│  - Linear RS4-101 → Cancelled           │
│  - TaskMaster 2 still pending           │
│  → Flag: Cancel TM task + subtasks      │
│                                         │
│  Build Linear → TM sync list:           │
│  [                                      │
│    {type: "priority", linear: "RS4-100",│
│     tm_task: "1", from: "medium",       │
│     to: "high"},                        │
│    {type: "scope", linear: "RS4-102",   │
│     tm_task: "3", new_requirements: []}  │
│  ]                                      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 4: Detect TM → Linear Changes    │
│                                         │
│  For each TaskMaster task:              │
│                                         │
│  COMPLETION DETECTION:                  │
│  - Task 1: All subtasks done?           │
│    - 1.1: done ✓                        │
│    - 1.2: done ✓                        │
│    - 1.3: done ✓                        │
│    - 1.4: done ✓                        │
│  → Flag: Propose RS4-100 → Done         │
│                                         │
│  BLOCKER DETECTION:                     │
│  - Task 2 subtask has blocker note      │
│  → Flag: Add comment to RS4-101         │
│                                         │
│  Build TM → Linear sync list:           │
│  [                                      │
│    {type: "completion", tm_task: "1",   │
│     linear: "RS4-100",                  │
│     evidence: "4/4 subtasks done"},     │
│    {type: "blocker", tm_task: "2.3",    │
│     linear: "RS4-101",                  │
│     note: "Waiting on API spec"}        │
│  ]                                      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 5: Generate Sync Proposals       │
│  ═══════════════════════════════════════│
│  BATCH APPROVAL PATTERN                 │
│  ═══════════════════════════════════════│
│                                         │
│  ## Task Sync Proposals                 │
│                                         │
│  ### Linear → TaskMaster                │
│                                         │
│  - [ ] **RS4-100** priority: medium→high│
│        TaskMaster task 1 will update    │
│                                         │
│  - [ ] **RS4-102** scope expanded       │
│        New requirements detected:       │
│        - "Add OAuth support"            │
│        - "Support MFA"                  │
│        → Review task 3 subtasks         │
│                                         │
│  - [ ] **RS4-103** cancelled in Linear  │
│        TaskMaster task 4 → cancelled    │
│        (3 subtasks will also cancel)    │
│                                         │
│  ### TaskMaster → Linear                │
│                                         │
│  - [ ] **Task 1** complete (4/4 done)   │
│        → Mark RS4-100 as Done           │
│                                         │
│  - [ ] **Task 2.3** has blocker         │
│        → Add comment to RS4-101         │
│                                         │
│  **Apply sync? (yes/no/edit):**         │
│                                         │
│  ⏸️  WAITING FOR USER CONFIRMATION      │
└────────────────┬────────────────────────┘
                 │
        User confirms
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 6: Execute Linear → TM Sync      │
│                                         │
│  PRIORITY UPDATE:                       │
│  Tool: mcp__taskmaster-ai__update_task  │
│  Params: {                              │
│    id: "1",                             │
│    projectRoot: "[path]",               │
│    prompt: "Update priority to high     │
│      (synced from Linear RS4-100)"      │
│  }                                      │
│                                         │
│  CANCELLATION CASCADE:                  │
│  Tool: mcp__taskmaster-ai__set_task_status
│  Params: {                              │
│    id: "4",                             │
│    status: "cancelled",                 │
│    projectRoot: "[path]"                │
│  }                                      │
│  (Repeat for each subtask: 4.1, 4.2...) │
│                                         │
│  SCOPE REVIEW:                          │
│  Display to user:                       │
│  "RS4-102 has new scope. Run            │
│   'task-master expand --id=3' to        │
│   generate new subtasks?"               │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 7: Execute TM → Linear Sync      │
│                                         │
│  COMPLETION:                            │
│  Tool: mcp__linear__update_issue        │
│  Params: {                              │
│    id: "RS4-100",                       │
│    state: "Done"                        │
│  }                                      │
│                                         │
│  Tool: mcp__linear__create_comment      │
│  Params: {                              │
│    issueId: "RS4-100",                  │
│    body: "Completed via /sync-tasks.    │
│           All 4 subtasks finished:      │
│           - 1.1: JWT setup ✓            │
│           - 1.2: Auth middleware ✓      │
│           - 1.3: Login endpoint ✓       │
│           - 1.4: Token refresh ✓"       │
│  }                                      │
│                                         │
│  BLOCKER COMMENT:                       │
│  Tool: mcp__linear__create_comment      │
│  Params: {                              │
│    issueId: "RS4-101",                  │
│    body: "⚠️ Blocker detected in        │
│           subtask 2.3: Waiting on       │
│           external API specification"   │
│  }                                      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  OUTPUT: Sync Summary                   │
│                                         │
│  ## Task Sync Complete                  │
│                                         │
│  ### Linear → TaskMaster                │
│                                         │
│  ✅ Task 1 priority → high              │
│  ✅ Task 4 cancelled (+ 3 subtasks)     │
│  ⚠️ Task 3 needs subtask review         │
│     (scope expanded in RS4-102)         │
│                                         │
│  ### TaskMaster → Linear                │
│                                         │
│  ✅ RS4-100 → Done                      │
│     (comment added with subtask summary)│
│  ✅ RS4-101 → Blocker comment added     │
│                                         │
│  ### Action Items                       │
│                                         │
│  1. Review RS4-102 scope expansion      │
│     Run: task-master expand --id=3      │
│                                         │
│  2. Resolve blocker on RS4-101          │
│     External API spec needed            │
│                                         │
│  ### Stats                              │
│                                         │
│  - Tasks synced: 4                      │
│  - Linear updates: 2                    │
│  - TaskMaster updates: 5                │
│  - Comments added: 2                    │
└─────────────────────────────────────────┘
```

---

## Tool Calls Summary

### Read Operations (Auto-Allowed)

| Tool | Purpose | Parameters |
|------|---------|------------|
| `mcp__linear__get_project` | Project ID | `{query: "..."}` |
| `mcp__linear__list_issues` | All project issues | `{project: "..."}` |
| `mcp__linear__get_issue` | Issue details | `{id: "..."}` |
| `mcp__taskmaster-ai__get_tasks` | All TM tasks | `{withSubtasks: true}` |
| `mcp__taskmaster-ai__get_task` | Task details | `{id: "..."}` |

### Write Operations (Require Batch Approval)

| Tool | Purpose | Approval |
|------|---------|----------|
| `mcp__linear__update_issue` | Update status | **BATCH APPROVAL** |
| `mcp__linear__create_comment` | Add context | **BATCH APPROVAL** |
| `mcp__taskmaster-ai__update_task` | Update TM task | **BATCH APPROVAL** |
| `mcp__taskmaster-ai__set_task_status` | Change status | **BATCH APPROVAL** |

---

## Skills Required

| Skill | Why Needed |
|-------|------------|
| `linear-integration` | Read issues, write updates |
| `taskmaster-mcp` | Read/write TaskMaster state |
| `project-context` | Detect current project |

---

## Linking Convention

Tasks are linked via:

1. **TaskMaster task description** contains Linear ID:
   ```
   Linear: RS4-100
   ```

2. **Linear issue description** contains TaskMaster reference:
   ```
   TaskMaster: Task 1
   ```

3. **Metadata field** (if supported):
   ```json
   {"linear_id": "RS4-100"}
   ```

---

## Cascade Rules

### When Linear Issue Cancelled

```
Linear: RS4-103 → Cancelled
        │
        ▼
TaskMaster: Task 4 → cancelled
            ├── 4.1 → cancelled
            ├── 4.2 → cancelled
            └── 4.3 → cancelled
```

### When All Subtasks Complete

```
TaskMaster: Task 1
            ├── 1.1 ✓ done
            ├── 1.2 ✓ done
            ├── 1.3 ✓ done
            └── 1.4 ✓ done
        │
        ▼ (propose)
Linear: RS4-100 → Done
        + Comment with summary
```

---

## Error Handling

| Scenario | Handling |
|----------|----------|
| Linear issue not linked | Skip, report unlinked tasks |
| TaskMaster task not found | Report, offer to create link |
| Conflicting status | Show both, ask user to resolve |
| API failure | Report, continue with other syncs |
| Cascade would orphan work | Warn user, require explicit confirm |

---

## Example Output

```markdown
## Task Sync Complete: PE Evaluation

### Sync Summary

| Direction | Changes | Applied |
|-----------|---------|---------|
| Linear → TaskMaster | 3 | 2 |
| TaskMaster → Linear | 2 | 2 |

---

### Linear → TaskMaster

✅ **RS4-100** priority synced
   - TaskMaster Task 1: medium → high

✅ **RS4-103** cancellation cascaded
   - TaskMaster Task 4: cancelled
   - Subtasks 4.1, 4.2, 4.3: cancelled

⚠️ **RS4-102** scope review needed
   - Linear description expanded with:
     - "Add OAuth support"
     - "Support MFA"
   - **Action**: Run `task-master expand --id=3 --force`

---

### TaskMaster → Linear

✅ **RS4-100** marked Done
   - All 4 subtasks completed
   - Comment added with summary:
     ```
     Completed via /sync-tasks.
     Subtasks: JWT setup, auth middleware,
     login endpoint, token refresh
     ```

✅ **RS4-101** blocker documented
   - Comment added:
     ```
     ⚠️ Blocker in subtask 2.3:
     Waiting on external API spec
     ```

---

### Unlinked Items (Not Synced)

**Linear issues without TaskMaster link**:
- RS4-105: "Update readme" (no TM task)

**TaskMaster tasks without Linear link**:
- Task 5: "Refactor utils" (no Linear ID)

*Run `/sync-tasks --link` to create links*

---

### Next Steps

1. Review scope expansion for RS4-102:
   ```bash
   task-master show 3
   task-master expand --id=3 --force
   ```

2. Resolve blocker on RS4-101:
   - Contact vendor for API spec
   - Update task 2.3 when resolved

3. Link unlinked items (optional):
   ```bash
   /sync-tasks --link
   ```
```
