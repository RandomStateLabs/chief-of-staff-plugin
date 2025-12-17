---
name: github-integration
description: Create and manage GitHub repositories for RS42 projects. Use when scaffolding new projects, checking if repos exist, or linking existing repositories to Linear projects. Covers repo creation, existence checks, and cross-system linking patterns.
---
# GitHub Integration Skill

This skill provides patterns for GitHub repository operations within the Chief of Staff plugin, primarily for project creation and cross-system linking.

## When to Use This Skill

Activate this skill when:

- Creating a new project that needs a GitHub repository
- Checking if a repository already exists before creation
- Linking an existing repository to a Linear project
- Getting repository information for project briefs

## Quick Patterns

### Pattern 1: Check if Repository Exists

Before creating a repo, always check if it exists:

```python
# Search for existing repo by name
mcp__MCP_DOCKER__search_repositories(
    query="[repo-name] user:yandifarinango"
)

# Or check specific repo directly
mcp__MCP_DOCKER__get_file_contents(
    owner="yandifarinango",
    repo="[repo-name]",
    path="README.md"
)
# If returns content → repo exists
# If error → repo doesn't exist
```

### Pattern 2: Create New Repository

```python
# Create private repo with README
mcp__MCP_DOCKER__create_repository(
    name="[repo-name]",
    description="[Project description]",
    private=True,
    auto_init=True  # Creates with README
)
# Returns: { full_name, html_url, clone_url, ... }
```

### Pattern 3: Get Repository URL

```python
# Format: https://github.com/yandifarinango/[repo-name]
repo_url = f"https://github.com/yandifarinango/{repo_name}"

# Or construct from search results
result = mcp__MCP_DOCKER__search_repositories(query="[repo-name]")
repo_url = result["items"][0]["html_url"]
```

## Detailed Operations

### Repository Creation Workflow

For `/create-project`, follow this sequence:

#### Step 1: Generate Repository Name

Convert project name to slug:
- Lowercase
- Replace spaces with hyphens
- Remove special characters
- Example: "PE Evaluation" → "pe-evaluation"

#### Step 2: Check for Existing Repository

```python
# Search user's repos
results = mcp__MCP_DOCKER__search_repositories(
    query="[slug] user:yandifarinango"
)

if results["total_count"] > 0:
    # Repo exists - offer to link instead
    existing_url = results["items"][0]["html_url"]
    # Ask user: "Repository already exists at {existing_url}. Link this one?"
else:
    # Safe to create new repo
    pass
```

#### Step 3: Create Repository

```python
# Standard RS42 project repository
new_repo = mcp__MCP_DOCKER__create_repository(
    name="[slug]",
    description="[Project goal - 1 sentence]",
    private=True,
    auto_init=True
)

repo_url = new_repo["html_url"]
clone_url = new_repo["clone_url"]
```

#### Step 4: Add Initial Files (Optional)

After creation, add project files:

```python
# Add CLAUDE.md
mcp__MCP_DOCKER__create_or_update_file(
    owner="yandifarinango",
    repo="[slug]",
    path="CLAUDE.md",
    content="[CLAUDE.md template content]",
    message="Add CLAUDE.md with project context",
    branch="main"
)
```

### Repository Information Retrieval

For project briefs and sync operations:

```python
# Get repo details
mcp__MCP_DOCKER__get_file_contents(
    owner="yandifarinango",
    repo="[repo-name]",
    path=""  # Root directory listing
)

# Get specific file
mcp__MCP_DOCKER__get_file_contents(
    owner="yandifarinango",
    repo="[repo-name]",
    path="README.md"
)

# Get recent commits
mcp__MCP_DOCKER__list_commits(
    owner="yandifarinango",
    repo="[repo-name]",
    perPage=10
)
```

## RS42 Repository Standards

### Naming Conventions

| Project Type | Repo Name Pattern | Example |
|--------------|-------------------|---------|
| Standard project | `[project-slug]` | `pe-evaluation` |
| Claude Code plugin | `[name]-plugin` | `chief-of-staff-plugin` |
| Client project | `[client]-[project]` | `cshc-smartops` |
| Internal tool | `[tool-name]` | `sourcing-agent` |

### Default Repository Structure

New RS42 repositories should have:

```
[repo-name]/
├── .claude/
│   └── settings.json
├── .taskmaster/
│   └── docs/
│       └── prd.md (if TaskMaster initialized)
├── CLAUDE.md
├── README.md
└── src/
```

### Repository Description Format

Keep descriptions concise and goal-oriented:

```
[Action verb] [what] for [who/purpose]
```

Examples:
- "AI-powered document analysis for PE due diligence"
- "Claude Code plugin for unified project management"
- "Invoice processing automation for CSHC operations"

## Error Handling

### Repository Already Exists

```python
# Check first, then handle gracefully
results = mcp__MCP_DOCKER__search_repositories(query="[name] user:yandifarinango")

if results["total_count"] > 0:
    existing = results["items"][0]
    # Present to user:
    # "Repository '{name}' already exists at {html_url}"
    # "Would you like to link this existing repository instead of creating new?"
```

### Repository Creation Failed

Common failures:
- **Name already taken**: Check exists first
- **Invalid name**: Ensure slug format (no spaces, special chars)
- **Permission denied**: Check GitHub token permissions

```python
try:
    result = mcp__MCP_DOCKER__create_repository(...)
except Exception as e:
    if "name already exists" in str(e):
        # Offer to link existing
    elif "validation failed" in str(e):
        # Fix naming
    else:
        # Report error, continue with other resources
```

### Repository Not Found

```python
try:
    mcp__MCP_DOCKER__get_file_contents(owner="yandifarinango", repo="[name]", path="")
except Exception:
    # Repo doesn't exist
    # For create-project: safe to create
    # For project-brief: note repo not found, continue with other sources
```

## Integration with Other Skills

### With linear-project-templates

When creating a project:
1. Create GitHub repo (this skill)
2. Create Linear project with repo URL in description (linear-project-templates)
3. Store cross-reference in Graphiti (graphiti-memory)

### With project-context

Use project-context skill to:
- Generate consistent slug from project name
- Resolve project identity across systems
- Map local directories to GitHub repos

## GitHub MCP Tool Reference

### Available Tools

| Tool | Purpose |
|------|---------|
| `mcp__MCP_DOCKER__search_repositories` | Find repos by name/query |
| `mcp__MCP_DOCKER__get_file_contents` | Read files/directories |
| `mcp__MCP_DOCKER__create_repository` | Create new repository |
| `mcp__MCP_DOCKER__create_or_update_file` | Add/update files |
| `mcp__MCP_DOCKER__list_commits` | Get commit history |
| `mcp__MCP_DOCKER__create_branch` | Create new branch |
| `mcp__MCP_DOCKER__fork_repository` | Fork existing repo |

### Tool Parameter Details

```python
# search_repositories
mcp__MCP_DOCKER__search_repositories(
    query: str,      # Search query (supports user:, org:, etc.)
    page: int = 1,
    perPage: int = 30
)

# create_repository
mcp__MCP_DOCKER__create_repository(
    name: str,           # Repository name (slug format)
    description: str,    # Short description
    private: bool,       # True for RS42 projects
    auto_init: bool      # True to create with README
)

# get_file_contents
mcp__MCP_DOCKER__get_file_contents(
    owner: str,      # "yandifarinango"
    repo: str,       # Repository name
    path: str,       # File path (empty for root listing)
    branch: str      # Optional branch name
)

# create_or_update_file
mcp__MCP_DOCKER__create_or_update_file(
    owner: str,
    repo: str,
    path: str,       # File path to create/update
    content: str,    # File content
    message: str,    # Commit message
    branch: str,     # Branch name
    sha: str         # Required for updates (file's current SHA)
)
```

## Best Practices

1. **Always check before creating** - Avoid duplicate repos
2. **Use private repos** - RS42 projects default to private
3. **Include CLAUDE.md** - Essential for AI assistance context
4. **Link to Linear** - Always add GitHub URL to Linear project description
5. **Consistent naming** - Follow slug conventions for cross-system matching
6. **Auto-init repos** - Create with README for immediate usability

## Common Mistakes

- Creating repo without checking if it exists first
- Using spaces or special characters in repo names
- Creating public repos for private projects
- Forgetting to link repo URL in Linear project
- Not adding CLAUDE.md for AI context
