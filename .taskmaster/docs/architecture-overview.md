# Chief of Staff Plugin - Architecture Overview

> **Purpose**: This document captures the architectural decisions, skill design philosophy, and component relationships for the Chief of Staff plugin. Use this for understanding how commands, agents, skills, and tools connect.

---

## Table of Contents

1. [Architectural Decisions](#architectural-decisions)
2. [Skill Design Philosophy](#skill-design-philosophy)
3. [Agent Spawning Strategy](#agent-spawning-strategy)
4. [Tool Permission Model](#tool-permission-model)
5. [Skill Inventory](#skill-inventory)
6. [Agent Inventory](#agent-inventory)
7. [Feature-to-Skill Coverage Matrix](#feature-to-skill-coverage-matrix)
8. [Command Flow Pattern](#command-flow-pattern)

---

## Architectural Decisions

### 1. Fat Skills with Read/Write Documentation

**Decision**: Use comprehensive skills that contain all tools for a domain (Fat Skills), with clear documentation distinguishing read vs write operations.

**Why not split Read/Write into separate skills?**

- Skills auto-load based on description matching - splitting creates confusion about when each loads
- The batch approval pattern is enforced by **agent instructions**, not tool restrictions
- A single `linear-integration` skill is easier to maintain than `linear-reader` + `linear-writer`

### 2. Batch Approval Pattern for All Writes

**Decision**: All write operations to external systems (Linear, GitHub, Graphiti) MUST follow the batch approval pattern.

**Pattern**:

1. **Analyze** - Determine what should change
2. **Propose** - Show user in checklist format with evidence
3. **Wait** - Explicitly ask for approval
4. **Confirm** - User must say yes
5. **Execute** - Only then write to system
6. **Verify** - Confirm success

**Never** auto-write without approval.

### 3. Conservative Git Permissions

**Decision**: Use explicit, read-only git command permissions. NO wildcards like `Bash(git:*)`.

**Approved Git Commands**:

```yaml
# Read-only (safe)
- Bash(git status:*)
- Bash(git log:*)
- Bash(git diff:*)
- Bash(git branch -a:*)
- Bash(git branch -l:*)
- Bash(git remote -v:*)
- Bash(git rev-parse:*)
- Bash(git show:*)

# DANGEROUS - Require explicit user request
- Bash(git push:*)      # ❌ NOT auto-allowed
- Bash(git commit:*)    # ❌ NOT auto-allowed
- Bash(git checkout:*)  # ❌ NOT auto-allowed
- Bash(git reset:*)     # ❌ NOT auto-allowed
```

### 4. GitHub Remote Read Access

**Decision**: /morning-brief and /project-brief need GitHub READ access for remote repository state.

**Allowed GitHub Read Operations**:

```yaml
- mcp__MCP_DOCKER__get_file_contents
- mcp__MCP_DOCKER__search_repositories
- mcp__MCP_DOCKER__list_commits
- mcp__MCP_DOCKER__list_pull_requests
- mcp__MCP_DOCKER__get_pull_request
- mcp__MCP_DOCKER__get_pull_request_status
- mcp__MCP_DOCKER__search_code

# Write operations - Require explicit approval
- mcp__MCP_DOCKER__create_repository    # ⚠️ APPROVAL REQUIRED
- mcp__MCP_DOCKER__create_pull_request  # ⚠️ APPROVAL REQUIRED
- mcp__MCP_DOCKER__push_files           # ⚠️ APPROVAL REQUIRED
```

---

## Agent Spawning Strategy

**Decision Matrix**: When should commands spawn agents vs handle directly?


| Command                  | Spawn Agent? | Reasoning                                   |
| -------------------------- | -------------- | --------------------------------------------- |
| `/project-brief [name]`  | ✅ Yes       | 200K context for multi-system synthesis     |
| `/capture-note`          | ❌ No        | Single MCP call, stays in main context      |
| `/create-project [name]` | ✅ Yes       | Multi-system scaffolding, error handling    |
| `/create-issue`          | ❌ No        | Simple templated creation with approval     |
| `/audit-notes`           | ✅ Yes       | Heavy processing, may need multiple passes  |
| `/evening-sync`          | ✅ Yes       | Complex evidence gathering + batch approval |
| `/project-sync`          | ✅ Yes       | Multi-system coordination, batch approval   |
| `/sync-tasks`            | ✅ Yes       | Bidirectional sync, cascade detection       |
| `/morning-brief`         | ✅ Yes       | 200K context for cross-project synthesis    |

**Guidelines**:

- **Spawn agent when**: Complex multi-step synthesis, 200K context needed, multiple systems coordinated
- **Stay in main context when**: Single operation, simple approval, no cross-system synthesis

---

## Tool Permission Model

### By System

#### Obsidian MCP

```yaml
Read (Auto-allowed):
  - mcp__MCP_DOCKER__obsidian_simple_search
  - mcp__MCP_DOCKER__obsidian_get_file_contents
  - mcp__MCP_DOCKER__obsidian_batch_get_file_contents
  - mcp__MCP_DOCKER__obsidian_list_files_in_dir
  - mcp__MCP_DOCKER__obsidian_list_files_in_vault
  - mcp__MCP_DOCKER__obsidian_complex_search
  - mcp__MCP_DOCKER__obsidian_get_recent_changes
  - mcp__MCP_DOCKER__obsidian_get_periodic_note
  - mcp__MCP_DOCKER__obsidian_get_recent_periodic_notes

Write (Approval Required):
  - mcp__MCP_DOCKER__obsidian_append_content
  - mcp__MCP_DOCKER__obsidian_patch_content
  - mcp__MCP_DOCKER__obsidian_delete_file
```

#### Linear MCP

```yaml
Read (Auto-allowed):
  - mcp__linear__list_issues
  - mcp__linear__get_issue
  - mcp__linear__list_projects
  - mcp__linear__get_project
  - mcp__linear__list_teams
  - mcp__linear__get_team
  - mcp__linear__list_users
  - mcp__linear__get_user
  - mcp__linear__list_comments
  - mcp__linear__list_cycles
  - mcp__linear__list_issue_statuses
  - mcp__linear__list_issue_labels
  - mcp__linear__list_documents
  - mcp__linear__get_document

Write (Batch Approval Required):
  - mcp__linear__create_issue
  - mcp__linear__update_issue
  - mcp__linear__create_comment
  - mcp__linear__create_project
  - mcp__linear__update_project
  - mcp__linear__create_issue_label
```

#### Graphiti MCP

```yaml
Read (Auto-allowed):
  - mcp__graphiti__search_memory_facts
  - mcp__graphiti__search_nodes
  - mcp__graphiti__get_episodes
  - mcp__graphiti__get_entity_edge
  - mcp__graphiti__get_status

Write (Generally safe, but audit):
  - mcp__graphiti__add_memory
  - mcp__graphiti__delete_episode
  - mcp__graphiti__delete_entity_edge
  - mcp__graphiti__clear_graph  # ⚠️ DANGEROUS
```

#### GitHub MCP

```yaml
Read (Auto-allowed):
  - mcp__MCP_DOCKER__get_file_contents
  - mcp__MCP_DOCKER__search_repositories
  - mcp__MCP_DOCKER__search_code
  - mcp__MCP_DOCKER__list_commits
  - mcp__MCP_DOCKER__list_pull_requests
  - mcp__MCP_DOCKER__get_pull_request
  - mcp__MCP_DOCKER__get_pull_request_status
  - mcp__MCP_DOCKER__get_pull_request_files
  - mcp__MCP_DOCKER__get_pull_request_reviews
  - mcp__MCP_DOCKER__list_issues
  - mcp__MCP_DOCKER__get_issue
  - mcp__MCP_DOCKER__search_issues

Write (Explicit Approval Required):
  - mcp__MCP_DOCKER__create_repository
  - mcp__MCP_DOCKER__create_pull_request
  - mcp__MCP_DOCKER__push_files
  - mcp__MCP_DOCKER__create_or_update_file
  - mcp__MCP_DOCKER__create_issue
  - mcp__MCP_DOCKER__create_branch
  - mcp__MCP_DOCKER__fork_repository
```

#### Bash (Git Operations)

```yaml
Read (Auto-allowed - EXPLICIT COMMANDS ONLY):
  - Bash(git status:*)
  - Bash(git log:*)
  - Bash(git diff:*)
  - Bash(git branch -a:*)
  - Bash(git branch -l:*)
  - Bash(git branch -r:*)
  - Bash(git remote -v:*)
  - Bash(git rev-parse:*)
  - Bash(git show:*)
  - Bash(git describe:*)
  - Bash(git ls-files:*)
  - Bash(pwd:*)
  - Bash(basename:*)
  - Bash(date:*)

Write (NEVER Auto-allowed):
  - Bash(git push:*)      # ❌
  - Bash(git commit:*)    # ❌
  - Bash(git checkout:*)  # ❌
  - Bash(git reset:*)     # ❌
  - Bash(git merge:*)     # ❌
  - Bash(git rebase:*)    # ❌
```

## Agent Inventory


| Agent              | Spawned By                        | Purpose                                     | Skills It Uses                                                                           |
| -------------------- | ----------------------------------- | --------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `morning-planner`  | `/morning-brief`                  | Cross-project synthesis                     | project-context, graphiti-memory, taskmaster-docs, obsidian-reader, linear-integration, github-integration, git-operations |
| `evening-reviewer` | `/evening-sync`                   | Evidence gathering + batch Linear proposals | project-context, graphiti-memory, taskmaster-docs, obsidian-reader, linear-integration, git-operations |
| `project-analyst`  | `/project-brief`, `/project-sync` | Deep single-project analysis                | project-context, graphiti-memory, taskmaster-docs, obsidian-reader, linear-integration, github-integration, git-operations |
| `project-creator`  | `/create-project`                 | Multi-system scaffolding                    | project-context, linear-integration, graphiti-memory, github-integration                 |
| `note-auditor`     | `/audit-notes`                    | Semantic analysis + vault sync              | taskmaster-docs, obsidian-reader, obsidian-writer, graphiti-memory, semantic-analysis    |
| `task-syncer`      | `/sync-tasks`                     | Linear ↔ TaskMaster bidirectional          | linear-integration, task-sync                                                            |

---

## Feature-to-Skill Coverage Matrix

> **Note**: Obsidian is a lightweight supplement to Graphiti (limit: 5 notes). TaskMaster Docs (`.taskmaster/docs/`) is the primary local source for PRD and captured notes.

| Feature                  | Graphiti Read | TaskMaster Docs | Obsidian Read | Obsidian Write | Linear Read | Linear Write | GitHub Read | GitHub Write | Git Read |
| -------------------------- | --------------- | ----------------- | --------------- | ---------------- | ------------- | -------------- | ------------- | -------------- | ---------- |
| `/morning-brief`         | ✅ (primary)  | ✅              | ✅ (light)    |                | ✅          |              | ✅          |              | ✅       |
| `/evening-sync`          | ✅ (primary)  | ✅              | ✅ (light)    |                | ✅          | ✅           |             |              | ✅       |
| `/project-brief [name]`  | ✅ (primary)  | ✅              | ✅ (light)    |                | ✅          |              | ✅          |              | ✅       |
| `/project-sync`          | ✅ (primary)  | ✅              | ✅ (light)    |                | ✅          | ✅           |             |              | ✅       |
| `/create-project [name]` |               |                 |               |                |             | ✅           |             | ✅           |          |
| `/capture-note`          |               | ✅              |               |                |             |              |             |              |          |
| `/audit-notes`           |               | ✅              | ✅            | ✅             |             |              |             |              |          |
| `/create-issue`          |               |                 |               |                |             | ✅           |             |              |          |
| `/sync-tasks`            |               |                 |               |                | ✅          | ✅           |             |              |          |

---

## Command Flow Pattern

Every command follows this conceptual flow:

```
User Input
    │
    ▼
┌─────────────────┐
│    Command      │  ← Commands are user entry points
│  (*.md file)    │  ← They load skills and may spawn agents
└────────┬────────┘
         │
         │ Loads skill(s)
         ▼
┌─────────────────┐
│     Skill       │  ← Skills provide knowledge + tool access
│   (SKILL.md)    │  ← They auto-load based on description match
└────────┬────────┘
         │
         │ Complex task?
         ▼
    ┌────┴────┐
    │         │
 Simple    Complex
    │         │
    ▼         ▼
┌─────────┐ ┌─────────────────┐
│ Execute │ │  Spawn Agent    │  ← Agents get 200K context
│ in Main │ │  (agent.md)     │  ← They handle multi-step synthesis
│ Context │ │  with skills    │
└─────────┘ └────────┬────────┘
                     │
                     │ Activates skills
                     ▼
              ┌─────────────────┐
              │   Tool Calls    │  ← Actual MCP tool invocations
              │ (mcp__*__*)     │
              └────────┬────────┘
                       │
                       │ Write operation?
                       ▼
                  ┌────┴────┐
                  │         │
               Read      Write
                  │         │
                  ▼         ▼
            ┌─────────┐ ┌─────────────────┐
            │ Execute │ │ Batch Approval  │
            │ Directly│ │ Pattern         │
            └─────────┘ │ 1. Propose      │
                        │ 2. Wait         │
                        │ 3. Confirm      │
                        │ 4. Execute      │
                        │ 5. Verify       │
                        └─────────────────┘
```

---

## Command Flow Traces

Detailed traces for each command are available in the `command-flows/` directory, organized by priority:

### 1st Focus (Project-Scoped Commands)
- [1.1-project-brief-flow.md](./command-flows/1st-focus/1.1-project-brief-flow.md) ✅
- [1.2-capture-note-flow.md](./command-flows/1st-focus/1.2-capture-note-flow.md) ✅

### 2nd Focus (Sync Commands)
- [evening-sync-flow.md](./command-flows/2nd-focus/evening-sync-flow.md)
- [project-sync-flow.md](./command-flows/2nd-focus/project-sync-flow.md)

### 3rd Focus (Creation Commands)
- [create-project-flow.md](./command-flows/3rd-focus/create-project-flow.md)
- [create-issue-flow.md](./command-flows/3rd-focus/create-issue-flow.md)

### Future Focus
- [morning-brief-flow.md](./command-flows/future-focus/morning-brief-flow.md)
- [audit-notes-flow.md](./command-flows/future-focus/audit-notes-flow.md)
- [sync-tasks-flow.md](./command-flows/future-focus/sync-tasks-flow.md)

---

## Graphiti Group Convention

All Graphiti operations use hierarchical group_ids:

```python
group_id = "work"                    # Cross-project facts
group_id = "work:pe-eval"            # Project-specific
group_id = "work:chief-of-staff"     # This plugin's project
```

**Query Patterns**:

- Cross-project: `group_ids=["work"]`
- Single project: `group_ids=["work:project-slug"]`
- Multiple specific: `group_ids=["work:project-a", "work:project-b"]`
