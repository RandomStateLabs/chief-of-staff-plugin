---
name: obsidian-reader
description: Access and analyze Obsidian vault content including daily journals, recent notes, and searches. Use when you need to read from the user's Obsidian vault for briefings, project analysis, or context gathering.
---

# Obsidian Reader Skill

This skill helps you access and analyze content from the user's Obsidian vault using the Obsidian MCP server. Use this for gathering context from notes, journals, and documentation.

## When to Use This Skill

Activate this skill when:
- User asks for information from their Obsidian vault
- You need to access daily journals or periodic notes
- Commands like `/morning-brief` or `/project-brief` need note context
- User wants to search their notes
- You need to find recent note activity

## Core Workflow

### 1. Determine What to Access

Common patterns:
- **Today's journal**: Get the daily note for today
- **Recent activity**: Get notes modified in last N days
- **Project context**: Search for notes related to specific project
- **Specific note**: Read a particular note by path
- **Related notes**: Find notes connected to a topic

### 2. Choose the Right Tool

**Quick Reference** (most common operations):

| Need | Tool | Example |
|------|------|---------|
| Today's journal | `obsidian_get_periodic_note` | `period="daily"` |
| Recent changes | `obsidian_get_recent_changes` | `days=2, limit=10` |
| Search notes | `obsidian_simple_search` | `query="project name"` |
| Read note | `obsidian_get_file_contents` | `filepath="path/note.md"` |
| Read multiple | `obsidian_batch_get_file_contents` | `filepaths=[...]` |

### 3. Execute and Process

Use the MCP tools (examples below) and process results:
- Extract relevant information
- Summarize long notes
- Identify patterns or themes
- Cross-reference with other systems

## Quick Patterns

Common use cases you can handle without loading references:

### Pattern 1: Get Today's Journal

```
# Get today's daily note
mcp__MCP_DOCKER__obsidian_get_periodic_note(
    period="daily"
)

# Returns: The content of today's daily note, or error if doesn't exist
# Handle: If doesn't exist, note that journal hasn't been started today
```

### Pattern 2: Get Recent Activity

```
# Get notes modified in last 48 hours
mcp__MCP_DOCKER__obsidian_get_recent_changes(
    days=2,
    limit=10
)

# Returns: List of recently modified notes with paths and timestamps
# Process: Review titles and modification dates, read relevant ones
```

### Pattern 3: Search for Project Notes

```
# Search vault for project-related notes
mcp__MCP_DOCKER__obsidian_simple_search(
    query="Auth Service OR authentication",
    context_length=100
)

# Returns: Matching notes with context snippets
# Process: Identify most relevant notes, read full content if needed
```

### Pattern 4: Read Specific Note

```
# Read a specific note by path
mcp__MCP_DOCKER__obsidian_get_file_contents(
    filepath="1 - Main Notes/Auth Service Architecture.md"
)

# Returns: Full note content with line numbers
# Process: Extract key information, summarize if long
```

### Pattern 5: Read Multiple Notes

```
# Read several related notes at once
mcp__MCP_DOCKER__obsidian_batch_get_file_contents(
    filepaths=[
        "Projects/Auth Service.md",
        "1 - Main Notes/API Design.md",
        "Daily Notes/2025-10-20.md"
    ]
)

# Returns: Content of all notes concatenated
# Process: Analyze together for comprehensive context
```

### Pattern 6: Get Weekly/Monthly Notes

```
# Get weekly note
mcp__MCP_DOCKER__obsidian_get_periodic_note(
    period="weekly"
)

# Get monthly note
mcp__MCP_DOCKER__obsidian_get_periodic_note(
    period="monthly"
)

# Get recent weekly notes
mcp__MCP_DOCKER__obsidian_get_recent_periodic_notes(
    period="weekly",
    limit=4,
    include_content=true
)

# Returns: Periodic note content
# Use for: Weekly reviews, monthly planning, historical context
```

## MCP Tools Available

### Core Tools

**`obsidian_get_periodic_note`**
- Purpose: Get daily, weekly, monthly, quarterly, or yearly notes
- Parameters:
  - `period`: "daily" | "weekly" | "monthly" | "quarterly" | "yearly"
- Returns: Note content or error if doesn't exist
- Use for: Getting today's journal, weekly reviews

**`obsidian_get_recent_changes`**
- Purpose: Get notes modified recently
- Parameters:
  - `days`: Number of days to look back (default 90)
  - `limit`: Max number of notes to return (default 10, max 100)
- Returns: List of files with modification timestamps
- Use for: Finding recent activity, trending topics

**`obsidian_simple_search`**
- Purpose: Text search across all vault notes
- Parameters:
  - `query`: Search terms
  - `context_length`: Characters of context around match (default 100)
- Returns: Matching notes with context snippets
- Use for: Finding project notes, searching for topics

**`obsidian_get_file_contents`**
- Purpose: Read a specific note
- Parameters:
  - `filepath`: Path to note (relative to vault root)
- Returns: Full note content with line numbers
- Use for: Reading known notes, getting details

**`obsidian_batch_get_file_contents`**
- Purpose: Read multiple notes at once
- Parameters:
  - `filepaths`: Array of note paths
- Returns: Concatenated content of all notes
- Use for: Analyzing related notes together

**`obsidian_list_files_in_vault`**
- Purpose: List all files in vault root
- Parameters: None
- Returns: List of files and directories
- Use for: Exploring vault structure

**`obsidian_list_files_in_dir`**
- Purpose: List files in specific directory
- Parameters:
  - `dirpath`: Path to directory
- Returns: List of files in that directory
- Use for: Browsing specific folders

**`obsidian_complex_search`**
- Purpose: Advanced search with JsonLogic queries
- Parameters:
  - `query`: JsonLogic query object
- Returns: Matching documents
- Use for: Complex filtering (see references for examples)

## Common Workflows

### Morning Brief Workflow

1. Get today's journal:
   ```
   mcp__MCP_DOCKER__obsidian_get_periodic_note(period="daily")
   ```

2. Get recent changes:
   ```
   mcp__MCP_DOCKER__obsidian_get_recent_changes(days=2, limit=10)
   ```

3. Process: Summarize journal, identify key notes, extract insights

### Project Brief Workflow

1. Search for project notes:
   ```
   mcp__MCP_DOCKER__obsidian_simple_search(
       query="[project name] OR [keywords]",
       context_length=150
   )
   ```

2. Read relevant notes:
   ```
   mcp__MCP_DOCKER__obsidian_batch_get_file_contents(
       filepaths=[list from search results]
   )
   ```

3. Process: Map to Linear issues, extract decisions, identify status

### Evening Sync Workflow

1. Get today's journal:
   ```
   mcp__MCP_DOCKER__obsidian_get_periodic_note(period="daily")
   ```

2. Get notes modified today:
   ```
   mcp__MCP_DOCKER__obsidian_get_recent_changes(days=1, limit=20)
   ```

3. Process: Match to Linear tasks, identify completed work

## Best Practices

1. **Start Broad**: Use search or recent changes first, then read specifics
2. **Limit Results**: Don't request 100 notes when 10 will do
3. **Context Length**: 100-150 chars usually sufficient for search
4. **Batch Reads**: Use batch tool for multiple notes (more efficient)
5. **Handle Missing**: Gracefully handle missing journals or notes
6. **Summarize Long Notes**: Don't output full content of long notes

## Error Handling

- **Note doesn't exist**: Normal - user may not have created journal yet
- **Path not found**: Double-check path, ask user for correct location
- **Search returns nothing**: Broaden query or try different keywords
- **MCP not connected**: Inform user Obsidian MCP needs configuration

## Advanced Features

For complex requirements, see:
- `references/search-patterns.md` - Advanced search techniques, JsonLogic queries
- `references/folder-structure.md` - Common vault organization patterns
- `references/frontmatter-parsing.md` - Extracting metadata from notes

## Integration with Other Skills

This skill commonly works with:
- **linear-integration** skill - match notes to Linear tasks
- **graphiti-memory** skill - store insights from notes for future context
- Commands that need note context (`/morning-brief`, `/project-brief`, etc.)

## Token Optimization

- Use `context_length` to control search result size
- Batch read multiple notes instead of separate calls
- Request specific notes when path known (faster than search)
- Use `limit` parameter to control result count
