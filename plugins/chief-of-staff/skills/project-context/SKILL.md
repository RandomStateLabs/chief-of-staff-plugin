---
name: project-context
description: Resolve project identity across systems and generate consistent naming. Use when creating projects, matching directories to Linear projects, generating slugs, or determining the canonical project name from various inputs. Essential for cross-system operations.
---
# Project Context Skill

This skill handles project identity resolution, naming conventions, and cross-system mapping for the Chief of Staff plugin. It ensures consistent project identification across GitHub, Linear, Graphiti, and local directories.

## When to Use This Skill

Activate this skill when:

- Creating a new project (generate slug, validate naming)
- Matching a local directory to a Linear project
- Resolving project name from partial input
- Building cross-references between systems
- Determining canonical project identity

## Quick Patterns

### Pattern 1: Generate Project Slug

Convert display name to slug:

```python
def generate_slug(project_name: str) -> str:
    """
    "PE Evaluation" → "pe-evaluation"
    "Auth Service" → "auth-service"
    "CSHC SmartOps" → "cshc-smartops"
    """
    slug = project_name.lower()
    slug = slug.replace(" ", "-")
    slug = re.sub(r'[^a-z0-9-]', '', slug)
    slug = re.sub(r'-+', '-', slug)  # Collapse multiple hyphens
    return slug.strip('-')
```

### Pattern 2: Infer Project from Directory

```python
# Get current directory name
import os
dir_name = os.path.basename(os.getcwd())
# "pe-evaluation" or "pe_evaluation"

# Normalize for matching
normalized = dir_name.lower().replace("_", "-")
# "pe-evaluation"

# Search Linear for match
mcp__linear__list_projects()
# Filter locally for name containing normalized slug
```

### Pattern 3: Match Directory to Linear Project

```python
# Priority order for matching:
# 1. Exact slug match
# 2. Slug contained in project name (case-insensitive)
# 3. Check CLAUDE.md for explicit Linear URL
# 4. Fuzzy match on project name words

def match_project(dir_name: str, linear_projects: list) -> str:
    slug = dir_name.lower().replace("_", "-")

    for project in linear_projects:
        project_slug = generate_slug(project["name"])
        if slug == project_slug:
            return project  # Exact match

    for project in linear_projects:
        if slug in project["name"].lower():
            return project  # Partial match

    return None  # No match found
```

## Project Naming Standards

### RS42 Naming Conventions

| Component | Format | Example |
|-----------|--------|---------|
| Display Name | Title Case with spaces | "PE Evaluation" |
| Slug | lowercase-with-hyphens | "pe-evaluation" |
| Directory | Same as slug | `~/RS42/pe-evaluation/` |
| GitHub Repo | Same as slug | `github.com/.../pe-evaluation` |
| Linear Project | Display Name | "PE Evaluation" |
| Graphiti Query | Include display name in content | `query="PE Evaluation..."` |

### Naming Transformation Rules

```
Display Name    →  Slug              →  Uses
─────────────────────────────────────────────────
PE Evaluation   →  pe-evaluation     →  Directory, GitHub, URLs
Auth Service    →  auth-service      →  Directory, GitHub, URLs
CSHC SmartOps   →  cshc-smartops     →  Directory, GitHub, URLs
Mobile App v2   →  mobile-app-v2     →  Directory, GitHub, URLs
```

### Reserved/Special Cases

| Pattern | Handling |
|---------|----------|
| Version numbers | Preserve: "v2" → "v2" |
| Abbreviations | Lowercase: "API" → "api" |
| Client prefixes | Preserve: "CSHC" → "cshc" |
| Plugin suffix | Preserve: "-plugin" |

## Cross-System Identity Resolution

### Project Identity Structure

A fully resolved project has:

```yaml
project:
  display_name: "PE Evaluation"
  slug: "pe-evaluation"
  local_path: "~/RS42/pe-evaluation/"
  github_url: "https://github.com/yandifarinango/pe-evaluation"
  linear_project_id: "abc123"
  linear_url: "https://linear.app/rs42/project/pe-evaluation-xxx"
  graphiti_group: "work"  # Always "work" for Chief of Staff
  team: "RS42"
```

### Resolution Workflow

When given partial information, resolve the full identity:

#### From Directory Name

```python
# Input: current directory "pe-evaluation"
# 1. Generate display name candidate: "PE Evaluation" (title case)
# 2. Search Linear projects
projects = mcp__linear__list_projects()
# 3. Match by slug
matched = match_project("pe-evaluation", projects)
# 4. Check CLAUDE.md for explicit references
claude_md = read_file("CLAUDE.md")
# Extract Linear URL if present
# 5. Build full identity
```

#### From User Input

```python
# Input: "pe eval" (partial/informal)
# 1. Normalize: lowercase, expand abbreviations
candidates = ["pe evaluation", "pe-eval", "pe eval"]
# 2. Search Linear projects
projects = mcp__linear__list_projects()
# 3. Fuzzy match each candidate
for candidate in candidates:
    matched = fuzzy_match(candidate, projects)
    if matched:
        return matched
# 4. If ambiguous, present options to user
```

#### From Linear Project

```python
# Input: Linear project "PE Evaluation"
# 1. Generate slug: "pe-evaluation"
# 2. Derive paths
local_path = f"~/RS42/{slug}/"
github_url = f"https://github.com/yandifarinango/{slug}"
# 3. Verify existence (optional)
```

## CLAUDE.md Cross-Reference

### Standard CLAUDE.md Project Section

Every RS42 project should have this in CLAUDE.md:

```markdown
# [Display Name]

**Linear Project**: [linear-url]
**GitHub Repo**: [github-url]
**Graphiti**: Stored in `work` group

## Project Context
[Brief description]
```

### Parsing CLAUDE.md for Identity

```python
def extract_project_identity(claude_md_content: str) -> dict:
    """Extract project identity from CLAUDE.md content."""
    identity = {}

    # Extract Linear URL
    linear_match = re.search(r'\*\*Linear Project\*\*:\s*\[([^\]]+)\]\(([^)]+)\)', claude_md_content)
    if linear_match:
        identity['linear_url'] = linear_match.group(2)

    # Extract GitHub URL
    github_match = re.search(r'\*\*GitHub Repo\*\*:\s*\[([^\]]+)\]\(([^)]+)\)', claude_md_content)
    if github_match:
        identity['github_url'] = github_match.group(2)

    # Extract project name from title
    title_match = re.search(r'^# (.+)$', claude_md_content, re.MULTILINE)
    if title_match:
        identity['display_name'] = title_match.group(1)

    return identity
```

## Team Detection

### RS42 Teams

| Team | Use For |
|------|---------|
| RS42 | Internal projects, plugins, tools |
| Evonik | Client work for Evonik |

### Auto-Detection Logic

```python
def detect_team(project_name: str, local_path: str) -> str:
    """Determine which Linear team owns this project."""

    # Check path patterns
    if "evonik" in local_path.lower():
        return "Evonik"
    if "client" in local_path.lower():
        return "Evonik"  # Default client team

    # Check project name patterns
    if project_name.lower().startswith("cshc"):
        return "Evonik"

    # Default to RS42
    return "RS42"
```

## Validation

### Project Name Validation

```python
def validate_project_name(name: str) -> tuple[bool, str]:
    """Validate project name for creation."""

    if not name or len(name.strip()) == 0:
        return False, "Project name cannot be empty"

    if len(name) > 100:
        return False, "Project name too long (max 100 chars)"

    slug = generate_slug(name)
    if len(slug) < 2:
        return False, "Project slug too short after normalization"

    if slug.startswith("-") or slug.endswith("-"):
        return False, "Invalid slug format"

    return True, slug
```

### Duplicate Detection

Before creating a project, check all systems:

```python
def check_existing(slug: str, display_name: str) -> dict:
    """Check if project already exists in any system."""
    existing = {}

    # Check GitHub
    repos = mcp__MCP_DOCKER__search_repositories(query=f"{slug} user:yandifarinango")
    if repos["total_count"] > 0:
        existing["github"] = repos["items"][0]["html_url"]

    # Check Linear
    projects = mcp__linear__list_projects()
    for p in projects:
        if generate_slug(p["name"]) == slug:
            existing["linear"] = p
            break

    # Check local directory
    local_path = os.path.expanduser(f"~/RS42/{slug}")
    if os.path.exists(local_path):
        existing["local"] = local_path

    # Check Graphiti
    facts = mcp__graphiti__search_memory_facts(
        query=f"{display_name} project created",
        group_ids=["work"],
        max_facts=5
    )
    if facts:
        existing["graphiti"] = facts

    return existing
```

## Integration Points

### With /create-project Command

1. Receive project name from user
2. Validate name
3. Generate slug
4. Check for existing resources (all systems)
5. Return identity structure for creation

### With /project-brief Command

1. Receive partial project input
2. Infer from directory if not provided
3. Resolve to Linear project
4. Return full identity for data gathering

### With /project-sync Command

1. Auto-detect project from current directory
2. Resolve full identity
3. Use for cross-system sync

## Error Handling

### Ambiguous Project Input

```python
# If multiple matches found
matches = find_matching_projects(user_input)
if len(matches) > 1:
    # Present options to user
    print("Multiple projects match your input:")
    for i, match in enumerate(matches):
        print(f"  {i+1}. {match['name']} ({match['url']})")
    print("Please specify which project you meant.")
```

### No Project Found

```python
# If no matches in any system
if not matches:
    print(f"No project found matching '{user_input}'.")
    print("Would you like to create a new project with this name?")
    # Offer /create-project flow
```

### Invalid Directory Context

```python
# If not in a recognized project directory
cwd = os.getcwd()
if not cwd.startswith(os.path.expanduser("~/RS42/")):
    print("Not in an RS42 project directory.")
    print("Please specify a project name or navigate to a project folder.")
```

## Best Practices

1. **Always normalize** - Convert user input to consistent format before matching
2. **Prefer explicit** - Use CLAUDE.md references when available
3. **Fall back gracefully** - If one system fails, try others
4. **Ask when ambiguous** - Don't guess, confirm with user
5. **Cache identity** - Once resolved, use full identity throughout operation
6. **Update CLAUDE.md** - Ensure cross-references are maintained

## Common Mistakes

- Assuming directory name exactly matches Linear project name
- Not checking CLAUDE.md for explicit project links
- Creating duplicate projects without checking all systems
- Using case-sensitive matching (should be case-insensitive)
- Forgetting to include project name in Graphiti queries (flat namespace)
