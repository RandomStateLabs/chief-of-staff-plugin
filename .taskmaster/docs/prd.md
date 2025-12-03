# Chief of Staff Plugin - Product Requirements Document

**Status**: Draft - Alignment Phase
**Last Updated**: 2025-11-30
**Author**: RS42 + Claude

---

## Executive Summary

The Chief of Staff plugin is a Claude Code plugin that provides AI-assisted project management by integrating:

- **Obsidian** (knowledge management)
- **Linear** (task tracking)
- **Graphiti** (temporal knowledge graph)
- **GitHub** (code repositories)

**Vision**: Create a unified project-centric development system where AI assists with project creation, task management, note capture, and knowledge synthesis across all systems.

## Current State

### Linear Projects

Our Linear project template is:

```markdown
 # Project Overview

### Repository
**GitHub**: [repo-url]

### Goal
[Transform X into Y using Z]

### Target Users
- [User type 1]
- [User type 2]

### Platform & Distribution
- **Platform**: [tech stack]
- **Client**: [who's paying]
- **Deployment**: [how it's deployed]

## Technology Stack
- **[Category]**: [Technology]
- ...

## Success Criteria
- [ ] Criterion 1
- [ ] Criterion 2

## Development Roadmap

### Phase 1: [Phase Name] [Status Emoji]
- [RS4-XX](link) Task description
- ...

### Phase 2: [Phase Name] [Status Emoji]
- ...

## Related Projects
- **Linear Project**: [Link]
```

### Graphiti

- **group_id**: `"work"` for all operations

# Proposed Architecture

### The Project-Centric Workflow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PROJECT LIFECYCLE                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. PROJECT CREATION (/create-project "Project Name")                        │
│     ┌──────────────────────────────────────────────────────────────────┐    │
│     │  Creates and links all systems:                                  │    │
│     │  ├── GitHub: Create/link repository                              │    │
│     │  ├── Linear: Create project with TEMPLATE                        │    │
│     │  ├── Local: Scaffold project folder structure                    │    │
│     │  ├── Graphiti: Initialize group_id "work:[project-slug]"         │    │
│     │  └── (Optional) TaskMaster: Init .taskmaster/ for PRD breakdown  │    │
│     └──────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  2. DURING DEVELOPMENT                                                       │
│     ┌──────────────────────────────────────────────────────────────────┐    │
│     │  Notes → Project Folder (not main Obsidian vault)                │    │
│     │  Tasks → Two-view system (see Decision 6):                       │    │
│     │    • Linear: Stakeholder view (milestones, phases)               │    │
│     │    • TaskMaster: Developer view (subtasks, implementation)       │    │
│     │  Memory → Graphiti with project group_id                         │    │
│     │  Sync → /sync-tasks for bidirectional Linear ↔ TaskMaster        │    │
│     └──────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  3. NOTE AUDIT (/audit-notes)                                                │
│     ┌──────────────────────────────────────────────────────────────────┐    │
│     │  Before syncing to main vault:                                   │    │
│     │  ├── Atomic check (single idea per note)                         │    │
│     │  ├── Duplicate check (semantic similarity)                       │    │
│     │  ├── Knowledge graph placement (tags, links, folder)             │    │
│     │  └── Batch approval (propose → approve → execute)                │    │
│     └──────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  4. PROJECT SYNC (/project-sync)                                             │
│     ┌──────────────────────────────────────────────────────────────────┐    │
│     │  Sync state across systems:                                      │    │
│     │  ├── Git activity → Linear task updates                          │    │
│     │  ├── Notes evidence → Task completion proposals                  │    │
│     │  └── Decisions → Graphiti memory                                 │    │
│     └──────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Decision Points

### Decision 1: Project Folder Structure ✅ DECIDED

**Choice**: Option A - Notes in project directory with Task Master integration

```
~/RS42/[project-name]/
├── .claude/
│   └── settings.json           # Claude Code configuration
├── .taskmaster/                # Task Master (Developer View) ⭐
│   ├── tasks/                  # Task files
│   │   ├── tasks.json          # Main task database
│   │   └── task-*.md           # Individual task markdown files
│   ├── docs/                   # Project-local notes
│   │   └── prd.md              # PRD
│   ├── reports/                # Analysis reports
│   │   └── task-complexity-report.json
│   ├── templates/              # 
│   ├── config.json             # AI model configuration
│   ├── state.json              # State tracking
│   ├── CLAUDE.md               # Task Master Claude instructions
│   └── AGENT.md                # Task Master agent instructions
├── CLAUDE.md                   # Project context for Claude
├── README.md                   # Standard readme
├── LICENSE                     # License file
├── src/                        # Source code
```

**Key Points**:

- **Task Master** (`/.taskmaster/`) handles all developer-facing task management
- **Linear** is accessed via MCP API (no local mirroring needed)
- **Notes** stay local until audited and synced to Obsidian vault

### Decision 2: Linear Issue Mirrors Main Project Milestones

**Rationale**:

- Task Master handles all developer-facing task details in `.taskmaster/`
- Linear is the "Stakeholder View" accessed via `mcp__linear__*` tools

### Decision 3: Graphiti Group Structure 🟡 NEEDS CONFIRMATION

**Proposed**: Option C - Hierarchical naming

```python
# Cross-project insights
group_id = "work"

# Project-specific
group_id = "work:pe-eval"
group_id = "work:chief-of-staff"
group_id = "work:sourcing-agent"
```

**Benefits**:

- Query one project: `group_ids=["work:pe-eval"]`
- Query all work: `group_ids=["work"]` (may need to list all sub-groups)
- Clear organization

**Question**: Does Graphiti support hierarchical group_id querying, or do we need to query each explicitly?

### Decision 5: Linear Issue Template 🔴 NEEDS DEFINITION

**Proposed template**:

```yaml
# Linear Issue Template
title_format: "[Type]: [Brief description]"
types:
  - "feat"     # New feature
  - "fix"      # Bug fix
  - "chore"    # Maintenance
  - "docs"     # Documentation
  - "refactor" # Code refactoring
  - "test"     # Testing

description_template: |
  ## Context
  [Why this issue exists, what problem it solves]

  ## Acceptance Criteria
  - [ ] Criterion 1
  - [ ] Criterion 2

  ## Technical Notes
  [Implementation details, constraints, dependencies]

  ## Related
  - Note: [[related-note]]
  - Issue: RS4-XX
  - Commit: [sha]

default_fields:
  team: "auto-detect from project"
  project: "auto-detect from directory"
  priority: 3  # Normal
  labels: ["dev"]
```

**Question**: What title format do you prefer? What sections are essential?

### Decision 6: TaskMaster + Linear Integration ✅ DECIDED

**Core Concept**: Two-view system where TaskMaster serves developers and Linear serves stakeholders.


| System         | Role             | Contains                                                   | Who Uses It                       |
| ---------------- | ------------------ | ------------------------------------------------------------ | ----------------------------------- |
| **Linear**     | Stakeholder View | High-level milestones, phases, priorities                  | PMs, stakeholders, team leads     |
| **TaskMaster** | Developer View   | Granular subtasks, implementation details, test strategies | Developers during coding sessions |

**Why Two Systems?**

- Stakeholders don't need to see "15.3.2: Fix edge case in parser"
- Developers need granular tracking that would clutter Linear
- Linear is the "contract" (what we promised), TaskMaster is "how we deliver"

**Sync Direction**: Bidirectional with clear rules


| Direction            | When                            | What Syncs                                  |
| ---------------------- | --------------------------------- | --------------------------------------------- |
| Linear → TaskMaster | Stakeholder changes milestone   | Priority, scope, requirements cascade down  |
| TaskMaster → Linear | Developer completes parent task | Milestone marked done (subtasks stay local) |

**Key Principle**: Linear changes are authoritative. When a stakeholder updates a Linear issue, TaskMaster tasks may need to be re-expanded or updated to reflect new scope.

**Cascade Updates**: When Linear scope changes significantly:

1. Detect what changed (priority? scope expansion? requirement pivot?)
2. Analyze impact on TaskMaster subtasks
3. Propose updates (add subtasks, modify existing, defer others)
4. Human approval before executing changes
5. Store change history in Graphiti

**Workflow**:

```
PRD.md → TaskMaster parse-prd → tasks.json (developer works here)
                                     ↓
                              /project-sync
                                     ↓
                    ┌────────────────┴────────────────┐
                    ↓                                 ↓
          Linear milestone complete ←── Parent task done
          (stakeholder sees progress)   (subtasks stay in TaskMaster)
```

**What Does NOT Sync**:

- Subtask completions (15.1, 15.2 done) → stays local
- Implementation notes → stays local
- Test strategies → stays local

**What DOES Sync**:

- Parent task completion → Linear milestone done
- Linear priority change → TaskMaster priority update
- Linear scope change → TaskMaster re-expansion (with approval)

---

### Decision 7: Note Audit Process 🔴 NEEDS DEFINITION

**Proposed workflow**:

```
1. COLLECT
   - Gather notes from project/notes/sync-queue/

2. ANALYZE EACH NOTE
   For each note:
   ├── Atomic check: Is it focused on ONE idea?
   │   - If >500 words or multiple topics → Propose: SPLIT
   │
   ├── Duplicate check: Similar notes exist?
   │   - Search Obsidian vault for similar content
   │   - Search Graphiti for similar facts
   │   - If similar found → Propose: MERGE or LINK
   │
   └── Placement check: Where should it go?
       - Suggest folder (1 - Main Notes/, etc.)
       - Suggest tags (#project, #architecture, etc.)
       - Suggest wikilinks ([[Related Note]])

3. BATCH PROPOSAL
   Present all recommendations:
   ┌────────────────────────────────────────────┐
   │ ## Ready to Add to Vault                   │
   │ - [ ] "Auth Decision" → 1 - Main Notes/    │
   │       Tags: #architecture, #auth           │
   │                                            │
   │ ## Needs Split                             │
   │ - [ ] "Long Note" → Split into 3 notes     │
   │                                            │
   │ ## Merge with Existing                     │
   │ - [ ] "API v2" → Merge into [[API Design]] │
   │                                            │
   │ Approve? (yes/no/edit)                     │
   └────────────────────────────────────────────┘

4. EXECUTE (After Approval)
   - Move notes to Obsidian vault
   - Apply frontmatter, tags, links
   - Update Graphiti with new facts
   - Archive processed notes
```

**Questions**:

- What makes a note "atomic" enough? (word count? topic count?)
- How aggressive should duplicate detection be?
- Should we auto-create wikilinks or just suggest them?

---

## Commands (Proposed)


| Command                  | Purpose                                                   | Status                    |
| -------------------------- | ----------------------------------------------------------- | --------------------------- |
| `/morning-brief`         | Daily project status synthesis                            | Exists (needs refinement) |
| `/evening-sync`          | End-of-day Linear updates                                 | Exists (needs refinement) |
| `/project-brief [name]`  | Deep dive into specific project                           | Exists (needs refinement) |
| `/project-sync`          | Sync current project across systems                       | Exists (needs testing)    |
| `/create-project [name]` | Scaffold new project with all integrations                | **NEW - Proposed**        |
| `/capture-note`          | Write note to project's local folder                      | **NEW - Proposed**        |
| `/audit-notes`           | Review project notes before vault sync                    | **NEW - Proposed**        |
| `/create-issue`          | Create Linear issue with template                         | **NEW - Proposed**        |
| `/sync-tasks`            | Bidirectional sync: Linear ↔ TaskMaster (see Decision 6) | **NEW - Proposed**        |

---

## Agents (Proposed)


| Agent              | Purpose                            | Status             |
| -------------------- | ------------------------------------ | -------------------- |
| `morning-planner`  | Morning briefing synthesis         | Exists             |
| `evening-reviewer` | Evening sync with Linear updates   | Exists             |
| `project-analyst`  | Deep project analysis              | Exists             |
| `project-creator`  | Scaffolds new projects             | **NEW - Proposed** |
| `note-auditor`     | Semantic analysis, vault placement | **NEW - Proposed** |

---

## Skills (Current + Proposed)


| Skill                      | Purpose                                                        | Status             |
| ---------------------------- | ---------------------------------------------------------------- | -------------------- |
| `obsidian-reader`          | Read from Obsidian vault                                       | Exists             |
| `linear-integration`       | Linear read/write with batch approval                          | Exists             |
| `graphiti-memory`          | Knowledge graph operations                                     | Exists             |
| `linear-project-templates` | Project/issue formatting                                       | Exists (empty)     |
| `github-integration`       | Repo creation, scaffolding                                     | **NEW - Proposed** |
| `note-capture`             | Project-local note writing                                     | **NEW - Proposed** |
| `semantic-analysis`        | Duplicate detection, similarity                                | **NEW - Proposed** |
| `task-sync`                | Bidirectional TaskMaster ↔ Linear sync with cascade detection | **NEW - Proposed** |

---

## MCP Tool Specifications

### For Each System, Define:

#### Obsidian

```yaml
search:
  tool: mcp__MCP_DOCKER__obsidian_simple_search
  parameters:
    query: "[project name] OR [keywords]"
    context_length: 200
  when_to_use: "Finding project-related notes"

get_file:
  tool: mcp__MCP_DOCKER__obsidian_get_file_contents
  parameters:
    filepath: "path/to/note.md"
  when_to_use: "Reading specific known note"

get_recent:
  tool: mcp__MCP_DOCKER__obsidian_get_recent_changes
  parameters:
    days: 7
    limit: 15
  when_to_use: "Finding recently modified notes"
```

#### Linear

```yaml
list_issues:
  tool: mcp__linear__list_issues
  parameters:
    team: "RS42"  # Query by TEAM, not assignee
  when_to_use: "Getting all team issues"
  note: "Filter for project/status locally after retrieval"

create_issue:
  tool: mcp__linear__create_issue
  parameters:
    team: "RS42"
    title: "[type]: [description]"
    project: "[project name]"
    description: "[template]"
  when_to_use: "Creating new task"
  requires: "Batch approval pattern"

update_issue:
  tool: mcp__linear__update_issue
  parameters:
    id: "[issue-id]"
    state: "Done"  # or "Blocked", "In Progress"
  when_to_use: "Updating task status"
  requires: "Batch approval pattern"
```

#### Graphiti

```yaml
search_facts:
  tool: mcp__graphiti__search_memory_facts
  parameters:
    query: "[project name] decisions blockers status"
    group_ids: ["work"]  # or ["work:project-slug"]
    max_facts: 15
  when_to_use: "Finding project history"

add_memory:
  tool: mcp__graphiti__add_memory
  parameters:
    name: "[descriptive name]"
    episode_body: "[content]"
    group_id: "work"  # or "work:project-slug"
    source: "text"
  when_to_use: "Storing decisions, insights, sync results"
```

---


## Open Questions

1. **Graphiti hierarchical groups**: Can we query `group_ids=["work"]` and get all sub-groups (`work:pe-eval`, `work:cshc`, etc.)? Or do we need to list them explicitly?
2. **Linear project auto-detection**: How should we map directory names to Linear projects? Exact match? Fuzzy match? CLAUDE.md reference?
3. **Note atomic threshold**: What defines "atomic"?

   - Word count (<500 words)?
   - Heading count (1 main heading)?
   - Topic detection (AI judges)?
4. **Duplicate sensitivity**: How similar is "too similar"?

   - Exact title match?
   - 80% content overlap?
   - Same key concepts?
5. **GitHub integration**: Create new repos automatically, or just link existing ones?

---

## Next Steps

1. **Review this PRD** - Confirm decisions, answer open questions
2. **Define Linear templates** - Finalize project and issue formats
3. **Test Graphiti groups** - Verify hierarchical naming works
4. **Build incrementally** - Start with Phase 1, test, iterate

---

## References

- [Chief of Staff Architecture Note](obsidian://open?vault=Vault&file=1%20-%20Main%20Notes%2FChief%20of%20Staff%20Plugin%20-%20Complete%20Architecture%20and%20Testing%20Guide)
- [Plugin-Dev Toolkit](obsidian://open?vault=Vault&file=1%20-%20Main%20Notes%2FPlugin-Dev%20Toolkit%20-%20Claude%20Code%20Plugin%20Development%20Framework)

---

## Ideas from Patterns from Research

**From TaskMaster**:

- PRD-to-tasks conversion with AI (`parse-prd` command)
- Structured task storage (`.taskmaster/tasks/tasks.json`)
- MCP + CLI integration (works with Claude Code subscription - no extra API costs)
- **Integration approach**: TaskMaster = Developer View, Linear = Stakeholder View (see Decision 6)

### Graphiti (Current State)

- **group_id**: `"work"` for all operations
- **No hierarchical structure** yet - Does this work and how we want her to explore this a little bit further?
