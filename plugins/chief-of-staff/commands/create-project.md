---
description: Scaffold a new project across all systems - creates GitHub repo, Linear project, local directory structure, and Graphiti memory entry
argument-hint: project name
---

# Create Project Command

You are now in **Project Creation Mode**. Help the user scaffold a new project across all RS42 systems: GitHub, Linear, local directory, and Graphiti.

## Command Purpose

This command creates a fully-linked project environment:
1. **GitHub Repository** - Code hosting with README
2. **Linear Project** - Task tracking with RS42 template
3. **Local Directory** - Standard folder structure with CLAUDE.md
4. **Graphiti Memory** - Creation event stored in `work` graph
5. **Optional**: TaskMaster initialization for PRD-based task generation

## Command Usage

```
/create-project [project-name]              # Create with specified name
/create-project "PE Evaluation"             # Example with spaces
/create-project auth-service                # Example with slug format
```

## Workflow

### Step 1: Parse Project Name

Extract the project name from the command argument:
- If quoted: Use exact string ("PE Evaluation")
- If unquoted: Convert slug to display name (auth-service → "Auth Service")

Validate the name:
- Not empty
- Reasonable length (< 100 characters)
- Can be converted to valid slug

### Step 2: Spawn Project Creator Agent

**Always spawn the `project-creator` agent** for this operation. Project creation is a complex, multi-system operation that benefits from the agent's dedicated context.

```
Use the Task tool:
- subagent_type: "chief-of-staff:project-creator"
- prompt: "Create project: [project name]. Check for existing resources, present scaffolding plan for approval, then create approved resources."
```

### Step 3: Agent Handles Creation

The project-creator agent will:

1. **Check Existing Resources**
   - Search GitHub for existing repo
   - Search Linear for existing project
   - Check local filesystem for existing directory
   - Query Graphiti for existing project entries

2. **Present Scaffolding Plan** (Batch Approval Pattern)
   - Show what will be created
   - Mark existing resources
   - Offer options (create new, link existing, skip)
   - Wait for user confirmation

3. **Execute Creation** (after approval)
   - Create GitHub repository
   - Create Linear project with RS42 template
   - Create local directory structure
   - Initialize Git with remote
   - Store creation event to Graphiti
   - Optionally initialize TaskMaster

4. **Report Results**
   - Show what was created with links
   - Provide next steps
   - Suggest verification via `/project-brief`

## Batch Approval Pattern

**Critical**: All external writes require explicit user approval.

The agent will present a plan like:

```markdown
## Project Creation Plan

**Project**: PE Evaluation
**Slug**: pe-evaluation

### Resources to Create

- [ ] **GitHub Repository** (NEW)
      Name: pe-evaluation
      Visibility: Private

- [ ] **Linear Project** (NEW)
      Name: PE Evaluation
      Team: RS42

- [ ] **Local Directory**
      Path: ~/RS42/pe-evaluation/

- [ ] **Graphiti Memory**
      Group: work

### Optional
- [ ] Initialize TaskMaster?
- [ ] Initialize Git?

**Proceed with creation? (yes/no)**
```

User must explicitly approve before any resources are created.

## Skills Used

This command leverages these skills:
- **project-context** - Name validation, slug generation, identity resolution
- **github-integration** - Repository creation and checking
- **linear-project-templates** - Project template application
- **graphiti-memory** - Memory storage patterns

## MCP Tools Used

### Read Operations (Auto-Allowed)
```python
mcp__MCP_DOCKER__search_repositories(query="...")  # Check GitHub
mcp__linear__list_projects()                        # Check Linear
mcp__graphiti__search_nodes(query, group_ids=["work"])  # Check Graphiti
Bash(ls ~/RS42/[slug])                              # Check local
```

### Write Operations (Require Approval)
```python
mcp__MCP_DOCKER__create_repository(...)    # GitHub (EXPLICIT approval)
mcp__linear__create_project(...)           # Linear (EXPLICIT approval)
Bash(mkdir -p ~/RS42/[slug]/...)           # Local (auto, low risk)
Write(file_path, content)                   # Local files (auto, low risk)
Bash(git init)                              # Git init (auto, low risk)
mcp__graphiti__add_memory(...)             # Graphiti (auto, low risk)
```

## Output Format

After successful creation:

```markdown
## Project Created: [Name]

### Resources Created

✅ **GitHub Repository**
   [github-url]

✅ **Linear Project**
   [linear-url]

✅ **Local Directory**
   ~/RS42/[slug]/
   ├── .claude/settings.json
   ├── .taskmaster/docs/
   ├── CLAUDE.md
   ├── README.md
   └── src/

✅ **Graphiti Memory**
   Stored in `work` group

✅ **Git Initialized**
   Remote: origin → [github-url]

### Next Steps

1. cd ~/RS42/[slug]
2. Update CLAUDE.md with project goals
3. git add . && git commit -m "Initial scaffold" && git push
4. /project-brief "[Name]" to verify
```

## Error Handling

### Invalid Project Name
```
Cannot create project: Invalid name "[input]"
- Name cannot be empty
- Name must contain at least 2 characters after normalization

Please provide a valid project name.
```

### Resource Already Exists
The agent will detect and offer options:
```
Found existing resources:
- GitHub: pe-evaluation exists at [url]

Options:
1. Link existing GitHub repo (don't create new)
2. Use different name
3. Cancel

Which would you like?
```

### Partial Failure
If some resources fail:
```
⚠️ Partial creation completed:

✅ GitHub: Created
❌ Linear: Failed - [error]
✅ Local: Created
✅ Graphiti: Stored

Would you like to:
1. Retry Linear creation
2. Create Linear project manually
3. Continue without Linear project
```

## Advanced Options

### Link Existing Repository
```
/create-project "My Project" --link-repo https://github.com/.../existing-repo
```

### Specify Team
```
/create-project "Client Project" --team Evonik
```

### Skip TaskMaster
```
/create-project "Quick Project" --no-taskmaster
```

### Include Initial Goal
```
/create-project "PE Evaluation" --goal "AI-powered document analysis for due diligence"
```

## Related Commands

- `/project-brief [name]` - Verify project setup after creation
- `/project-sync` - Sync state across systems
- `/evening-sync` - End-of-day review with Linear updates

## After Creation

Suggest to user:
```
Project scaffolded! Would you like me to:
- Open the project directory?
- Create an initial PRD for TaskMaster?
- Run /project-brief to verify the setup?
```
