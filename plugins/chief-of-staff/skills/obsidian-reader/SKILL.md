---
name: obsidian-reader
description: Access and analyze Obsidian vault content including daily journals, recent notes, and searches. Use when you need to read from the user's Obsidian vault for briefings, project analysis, or context gathering.
---

# Obsidian Reader Skill

This skill helps you access and analyze content from the user's Obsidian vault using the Obsidian MCP server. Use this for gathering context from notes, journals, and documentation.

## Vault Structure

The user's vault is organized as follows:

```
Vault/
├── 1 - Main Notes/              # Technical docs, project notes, troubleshooting
├── 2 - Tags/                    # Tag organization
├── 3 - Template/                # Templates (Daily Prep, Evening Reflection, etc.)
├── Personal/
│   ├── Journal/
│   │   ├── Morning Entries/     # Daily journals: YYYY-MM-DD.md
│   │   └── MMYY/                # Monthly archives (0325, 0425, etc.)
│   ├── Network/                 # Personal contacts
│   └── Thoughts & Ideas/        # Ideas and thoughts
├── Excalidraw/                  # Diagrams
└── copilot/                     # AI copilot content
```

### Key Locations

| Content Type | Path |
|--------------|------|
| Today's Journal | `Personal/Journal/Morning Entries/YYYY-MM-DD.md` |
| General Notes | `1 - Main Notes/` |
| Ideas | `Personal/Thoughts & Ideas/` |
| Templates | `3 - Template/` |

## When to Use This Skill

Activate this skill when:
- User asks for information from their Obsidian vault
- You need to access daily journals
- Commands like `/morning-brief` or `/project-brief` need note context
- User wants to search their notes
- You need to find recent note activity

## Quick Patterns

### Pattern 1: Get Today's Journal

**IMPORTANT**: This vault does NOT use the Periodic Notes plugin. Journals are stored as regular files.

```python
# Build today's date path
# Format: Personal/Journal/Morning Entries/YYYY-MM-DD.md

# Get today's journal (replace with actual date)
mcp__MCP_DOCKER__obsidian_get_file_contents(
    filepath="Personal/Journal/Morning Entries/2025-11-24.md"
)

# Returns: Full journal content
# Handle: If file doesn't exist, journal hasn't been created today
```

**Journal Structure** (what you'll find):
- **Atomic Habits**: Motivational quote and mindset section
- **Today's Focus**: Freeform journaling about the day
- **Gratitude**: 3 gratitude items
- **Top 3 Priorities**: Split into Evonik Work, RS42 Work, Personal
- **Evening Wind Down**: End of day habits

### Pattern 2: Get Recent Activity

```python
# Get notes modified in last 48 hours
mcp__MCP_DOCKER__obsidian_get_recent_changes(
    days=2,
    limit=15
)

# Returns: List of recently modified notes with paths and timestamps
# Process: Review titles, identify work-related notes, read relevant ones
```

### Pattern 3: Search for Project Notes

```python
# Search vault for project-related notes
mcp__MCP_DOCKER__obsidian_simple_search(
    query="CSHC OR SmartOps OR Pharmatec",
    context_length=150
)

# Returns: Matching notes with context snippets
# Process: Identify most relevant notes, read full content if needed
```

**Known Project Keywords**:

| Category | Keywords |
|----------|----------|
| Evonik Work | CSHC, HP SmartOps, Pharmatec, Logic App, Azure, Reindexing |
| RS42 Work | Claude Code, Linear, Graphiti, MCP, plugin, sourcing, Pe-eval |

### Pattern 4: Read Specific Note

```python
# Read a specific note by path
mcp__MCP_DOCKER__obsidian_get_file_contents(
    filepath="1 - Main Notes/CSHC SmartOps Complete Troubleshooting Journey.md"
)

# Returns: Full note content
# Process: Extract key information, summarize if long
```

### Pattern 5: Read Multiple Notes

```python
# Read several related notes at once
mcp__MCP_DOCKER__obsidian_batch_get_file_contents(
    filepaths=[
        "1 - Main Notes/CSHC SmartOps Complete Troubleshooting Journey.md",
        "1 - Main Notes/HP-SmartOps PROD Deployment - Subnet Capacity Issue.md",
        "Personal/Journal/Morning Entries/2025-11-17.md"
    ]
)

# Returns: Content of all notes concatenated
# Process: Analyze together for comprehensive context
```

### Pattern 6: List Recent Journal Entries

```python
# List available journal entries
mcp__MCP_DOCKER__obsidian_list_files_in_dir(
    dirpath="Personal/Journal/Morning Entries"
)

# Returns: List of journal files (YYYY-MM-DD.md format)
# Use to find which days have journal entries
```

### Pattern 7: Search Main Notes

```python
# Search only in technical notes folder
mcp__MCP_DOCKER__obsidian_simple_search(
    query="Claude Code plugin architecture",
    context_length=200
)

# Returns: Matching notes with context
# Most technical work notes are in "1 - Main Notes/"
```

## MCP Tools Available

### Core Tools

**`obsidian_get_file_contents`**
- Purpose: Read a specific note by path
- Parameters:
  - `filepath`: Path to note (relative to vault root)
- Returns: Full note content
- Use for: Reading journals, specific notes, known files

**`obsidian_get_recent_changes`**
- Purpose: Get notes modified recently
- Parameters:
  - `days`: Number of days to look back (default 90)
  - `limit`: Max number of notes to return (default 10, max 100)
- Returns: List of files with modification timestamps
- Use for: Finding recent activity, what's being worked on

**`obsidian_simple_search`**
- Purpose: Text search across all vault notes
- Parameters:
  - `query`: Search terms (supports OR for alternatives)
  - `context_length`: Characters of context around match (default 100)
- Returns: Matching notes with context snippets
- Use for: Finding project notes, searching for topics

**`obsidian_batch_get_file_contents`**
- Purpose: Read multiple notes at once
- Parameters:
  - `filepaths`: Array of note paths
- Returns: Concatenated content of all notes
- Use for: Analyzing related notes together

**`obsidian_list_files_in_dir`**
- Purpose: List files in specific directory
- Parameters:
  - `dirpath`: Path to directory
- Returns: List of files in that directory
- Use for: Browsing folders, finding available journals

**`obsidian_list_files_in_vault`**
- Purpose: List all files in vault root
- Parameters: None
- Returns: List of top-level files and directories
- Use for: Exploring vault structure

**`obsidian_complex_search`**
- Purpose: Advanced search with JsonLogic queries for structured filtering
- Parameters:
  - `query`: JsonLogic query object
- Returns: Matching documents
- Use for: Complex filtering by path patterns, frontmatter, tags

**JsonLogic Operators Available**:
- `glob`: Pattern matching - **case-sensitive** (e.g., `{"glob": ["*.md", {"var": "path"}]}`)
- `regexp`: Regex matching - **no (?i) flag** (e.g., `{"regexp": ["[Pp]e-eval", {"var": "basename"}]}`)
- `and`, `or`, `not`: Logical operators
- `in`: Check array membership (e.g., `{"in": ["tag-name", {"var": "tags"}]}`)
- `var`: Access properties: `path`, `basename`, `tags` (other frontmatter fields don't work)

**What complex_search CAN search**:
| Variable | Works | Example |
|----------|-------|---------|
| `path` | Yes | Full file path |
| `basename` | Yes | Filename only |
| `tags` | Yes | Frontmatter tags array (via `in` operator) |
| Other frontmatter (`status`, `type`, `date`) | No | Returns empty |
| File content | No | Use `simple_search` instead |

**Complex Search Examples**:

```python
# Find all notes in Main Notes folder
mcp__MCP_DOCKER__obsidian_complex_search(
    query={"glob": ["1 - Main Notes/**/*.md", {"var": "path"}]}
)

# Find notes with project name in path (case-insensitive via character class)
mcp__MCP_DOCKER__obsidian_complex_search(
    query={"regexp": ["[Pp]e-eval", {"var": "path"}]}
)

# Combine: Main Notes folder AND project-related filename
mcp__MCP_DOCKER__obsidian_complex_search(
    query={
        "and": [
            {"glob": ["1 - Main Notes/**/*.md", {"var": "path"}]},
            {"glob": ["*SmartOps*", {"var": "basename"}]}
        ]
    }
)

# Find by frontmatter tags (ONLY tags work, not other frontmatter fields)
mcp__MCP_DOCKER__obsidian_complex_search(
    query={"in": ["networking", {"var": "tags"}]}
)

# OR query: Find CSHC or Pharmatec notes
mcp__MCP_DOCKER__obsidian_complex_search(
    query={
        "or": [
            {"glob": ["*CSHC*", {"var": "path"}]},
            {"glob": ["*Pharmatec*", {"var": "path"}]}
        ]
    }
)
```

**IMPORTANT Limitations**:
- `glob` is **case-sensitive**: `*smartops*` won't match `SmartOps`
- `regexp` does **NOT support (?i) flag** - use character classes like `[Ss]martOps`
- Only `tags` frontmatter field works - `status`, `type`, `date` return empty
- Returns filename only, no content/context (use `simple_search` for that)

**Note**: This is NOT semantic/AI search - it's structured path/metadata filtering. Use `simple_search` for text content matching.

## Common Workflows

### Morning Brief Workflow

1. **Get today's journal** (build date dynamically):
   ```python
   mcp__MCP_DOCKER__obsidian_get_file_contents(
       filepath="Personal/Journal/Morning Entries/YYYY-MM-DD.md"
   )
   ```
   - Extract "Today's Focus" section for context
   - Extract "Top 3 Priorities" for planned work

2. **Get recent changes** (last 48 hours):
   ```python
   mcp__MCP_DOCKER__obsidian_get_recent_changes(days=2, limit=15)
   ```
   - Filter to notes in "1 - Main Notes/" for work context
   - Identify project-related notes

3. **Process**:
   - Summarize journal focus
   - Cross-reference recent notes with Linear tasks
   - Extract any blockers or decisions mentioned

### Project Brief Workflow

1. **Search for project notes**:
   ```python
   mcp__MCP_DOCKER__obsidian_simple_search(
       query="[project name] OR [project keywords]",
       context_length=150
   )
   ```

2. **Read relevant notes**:
   ```python
   mcp__MCP_DOCKER__obsidian_batch_get_file_contents(
       filepaths=[...paths from search results...]
   )
   ```

3. **Process**:
   - Extract key decisions and blockers
   - Map to Linear issues
   - Identify current status and next steps

### Evening Sync Workflow

1. **Get today's journal** (for priorities set in morning):
   ```python
   mcp__MCP_DOCKER__obsidian_get_file_contents(
       filepath="Personal/Journal/Morning Entries/YYYY-MM-DD.md"
   )
   ```

2. **Get notes modified today**:
   ```python
   mcp__MCP_DOCKER__obsidian_get_recent_changes(days=1, limit=20)
   ```

3. **Process**:
   - Compare morning priorities with notes created/modified
   - Identify completed work to update Linear
   - Extract learnings for Graphiti storage

## Parsing Journal Content

When reading a journal, look for these sections:

```markdown
# Top 3 Priorities
###### Evonik Work
- [ ] Task 1
- [x] Task 2 (completed)
###### RS42 Work
- [ ] Task 1
###### Personal
- [ ] Task 1
```

**Parsing tips**:
- `- [x]` = completed task
- `- [ ]` = incomplete task
- Section under `###### Evonik Work` = client/consulting work
- Section under `###### RS42 Work` = company/product work
- Look for `# Today's Focus` for freeform context

## Best Practices

1. **Build journal paths dynamically**: Use current date in format `YYYY-MM-DD`
2. **Start with recent changes**: Get overview before diving into specific notes
3. **Use OR in searches**: `"CSHC OR SmartOps"` for broader results
4. **Batch reads**: More efficient than multiple single reads
5. **Handle missing journals**: Not every day has an entry - that's normal
6. **Focus on "1 - Main Notes/"**: Most technical content lives here

## Error Handling

- **File doesn't exist**: Journal may not be created yet today - note this and continue
- **Path not found**: Double-check path format (case-sensitive)
- **Search returns nothing**: Try broader keywords or check spelling
- **MCP not connected**: Inform user to check Docker/MCP configuration

## Integration with Other Skills

This skill commonly works with:
- **linear-integration** skill - match notes to Linear tasks
- **graphiti-memory** skill - store insights from notes for future context
- Commands: `/morning-brief`, `/project-brief`, `/evening-sync`

## Token Optimization

- Use `context_length` parameter to control search result size
- Batch read multiple notes instead of separate calls
- Request specific notes when path is known (faster than search)
- Use `limit` parameter appropriately (10-15 usually sufficient)
- Filter recent changes mentally - not all modified files are relevant
