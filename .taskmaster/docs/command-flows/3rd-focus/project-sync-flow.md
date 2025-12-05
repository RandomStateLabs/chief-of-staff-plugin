# /project-sync Command Flow Trace

> **Purpose**: Syncs state for the current project across all systems. Git activity updates Linear tasks, note evidence proposes task completions, decisions store to Graphiti memory.

---

## Overview

| Property | Value |
|----------|-------|
| **Command** | `/project-sync` |
| **Spawns Agent** | Yes - `project-analyst` |
| **Complexity** | High (multi-system sync, batch approval for writes) |
| **Write Operations** | Linear updates (batch approval), Graphiti memory |

| Feature                  | Obsidian Read | Obsidian Write | Linear Read | Linear Write | Graphiti Read | Graphiti Write | GitHub Read | GitHub Write | Git Read |
| -------------------------- | --------------- | ---------------- | ------------- | -------------- | --------------- | ---------------- | ------------- | -------------- | ---------- |
| `/project-sync`          | ✅            |                | ✅          | ✅           | ✅            | ✅             |             |              | ✅       |

---

## Critical Safety Note

**This command uses SPECIFIC git read commands only:**

```yaml
ALLOWED:
  - Bash(git status:*)
  - Bash(git log:*)
  - Bash(git diff:*)
  - Bash(git branch -a:*)
  - Bash(git branch -l:*)
  - Bash(git remote -v:*)
  - Bash(git rev-parse:*)
  - Bash(git show:*)

NEVER ALLOWED (no wildcard Bash(git:*)):
  - git push
  - git commit
  - git checkout
  - git reset
  - git merge
  - git rebase
```

---

## Flow Diagram

```
User: /project-sync
         │
         ▼
┌─────────────────────────────────────────┐
│  COMMAND: project-sync.md               │
│                                         │
│  1. Load skill: project-context         │
│  2. Detect current project from:        │
│     - pwd → directory name              │
│     - CLAUDE.md → project reference     │
│     - Linear project match              │
│  3. Spawn agent: project-analyst        │
│     with detected project context       │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  AGENT: project-analyst                 │
│  (200K context)                         │
│                                         │
│  Prompt: "Sync project: [detected-name] │
│  from directory: [pwd]"                 │
│                                         │
│  Activated Skills:                      │
│  - obsidian-reader                      │
│  - linear-integration (read + write)    │
│  - graphiti-memory (read + write)       │
│  - git-operations (read only - EXPLICIT)│
│  - project-context                      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 1: Gather Git Evidence           │
│  (READ ONLY - Specific Commands)        │
│                                         │
│  Tool: Bash(git status:*)               │
│  → Current branch, uncommitted changes  │
│                                         │
│  Tool: Bash(git log:*)                  │
│  Params: --oneline -20 --since="3 days" │
│  → Recent commits with messages         │
│                                         │
│  Tool: Bash(git diff:*)                 │
│  Params: --stat HEAD~10                 │
│  → Files changed recently               │
│                                         │
│  Tool: Bash(git branch -a:*)            │
│  → All branches                         │
│                                         │
│  Tool: Bash(git remote -v:*)            │
│  → Remote configuration                 │
│                                         │
│  Extract from commit messages:          │
│  - Task IDs (RS4-XXX pattern)           │
│  - Keywords: "fix", "complete", "wip"   │
│  - File patterns → task categories      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 2: Gather Obsidian Evidence      │
│                                         │
│  Tool: obsidian_simple_search           │
│  Params: {query: "[project-name]"}      │
│  → Find project-related notes           │
│                                         │
│  Tool: obsidian_get_recent_changes      │
│  Params: {days: 3, limit: 20}           │
│  → Recent notes that might be relevant  │
│                                         │
│  For relevant notes:                    │
│  Tool: obsidian_get_file_contents       │
│  → Read full content for evidence       │
│                                         │
│  Extract:                               │
│  - Task completions mentioned           │
│  - Decisions made                       │
│  - Blockers documented                  │
│  - Progress updates                     │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 3: Gather Linear Current State   │
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
│  → All project issues                   │
│                                         │
│  For each issue:                        │
│  - Current status                       │
│  - Assignee                             │
│  - Last updated                         │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 4: Cross-Reference & Detect      │
│  Sync Opportunities                     │
│                                         │
│  For each Linear task:                  │
│                                         │
│  1. Search git commits for task ID      │
│     - "fix RS4-123" → task worked on    │
│     - "complete RS4-123" → task done    │
│                                         │
│  2. Search notes for task mentions      │
│     - "finished the auth flow" → done   │
│     - "blocked on X" → blocked status   │
│                                         │
│  3. Match file patterns to tasks        │
│     - auth/* changes → auth task        │
│     - docs/* changes → docs task        │
│                                         │
│  Build evidence map:                    │
│  {                                      │
│    "RS4-123": {                         │
│      current_status: "in-progress",     │
│      suggested_status: "done",          │
│      git_evidence: ["commit abc123"],   │
│      note_evidence: ["note: completed"],│
│      confidence: "high"                 │
│    }                                    │
│  }                                      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 5: Detect Decisions to Store     │
│                                         │
│  Scan notes for decision patterns:      │
│  - "Decided to..."                      │
│  - "We're going with..."                │
│  - "Chose X over Y because..."          │
│                                         │
│  Extract decisions not yet in Graphiti: │
│  Tool: mcp__graphiti__search_memory_facts
│  Params: {                              │
│    query: "[decision keywords]",        │
│    group_ids: ["work:project-slug"]     │
│  }                                      │
│  → Check if decision already stored     │
│                                         │
│  Build new decisions list:              │
│  [                                      │
│    {                                    │
│      decision: "Use JWT for auth",      │
│      rationale: "Stateless scaling",    │
│      source: "note: auth-design.md"     │
│    }                                    │
│  ]                                      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 6: Generate Proposals            │
│  ═══════════════════════════════════════│
│  BATCH APPROVAL PATTERN                 │
│  ═══════════════════════════════════════│
│                                         │
│  ## Project Sync Proposals              │
│                                         │
│  ### Linear Task Updates                │
│                                         │
│  - [ ] RS4-123: In Progress → Done      │
│        Git: "complete auth flow"        │
│        Note: "auth implementation done" │
│        Confidence: High                 │
│                                         │
│  - [ ] RS4-124: Pending → In Progress   │
│        Git: 3 commits in api/           │
│        Confidence: Medium               │
│                                         │
│  ### Decisions to Store in Memory       │
│                                         │
│  - [ ] "JWT for authentication"         │
│        Source: auth-design.md           │
│        Group: work:pe-eval              │
│                                         │
│  - [ ] "PostgreSQL over MongoDB"        │
│        Source: db-decision.md           │
│        Group: work:pe-eval              │
│                                         │
│  **Review and confirm (yes/no/edit):**  │
│                                         │
│  ⏸️  WAITING FOR USER CONFIRMATION      │
└────────────────┬────────────────────────┘
                 │
        User confirms
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 7: Execute Approved Updates      │
│                                         │
│  LINEAR UPDATES:                        │
│                                         │
│  Tool: mcp__linear__update_issue        │
│  Params: {id: "RS4-123", state: "Done"} │
│                                         │
│  Tool: mcp__linear__create_comment      │
│  Params: {                              │
│    issueId: "RS4-123",                  │
│    body: "Synced from project-sync.     │
│           Evidence: [git commits, notes]"
│  }                                      │
│                                         │
│  GRAPHITI MEMORY:                       │
│                                         │
│  Tool: mcp__graphiti__add_memory        │
│  Params: {                              │
│    name: "Decision: JWT Auth",          │
│    episode_body: "Decided to use JWT    │
│      for authentication. Rationale:     │
│      stateless scaling, team expertise.",
│    group_id: "work:pe-eval",            │
│    source: "text",                      │
│    source_description: "project-sync"   │
│  }                                      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 8: Store Sync Event              │
│                                         │
│  Tool: mcp__graphiti__add_memory        │
│  Params: {                              │
│    name: "Project Sync - Dec 4",        │
│    episode_body: "Synced PE Eval:       │
│      - Updated RS4-123 to Done          │
│      - Started RS4-124                  │
│      - Stored 2 decisions",             │
│    group_id: "work:pe-eval",            │
│    source: "text",                      │
│    source_description: "project-sync"   │
│  }                                      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  OUTPUT: Sync Summary                   │
│                                         │
│  ## Project Sync Complete               │
│                                         │
│  ### Linear Updates Applied             │
│  ✅ RS4-123: Done                       │
│  ✅ RS4-124: In Progress                │
│                                         │
│  ### Decisions Stored to Memory         │
│  ✅ JWT for authentication              │
│  ✅ PostgreSQL over MongoDB             │
│                                         │
│  ### Sync Logged                        │
│  Event stored to work:pe-eval           │
│                                         │
│  ### Skipped (No Evidence)              │
│  - RS4-125: No activity detected        │
│  - RS4-126: Already up to date          │
└─────────────────────────────────────────┘
```

---

## Tool Calls Summary

### Read Operations (Auto-Allowed)

| Tool | Purpose | Parameters |
|------|---------|------------|
| `Bash(git status:*)` | Current state | - |
| `Bash(git log:*)` | Recent commits | `--oneline -20 --since="3 days"` |
| `Bash(git diff:*)` | Changed files | `--stat HEAD~10` |
| `Bash(git branch -a:*)` | All branches | - |
| `Bash(git remote -v:*)` | Remote config | - |
| `mcp__MCP_DOCKER__obsidian_simple_search` | Project notes | `{query: "..."}` |
| `mcp__MCP_DOCKER__obsidian_get_recent_changes` | Recent notes | `{days: 3}` |
| `mcp__MCP_DOCKER__obsidian_get_file_contents` | Read notes | `{filepath: "..."}` |
| `mcp__linear__get_project` | Project ID | `{query: "..."}` |
| `mcp__linear__list_issues` | All issues | `{project: "..."}` |
| `mcp__graphiti__search_memory_facts` | Existing facts | `{group_ids: [...]}` |

### Write Operations (Require Batch Approval)

| Tool | Purpose | Approval |
|------|---------|----------|
| `mcp__linear__update_issue` | Update task status | **BATCH APPROVAL** |
| `mcp__linear__create_comment` | Add sync context | **BATCH APPROVAL** |
| `mcp__graphiti__add_memory` | Store decisions | **BATCH APPROVAL** |
| `mcp__graphiti__add_memory` | Log sync event | Auto (low risk) |

---

## Skills Required

| Skill | Why Needed |
|-------|------------|
| `obsidian-reader` | Read project notes for evidence |
| `linear-integration` | Read issues, write updates (with approval) |
| `graphiti-memory` | Check existing facts, store new decisions |
| `git-operations` | Read git history (specific commands only) |
| `project-context` | Detect current project from directory |

---

## Project Detection Logic

```python
# Priority order for project detection:

1. CLAUDE.md in current directory
   - Look for "Project:" or "Linear Project:" reference

2. Directory name matching
   - pwd → basename → fuzzy match to Linear projects

3. Git remote URL
   - Parse repo name from origin URL

4. Explicit argument
   - User can override: /project-sync --project="PE Eval"
```

---

## Error Handling

| Scenario | Handling |
|----------|----------|
| Project not detected | Ask user to specify |
| No git repo | Continue with Obsidian + Linear only |
| Linear project not found | Search by name, suggest matches |
| No evidence found | Report "no changes detected", skip updates |
| User rejects all proposals | Skip updates, log rejection |
| Linear update fails | Report error, continue with other updates |

---

## Safety Guarantees

1. **No git writes** - Only reads git state, never pushes/commits
2. **Batch approval for Linear** - All updates shown first
3. **Batch approval for Graphiti** - Decisions require confirmation
4. **Evidence-based** - Every proposal cites source
5. **Audit trail** - Sync event logged with details
6. **Rollback context** - Previous state stored in Graphiti

---

## Example Output

```markdown
## Project Sync: PE Evaluation

**Directory**: ~/RS42/pe-eval
**Linear Project**: PE Evaluation
**Graphiti Group**: work:pe-eval

---

### Evidence Gathered

**Git Activity (Last 3 Days)**:
- 12 commits on feature/auth branch
- Key commits:
  - "Complete auth flow implementation (RS4-123)"
  - "Add API rate limiting"
  - "Fix edge case in token refresh"

**Recent Notes (3)**:
- "Auth implementation complete" - mentions RS4-123 done
- "API design decisions" - contains decision to store
- "Sprint 3 planning" - general context

**Current Linear State**:
- RS4-123: In Progress (should be Done)
- RS4-124: Pending (should be In Progress)
- RS4-125: Blocked (correct)

---

### Proposals

#### Linear Task Updates

- [x] **RS4-123**: In Progress → Done
      Git: "Complete auth flow implementation"
      Note: "Auth implementation complete"
      Confidence: **High**

- [x] **RS4-124**: Pending → In Progress
      Git: 5 commits in api/
      Confidence: **Medium**

#### Decisions to Store

- [x] **API Rate Limiting**: 100 req/min per user
      Source: api-design-decisions.md
      Not yet in Graphiti memory

**Apply these changes? (yes/no/edit):** yes

---

### Updates Applied

**Linear**:
✅ RS4-123 → Done (comment added with evidence)
✅ RS4-124 → In Progress (comment added)

**Graphiti**:
✅ Decision stored: "API Rate Limiting"
✅ Sync event logged

---

### Summary

- **Tasks Updated**: 2
- **Decisions Stored**: 1
- **Sync Duration**: 45 seconds
- **Next Sync**: Run again when more evidence accumulates
```
