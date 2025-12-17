<context>
# Overview

Chief of Staff is a Claude Code plugin that unifies project management across Obsidian (knowledge), Linear (tasks), Graphiti (temporal memory), and GitHub (code). It creates a project-centric workflow where AI handles project creation, task management, note capture, and cross-system synthesis.

# Core Features

**Project Creation** (`/create-project`): Scaffolds a new project across all systems. Creates/links GitHub repo, initializes Linear project from template, sets up local folder structure, creates Graphiti group, and optionally initializes TaskMaster for PRD breakdown.

**Two-View Task System**: Linear holds stakeholder-facing milestones and phases. TaskMaster holds developer-facing subtasks and implementation details. `/sync-tasks` handles bidirectional sync with cascade detection when Linear scope changes.

**Note Audit** (`/audit-notes`): Reviews project notes before vault sync. Checks atomicity (single idea per note), detects duplicates via semantic similarity, suggests placement (folder, tags, wikilinks), then executes after batch approval.

**Project Sync** (`/project-sync`): Syncs state across systems. Git activity updates Linear tasks, note evidence proposes task completions, decisions store to Graphiti memory.

**Daily Briefings**:

- `/project-brief [name]` - Deep-dives into a specific project (scoped to one project)
- `/project-sync` - Syncs state for current project across systems
- `/evening-sync` - Proposes Linear updates based on day's work (project-scoped)
- `/morning-brief` - Synthesizes status across ALL projects (cross-project, future phase)

> **Development Note**: We're starting with project-scoped commands (`/project-brief`, `/project-sync`, `/evening-sync`) since they're more focused and easier to test. Cross-project synthesis (`/morning-brief`) will be expanded later once project-level agents are solidified.

# User Experience

**Primary Users**: Solo developers or small teams managing multiple projects across Obsidian, Linear, and GitHub.

**Key Flows**:

1. Create project → all systems scaffold automatically
2. During development → notes stay local, tasks in TaskMaster, sync on demand
3. End of day → evening sync proposes Linear updates with evidence
4. Before vault sync → audit notes for atomicity and duplicates

**Batch Approval Pattern**: All write operations (Linear updates, note vault sync, task cascade) show proposals first, require explicit approval, then execute.
</context>

<PRD>
# Technical Architecture

## System Components

### Integrations


| System   | Tool Prefix                   | Purpose                        |
| ---------- | ------------------------------- | -------------------------------- |
| Obsidian | `mcp__MCP_DOCKER__obsidian_*` | Vault search, note read/write  |
| Linear   | `mcp__linear__*`              | Issue CRUD, project management |
| Graphiti | `mcp__graphiti__*`            | Temporal knowledge graph       |
| GitHub   | `mcp__MCP_DOCKER__*`          | Repo creation, file operations |

### Plugin Components

**Commands**:


| Command                  | Purpose                   | Status                        |
| -------------------------- | --------------------------- | ------------------------------- |
| `/morning-brief`         | Daily status synthesis    | Exists NEEDS REVISITING LATER |
| `/evening-sync`          | End-of-day Linear updates | Exists                        |
| `/project-brief [name]`  | Project deep dive         | Exists                        |
| `/project-sync`          | Cross-system sync         | Exists                        |
| `/create-project [name]` | Scaffold new project      | Exists                        |
| `/capture-note`          | Write to project folder   | Proposed                      |
| `/audit-notes`           | Pre-vault note review     | Proposed                      |
| `/create-issue`          | Templated Linear issue    | Proposed                      |
| `/sync-tasks`            | Linear ↔ TaskMaster sync | Proposed                      |

**Agents**:


| Agent              | Purpose                      | Status   |
| -------------------- | ------------------------------ | ---------- |
| `morning-planner`  | Morning briefing synthesis   | Exists   |
| `evening-reviewer` | Evening sync with Linear     | Exists   |
| `project-analyst`  | Deep project analysis        | Exists   |
| `project-creator`  | Scaffold new projects        | Exists   |
| `note-auditor`     | Semantic analysis, placement | Proposed |

**Skills**:


| Skill                | Purpose                               | Status   |
| ---------------------- | --------------------------------------- | ---------- |
| `obsidian-reader`    | Vault read operations (lightweight scan) | Exists   |
| `linear-integration` | Linear read/write with batch approval | Exists   |
| `graphiti-memory`    | Knowledge graph operations (primary historical source) | Exists   |
| `taskmaster-docs`    | Read .taskmaster/docs/ (PRD + captured notes) | Proposed |
| `github-integration` | Repo read/write, scaffolding          | Exists   |
| `git-operations`     | Local git state (status, log, diff)   | Proposed |
| `project-context`    | Resolve project identity across systems | Exists   |
| `semantic-analysis`  | Duplicate detection for note audit    | Proposed |
| `task-sync`          | TaskMaster ↔ Linear sync             | Proposed |

## Data Models

### Project Folder Structure

```
~/RS42/[project-name]/
├── .claude/settings.json
├── .taskmaster/
│   ├── docs/prd.md
├── CLAUDE.md
├── README.md
└── src/
```

### Linear Project Template

```markdown
# Project Overview

### Repository
**GitHub**: [repo-url]

### Goal
[Transform X into Y using Z]

### Target Users
- [User type]

### Platform & Distribution
- **Platform**: [tech stack]
- **Deployment**: [method]

## Success Criteria
- [ ] Criterion

## Development Roadmap
### Phase 1: [Name] [Status]
- [RS4-XX](link) Task description
```


### Linear Issue Template

```markdown

```


### Graphiti Group Structure

> **IMPORTANT**: Graphiti uses a FLAT namespace - hierarchical group_ids like `work:project-name` do NOT support hierarchical querying. Use only two graphs.

```python
# ✅ CORRECT - Two-Graph Model
group_id = "work"       # All professional/work context (Chief of Staff operations)
group_id = "personal"   # Personal notes, non-work context

# ❌ WRONG - Hierarchical patterns don't filter correctly
# group_id = "work:pe-eval"        # Querying ["work"] won't find this
# group_id = "work:chief-of-staff" # Creates isolated graph, not child of "work"
```

**Project Scoping Strategy**: Instead of separate graphs per project, include project names in:
1. **Query content** - `query="Pe-eval status decisions blockers"`
2. **Episode naming** - `name="Project Sync - Pe-eval - 2025-11-24"`
3. **Episode body** - Include project context in the stored content

This keeps all work knowledge in one searchable graph while maintaining project context through content.

## API Specifications

### Obsidian MCP

```yaml
search:
  tool: mcp__MCP_DOCKER__obsidian_simple_search
  params: {query: "[keywords]", context_length: 200}

get_file:
  tool: mcp__MCP_DOCKER__obsidian_get_file_contents
  params: {filepath: "path/to/note.md"}

get_recent:
  tool: mcp__MCP_DOCKER__obsidian_get_recent_changes
  params: {days: 7, limit: 5}  # Lightweight scan - Obsidian supplements Graphiti
```

### Linear MCP

```yaml
list_issues:
  tool: mcp__linear__list_issues
  params: {team: "RS42"}
  note: "Filter project/status locally"

create_issue:
  tool: mcp__linear__create_issue
  params: {team: "RS42", title: "[type]: [desc]", project: "[name]"}
  requires: "Batch approval"

update_issue:
  tool: mcp__linear__update_issue
  params: {id: "[id]", state: "Done"}
  requires: "Batch approval"
```

### Graphiti MCP

```yaml
search_facts:
  tool: mcp__graphiti__search_memory_facts
  params: {query: "[project] decisions", group_ids: ["work"], max_facts: 15}

add_memory:
  tool: mcp__graphiti__add_memory
  params: {name: "[name]", episode_body: "[content]", group_id: "work", source: "text"}
```

# Development Roadmap

## Phase 1: Foundation

### 1.1 Project Creation Command

- `/create-project [name]` scaffolds folder structure
- Links existing or creates new GitHub repo
- Creates Linear project from template
- Stores project creation event to Graphiti "work" graph (with project name in content)
- Optional TaskMaster init

### 1.2 Linear Templates

- Finalize project template format
- Define issue template with type prefixes (feat, fix, chore)
- Implement auto-detection of project from directory

### 1.3 Graphiti Configuration

- ✅ RESOLVED: Hierarchical group_ids don't work - use two-graph model instead
- All Chief of Staff operations use `group_ids=["work"]`
- Project scoping achieved through query content and episode naming, not separate graphs

## Phase 2: Two-View Task System

### 2.1 TaskMaster Integration

- TaskMaster handles developer-facing tasks
- Linear holds stakeholder milestones
- Parent task completion syncs to Linear

### 2.2 Sync Mechanism

- `/sync-tasks` for bidirectional sync
- Detect Linear scope changes
- Cascade updates to TaskMaster with approval

### 2.3 Sync Rules

- What syncs: parent completion, priority changes, scope changes
- What stays local: subtasks, implementation notes, test strategies

## Phase 3: Note Management

### 3.1 Note Capture

- `/capture-note` writes to project folder
- Notes stay local until audit

### 3.2 Note Audit

- Atomic check (single idea per note)
- Duplicate detection via semantic similarity
- Placement suggestions (folder, tags, links)

### 3.3 Vault Sync

- Batch approval before moving to Obsidian
- Apply frontmatter, tags, wikilinks
- Store facts to Graphiti
- Archive processed notes

## Phase 4: Refinement

### 4.1 Morning/Evening Flows

- Improve synthesis quality
- Better evidence gathering for proposals

### 4.2 Cross-Project Intelligence

- Query all `work:*` groups for patterns
- Surface blockers across projects

# Logical Dependency Chain

1. **Linear Templates** → needed for project creation
2. **Graphiti Groups** → test hierarchical querying first
3. **Project Creation** → depends on templates and groups
4. **TaskMaster ↔ Linear Sync** → depends on project structure
5. **Note Capture** → simple, can parallel with sync
6. **Note Audit** → depends on capture, needs semantic analysis
7. **Refinements** → after core flows work

Getting to visible value fast:

- Start with `/create-project` to unify setup
- Then `/sync-tasks` for daily workflow
- Then `/audit-notes` for knowledge management

# Risks and Mitigations

**Graphiti hierarchical groups may not work as expected**: Test first. Fallback: maintain explicit list of project group_ids.

**Linear project auto-detection unreliable**: Start with CLAUDE.md reference. Add fuzzy matching later.

**Semantic duplicate detection false positives**: Start conservative (high similarity threshold). Tune based on usage.

**Cascade updates too aggressive**: Always require approval. Store change history in Graphiti for undo context.

# Appendix

## MCP Server Configuration

Requires: Obsidian Local REST API, Linear MCP, Graphiti MCP, GitHub MCP (via Docker).

</PRD>
---

# Open Questions

1. **Graphiti hierarchical groups**: Does querying `group_ids=["work"]` return results from `work:pe-eval`, `work:chief-of-staff`, etc.? Or must we list each explicitly?
2. **Linear project auto-detection**: Map directory names to Linear projects via exact match, fuzzy match, or CLAUDE.md reference?
3. **Note atomic threshold**: Define "atomic" by word count (<500), heading count (1 main), or AI topic detection?
4. **Duplicate sensitivity**: What similarity threshold triggers merge/link suggestion: exact title, 80% content overlap, or shared key concepts?
5. **GitHub integration scope**: Create new repos automatically, or only link existing ones?
6. **Linear issue template sections**: Which are essential: Context, Acceptance Criteria, Technical Notes, Related?
