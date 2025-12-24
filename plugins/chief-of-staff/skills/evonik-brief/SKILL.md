---
description: Generate comprehensive morning brief for Evonik employment work by synthesizing Azure DevOps tasks, meeting notes, project documentation, and personal tracking. Use when the user asks for Evonik brief, Evonik morning brief, Evonik status, or employment work overview.
allowed-tools:
  # Azure DevOps - PRIMARY source
  - mcp__azure-devops__wit_my_work_items
  - mcp__azure-devops__wit_get_work_item
  - mcp__azure-devops__wit_list_work_item_comments
  - mcp__azure-devops__wit_get_work_items_batch_by_ids
  # Granola Meetings - PRIMARY source
  - mcp__granola-mcp__get_folder_meetings
  - mcp__granola-mcp__search_meetings
  - mcp__granola-mcp__get_meeting
  - mcp__granola-mcp__get_meeting_notes
  - mcp__granola-mcp__get_transcript
  - mcp__granola-mcp__list_meetings
  # Obsidian - CONTEXT source
  - mcp__MCP_DOCKER__obsidian_simple_search
  - mcp__MCP_DOCKER__obsidian_get_file_contents
  - mcp__MCP_DOCKER__obsidian_get_recent_changes
  - mcp__obsidian-mcp-tools__search_vault_smart
  # Graphiti - CONTEXT source
  - mcp__graphiti__search_memory_facts
  - mcp__graphiti__search_nodes
  - mcp__graphiti__get_episodes
  # Linear - TRACKING source (personal reflection only)
  - mcp__linear__list_issues
  - mcp__linear__get_issue
---

# Evonik Morning Brief Skill

Generate a comprehensive morning briefing for Evonik employment work by orchestrating multiple data-gathering agents in parallel.

## Source of Truth Hierarchy (Employment Context)

```
PRIMARY:    Azure DevOps (CS Enterprise AI) + Granola Meetings
CONTEXT:    Obsidian Notes + Graphiti Memory
TRACKING:   Linear (personal reflection only, NOT source of truth)
```

> **Critical**: DigitalLabs project is RETIRED. Always use **CS Enterprise AI** as the Azure DevOps project parameter.

## Multi-Agent Architecture

This skill uses a **parallel agent orchestration** pattern:

1. **Orchestrator** spawns 4 specialized data-gathering agents simultaneously
2. Each agent focuses on a single data source for maximum efficiency
3. Results are collected and synthesized following the Source of Truth hierarchy
4. Gap analysis compares meeting commitments against tracked work

### Data Source Agents

| Agent | Source | Priority | Purpose |
|-------|--------|----------|---------|
| `evonik-azure-gatherer` | Azure DevOps | PRIMARY | Official task assignments |
| `evonik-granola-gatherer` | Granola | PRIMARY | Meeting decisions & action items |
| `evonik-context-gatherer` | Obsidian + Graphiti | CONTEXT | Documentation & memory |
| `evonik-linear-gatherer` | Linear | TRACKING | Personal reflection notes |

## Tool Reference

### Azure DevOps (PRIMARY)

```python
# Get my active work items
mcp__azure-devops__wit_my_work_items(
    project="CS Enterprise AI",  # NOT DigitalLabs
    type="assignedtome",
    includeCompleted=False,
    top=20
)

# Get work item details with relations
mcp__azure-devops__wit_get_work_item(
    id=[work_item_id],
    project="CS Enterprise AI",
    expand="relations"
)

# Get comments on work item
mcp__azure-devops__wit_list_work_item_comments(
    project="CS Enterprise AI",
    workItemId=[id],
    top=10
)
```

### Granola Meetings (PRIMARY)

> **Critical**: Use folder-based filtering, NOT query search. Folder ID: `59da2db9-0fc4-4688-a0eb-b28304ae7813`

```python
# PRIMARY: Get all meetings from Evonik folder
mcp__granola-mcp__get_folder_meetings(
    folder="Evonik",
    from_date="7d",
    limit=15
)

# Alternative: Search within folder (if additional filtering needed)
mcp__granola-mcp__search_meetings(
    folder="Evonik",  # Filter by folder, NOT query
    from_date="7d",
    limit=10
)

# Get meeting details
mcp__granola-mcp__get_meeting(meeting_id="[id]")

# Get meeting notes (structured summary)
mcp__granola-mcp__get_meeting_notes(meeting_id="[id]")

# Get transcript if needed
mcp__granola-mcp__get_transcript(meeting_id="[id]")
```

### Obsidian Notes (CONTEXT)

```python
# Semantic search for Evonik content
mcp__obsidian-mcp-tools__search_vault_smart(
    query="Evonik project status",
    filter={"limit": 10}
)

# Simple text search
mcp__MCP_DOCKER__obsidian_simple_search(
    query="Evonik",
    context_length=200
)

# Get specific file content
mcp__MCP_DOCKER__obsidian_get_file_contents(
    filepath="path/to/note.md"
)
```

### Graphiti Memory (CONTEXT)

```python
# Search for Evonik-related facts
mcp__graphiti__search_memory_facts(
    query="Evonik project status decisions blockers",
    group_ids=["work"],  # Always use "work" group
    max_facts=15
)

# Search for entities
mcp__graphiti__search_nodes(
    query="Evonik",
    group_ids=["work"],
    max_nodes=10
)

# Get recent work episodes
mcp__graphiti__get_episodes(
    group_ids=["work"],
    max_episodes=10
)
```

### Linear (TRACKING - Personal Only)

```python
# Get personal tracking issues
mcp__linear__list_issues(
    team="Evonik",
    assignee="me",
    status="in-progress"
)

# Get issue details
mcp__linear__get_issue(id="[issue-id]")
```

## Output Format

The final briefing follows this structure:

```markdown
# Evonik Morning Brief - [Date]

## Active Work Items (Azure DevOps)
| ID | Title | Status | Priority | Recent Activity |
|-----|-------|--------|----------|-----------------|
| [id] | [title] | [status] | [priority] | [last comment/update] |

### Blockers & Dependencies
- [blocker 1 with context]
- [dependency waiting on X]

## Recent Meeting Context (Granola)
### Key Decisions
- [decision 1] - [meeting date]
- [decision 2] - [meeting date]

### Action Items Assigned to Me
- [ ] [action item] - [due date if any]

### Upcoming Meetings
- [meeting] - [date/time]

## Project Documentation (Obsidian/Graphiti)
### Relevant Notes
- [[Note 1]] - [brief summary]
- [[Note 2]] - [brief summary]

### Historical Context
- [relevant fact from Graphiti]
- [past decision that affects current work]

## Personal Tracking (Linear)
| Issue | Status | Notes |
|-------|--------|-------|
| [issue] | [status] | [personal notes] |

## Gap Analysis
### Meeting Commitments Not in Azure DevOps
- [commitment from meeting not tracked as work item]

### Work Items Without Recent Context
- [work item with no recent meetings/notes about it]

## Recommended Focus Today
1. **[Priority 1]** - [why]
2. **[Priority 2]** - [why]
3. **[Priority 3]** - [why]

### Blockers to Address
- [blocker with suggested action]

### Meetings to Prepare For
- [meeting] - [preparation needed]
```

## Error Handling

| Scenario | Handling |
|----------|----------|
| Azure DevOps unavailable | Report error, cannot proceed (PRIMARY source) |
| Granola unavailable | Continue with Azure DevOps, note missing meeting context |
| Obsidian unavailable | Continue with other sources, note missing documentation |
| Graphiti unavailable | Continue with other sources, note missing memory context |
| Linear unavailable | Continue without personal tracking |
| No work items found | Report clean slate, suggest checking project assignment |
| No meetings found | Note absence, focus on work items |

## Best Practices

1. **Parallel Execution**: Always run context sources (Granola, Obsidian, Graphiti) in parallel
2. **Source Priority**: Azure DevOps defines work, everything else provides context
3. **Gap Analysis**: Always compare meeting commitments against tracked work
4. **Actionable Output**: End with specific, prioritized recommendations
5. **Date Awareness**: Use appropriate date ranges (7d for meetings, active for work items)
