---
description: Sync your current project across all systems - auto-detects Linear project, gathers evidence from project directory, Obsidian, and Graphiti, then proposes Linear updates with batch approval
---

# Project Sync Command

You are now in **Project Sync Mode**. Help the user sync their current project across all systems (Linear, Obsidian, Graphiti) by gathering evidence and proposing Linear updates.

## Command Purpose

Perform a project-centric sync that:
1. Auto-detects which Linear project the user is working on
2. Gathers evidence from multiple sources (project dir, Obsidian, Graphiti)
3. Compares evidence against current Linear task status
4. Proposes updates with batch approval (Full CRUD)
5. Stores sync insights to Graphiti for future context

**CRITICAL: This command includes Linear WRITE operations. Always use batch approval pattern.**

## Workflow

### Step 0: Spawn Project Analyst Agent (DEFAULT)

**Always spawn the `project-analyst` agent for project syncs.**

Project sync is a complex, repeatable workflow that benefits from:
- Fresh 200K token context (no pollution from ongoing conversation)
- Comprehensive cross-system analysis (Git + Obsidian + Linear + Graphiti)
- Consistent, thorough evidence gathering
- Main thread context preservation for follow-up work

**Spawn the agent immediately:**
```
Use the Task tool with:
- subagent_type: "chief-of-staff:project-analyst"
- prompt: "Perform a comprehensive project sync for [detected project name].

  **Your mission:**
  1. Auto-detect the Linear project from current directory: [current directory path]
  2. Gather evidence from ALL sources:
     - Git: status, recent commits, uncommitted changes
     - Project CLAUDE.md and local docs
     - Obsidian notes matching project keywords
     - Graphiti facts for project group_id
  3. Get ALL Linear issues for the project (any state)
  4. Compare evidence against Linear status
  5. Return proposed updates in batch approval format

  **Output MUST include:**
  - Project identification (which Linear project matched)
  - Evidence sources consulted
  - Proposed Linear updates with evidence citations
  - Batch approval checklist format

  **CRITICAL:** Return the batch approval proposal. Do NOT execute Linear writes - the main thread handles that after user approval."
```

**Exception - Handle directly only when:**
- User explicitly requests "quick sync" or "fast sync"
- Only checking status of 1-2 specific issues (not full project)
- Context is already minimal and user wants immediate response

### Step 1: Auto-Detect Project (Agent Handles This)

Determine which Linear project the user is working on:

1. **Check current working directory name**:
   ```
   # Example: ~/RS42/pe-eval/ → "pe-eval"
   # Example: ~/RS42/cshc-smartops/ → "cshc-smartops"
   ```

2. **Search Linear projects for match**:
   ```python
   mcp__linear__list_projects()

   # Match logic (in order):
   # 1. Exact slug match: "pe-eval" matches "Pe-eval"
   # 2. Partial name match: "pe-eval" matches "Pe-eval: AI Document Analysis System"
   # 3. Case-insensitive: "CSHC" matches "CSHC SmartOps"
   ```

3. **If ambiguous, check project CLAUDE.md**:
   - Look for Linear project reference in CLAUDE.md
   - Look for project name mentions

4. **If still ambiguous, ask user**:
   ```
   I found multiple possible Linear projects:
   1. Pe-eval: AI Document Analysis System
   2. Pharmatec Security & Maintenance

   Which project are you working on?
   ```

### Step 2: Gather Evidence from All Sources

Once project is identified, gather from 5 sources:

#### Source 1: Git Analysis (Priority 1 - HIGHEST)

**Git is the source of truth for actual work completed.** Review uncommitted changes and recent history:

```bash
# Check current working tree status
git status

# View uncommitted changes (staged and unstaged)
git diff
git diff --cached

# Recent commit history (last 7 days or 20 commits)
git log --oneline --since="7 days ago" -20

# Detailed view of recent commits with files changed
git log --stat --since="7 days ago" -10

# Show specific commit details if relevant
git show [commit-hash]
```

**What to extract**:
- **Uncommitted changes**: Work in progress, what's being actively developed
- **Recent commits**: Completed work, features merged, bugs fixed
- **Commit messages**: Often reference Linear issue IDs (e.g., "RS4-123: Fix auth")
- **Files changed**: Which parts of codebase were touched
- **Branch name**: May indicate current task (e.g., `feat/RS4-123-auth-flow`)

**Matching commits to Linear issues**:
```python
# Look for patterns like "RS4-123", "EVNK-45" in commit messages
# These directly link to Linear issues and prove work completion
```

#### Source 2: Project Directory (Priority 1)

Read CLAUDE.md and local documentation:

```python
# Read project CLAUDE.md (if exists)
# Use standard Read tool for local files
Read(file_path="./CLAUDE.md")

# Check for docs/ or notes/ folder
Glob(pattern="./docs/**/*.md")
Glob(pattern="./notes/**/*.md")

# Read recent local documentation
```

**What to extract**:
- Current project status
- Recent work completed
- Blockers mentioned
- Decisions made

#### Source 3: Obsidian Vault (Priority 2)

Use both simple text search and complex structured queries:

```python
# Method 1: Simple text search for project keywords
mcp__MCP_DOCKER__obsidian_simple_search(
    query="[project-name] OR [project-keywords]",
    context_length=200
)

# Method 2: Complex search with JsonLogic for structured filtering
# Find notes in Main Notes folder with project-related content
mcp__MCP_DOCKER__obsidian_complex_search(
    query={
        "glob": ["1 - Main Notes/**/*[project-name]*.md", {"var": "path"}]
    }
)

# Find notes modified recently in specific folder
mcp__MCP_DOCKER__obsidian_complex_search(
    query={
        "and": [
            {"glob": ["1 - Main Notes/**/*.md", {"var": "path"}]},
            {"glob": ["*[project-keyword]*", {"var": "basename"}]}
        ]
    }
)

# Get recent changes that might relate to project
mcp__MCP_DOCKER__obsidian_get_recent_changes(days=7, limit=20)

# Read relevant notes
mcp__MCP_DOCKER__obsidian_batch_get_file_contents(
    filepaths=[...relevant paths...]
)
```

**Complex Search JsonLogic Operators**:
- `glob`: Pattern matching on path/basename
- `regexp`: Regex matching
- `and`, `or`, `not`: Logical operators
- `var`: Access properties like "path", "basename", "tags"

**What to extract**:
- Technical decisions documented
- Implementation notes
- Meeting notes mentioning project
- Any completion evidence

#### Source 4: Linear Issues (Priority 3)

Get all project issues:

```python
# Get project details
mcp__linear__get_project(query="[project-name]")

# Get ALL issues for project (any state)
mcp__linear__list_issues(
    project="[project-name]",
    limit=100
)

# Or get by team if no formal project
mcp__linear__list_issues(
    team="RS42",  # or "Evonik"
    query="[project-keywords]"
)
```

**What to extract**:
- Current task statuses
- In Progress tasks
- Blocked tasks
- Recently completed tasks

#### Source 5: Graphiti Memory (Priority 4)

Query project-specific context:

```python
# Search for project facts
mcp__graphiti__search_memory_facts(
    query="[project-name] status decisions blockers",
    group_ids=["[project-slug]"],
    max_facts=15
)

# Get project episodes
mcp__graphiti__get_episodes(
    group_ids=["[project-slug]"],
    max_episodes=10
)
```

**What to extract**:
- Historical decisions
- Previous blockers and resolutions
- Project evolution context

### Step 3: Analyze Evidence vs Linear Status

Compare gathered evidence against Linear:

1. **Find completion evidence**: Tasks with evidence of being done
2. **Find blocker evidence**: Tasks that are blocked
3. **Find new work**: Work mentioned in notes not tracked in Linear
4. **Find stale tasks**: In Progress tasks with no recent activity

### Step 4: Propose Updates (Batch Approval - MANDATORY)

**NEVER execute Linear writes without user approval.**

Present proposed updates in this format:

```markdown
## 📝 Proposed Linear Updates for [Project Name]

**Project**: [Linear Project Name]
**Team**: [RS42 or Evonik]
**Evidence Sources**: Project CLAUDE.md, 3 Obsidian notes, Graphiti facts

---

### ✅ Tasks to Mark Complete

- [ ] **[RS4-36]** "Complete Workflow 2 testing"
      📄 Evidence: CLAUDE.md shows "all 8 nodes operational, email bug fixed"
      📊 Status: In Progress → Done

- [ ] **[RS4-44]** "Complete system validation"
      📄 Evidence: Note "Pe-eval System Architecture.md" mentions "fully operational since Oct 2024"
      📊 Status: In Progress → Done

---

### 🚧 Tasks to Mark Blocked

- [ ] **[RS4-29]** "End-to-end validation with sample docs"
      📄 Evidence: CLAUDE.md shows "Blocking Issue: Google Sheets authentication broken"
      📊 Status: In Progress → Blocked
      💭 Suggested comment: "Blocked on Google Sheets/Drive authentication"

---

### ➕ New Tasks to Create

- [ ] **"Fix Google Sheets authentication"**
      📄 Source: CLAUDE.md "Service account credentials need to be regenerated"
      🏷️ Team: RS42
      📁 Project: Pe-eval
      🔥 Priority: High

- [ ] **"Update deployment documentation"**
      📄 Source: Note mentions undocumented deployment steps
      🏷️ Team: RS42
      📁 Project: Pe-eval
      🔥 Priority: Normal

---

### 💬 Comments to Add

- [ ] **[RS4-36]** Add completion summary
      Content: "✅ Completed Nov 24, 2025: All 8 workflow nodes validated. Email delivery bug fixed. System ready for client handoff."

---

**Review these changes carefully.**

✅ Type "yes" or "approve" to apply all updates
📝 Or specify which items to apply (e.g., "apply tasks 1 and 2 only")
❌ Type "no" to cancel
```

---

## Main Thread: After Agent Returns

The agent returns a batch approval proposal. The main thread then:

### Step 5: Present Agent's Proposal to User

Display the agent's batch approval proposal exactly as returned. The proposal should already be in the correct format with evidence citations.

### Step 6: Wait for User Approval

User must explicitly confirm:
- "Yes" / "yes" / "approve" → Apply all
- "Apply 1, 2, 3" → Apply specific items
- "No" / "cancel" → Cancel all

### Step 6: Execute Approved Updates

Only after confirmation:

```python
# Update issue status
mcp__linear__update_issue(
    id="[issue-id]",
    state="Done"  # or "Blocked"
)

# Create new issue
mcp__linear__create_issue(
    team="RS42",
    title="Fix Google Sheets authentication",
    description="Service account credentials need to be regenerated...",
    project="Pe-eval",
    priority=2  # High
)

# Add comment
mcp__linear__create_comment(
    issueId="[issue-id]",
    body="✅ Completed Nov 24, 2025: All 8 workflow nodes validated..."
)
```

### Step 7: Store Sync to Graphiti

After updates, store sync insights:

```python
mcp__graphiti__add_memory(
    name="Project Sync - [Project Name] - [Date]",
    episode_body="Synced [Project Name]. Completed: [list]. Blocked: [list]. Created: [list].",
    source="text",
    source_description="Project sync session",
    group_id="[project-slug]"  # Project-specific group
)
```

### Step 8: Generate Sync Report

Present summary:

```markdown
# Project Sync Complete - [Project Name]

**Date**: [Date]
**Project**: [Linear Project Name]

## ✅ Updates Applied

### Status Changes
- [RS4-36] → Done
- [RS4-29] → Blocked

### New Tasks Created
- [RS4-XX] "Fix Google Sheets authentication"
- [RS4-YY] "Update deployment documentation"

### Comments Added
- [RS4-36]: Completion summary

## 📊 Project Status After Sync

- **Total Tasks**: X
- **Completed**: Y
- **In Progress**: Z
- **Blocked**: W

## 💾 Stored to Graphiti

Project insights saved for future context.

---

Would you like me to:
- Generate a project brief? (`/project-brief`)
- Continue working on specific tasks?
- Review another project?
```

## Project Mapping Reference

Common project directory to Linear mappings:

| Directory Pattern | Linear Project |
|-------------------|----------------|
| `pe-eval`, `pe_eval` | Pe-eval: AI Document Analysis System |
| `cshc`, `cshc-smartops` | CSHC SmartOps |
| `hp-smartops`, `hp_smartops` | HP Smartops |
| `pharmatec` | Pharmatec Security & Maintenance |
| `sourcing-agent` | Sourcing Agent - Claude Code Plugin |
| `accelerator-scout` | Accelerator Scout - Single Agent Demo |

## Team Mapping

| Project Type | Team |
|--------------|------|
| Internal/Product work | RS42 |
| Client work (Evonik) | Evonik |

## Evidence Requirements

Only propose updates when evidence is clear:

**DO propose when**:
- CLAUDE.md explicitly states completion
- Notes document finished work
- Testing/validation documented as passed
- Blockers explicitly mentioned

**DON'T propose when**:
- Evidence is ambiguous
- Notes are just planning (not execution)
- Status unclear from context
- Would create duplicate tasks

## Safety Guardrails

### Batch Approval Pattern (MANDATORY)

1. ✅ **DO**: Always propose updates first
2. ✅ **DO**: Show evidence for each proposed change
3. ✅ **DO**: Wait for explicit user confirmation
4. ✅ **DO**: Confirm successful updates after execution
5. ✅ **DO**: Store sync to Graphiti with project group_id

6. ❌ **DON'T**: Auto-update Linear without approval
7. ❌ **DON'T**: Guess at task status without evidence
8. ❌ **DON'T**: Create tasks for minor notes
9. ❌ **DON'T**: Update more than 10 tasks without extra confirmation

## Error Handling

- **Project not found**: List available projects, ask user to select
- **Ambiguous match**: Present options, let user choose
- **Linear write fails**: Show error, don't retry automatically
- **Evidence unclear**: Skip that update, note it for user
- **No updates needed**: Report that project is in sync

## Related Commands

- `/project-brief [name]` - Deep dive into project status (read-only)
- `/evening-sync` - Personal daily sync across all projects
- `/morning-brief` - Daily overview of all work
