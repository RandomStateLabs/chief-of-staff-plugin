---
description: Get a comprehensive deep dive into a specific project by synthesizing all context from Obsidian notes, Linear issues, and Graphiti memory
argument-hint: project name
---

# Project Brief Command

You are now in **Project Brief Mode**. Help the user get comprehensive context on a specific project by synthesizing information across all systems.

## Command Purpose

Generate a detailed project briefing that:
1. Gathers all related context from Obsidian, Linear, and Graphiti
2. Maps documentation to tasks and tracks relationships
3. Analyzes project health, velocity, and blockers
4. Provides actionable insights and recommendations

## Command Usage

```
/project-brief [project-name]           # Deep dive into specific project
/project-brief Auth Service             # Example: Auth Service project
/project-brief "Mobile App Redesign"    # Example with spaces
```

## Workflow

### Step 1: Project Identification

- Infer the project name automatically from the current working directory or project path (e.g., use the last directory segment as the project slug).
- Search Linear projects for the inferred name:
  - Try exact match first (e.g., "auth-service" → "Auth Service")
  - If no exact match, perform a partial/case-insensitive match against Linear project names.
- If multiple candidates found or no clear match, ask the user to select which project they want a briefing on from a list of recent Linear projects.
- Use the selected or inferred project name for all further context gathering and analysis.

### Step 2: Activate Relevant Skills

This command should leverage:
- **obsidian-reader** skill - to find all project-related notes
- **linear-integration** skill - to get all project issues and details
- **graphiti-memory** skill - to retrieve project history and decisions

### Step 3: Spawn Project Analyst Agent

**Spawn the `project-analyst` agent** for project briefs. This is a context-heavy operation that benefits from the agent's dedicated 200K context window.

```
Use the Task tool:
- subagent_type: "chief-of-staff:project-analyst"
- prompt: Include the project name and any parameters (--quick, --timeline, --health, --blockers)
```

### Step 4: Comprehensive Data Gathering

Use skills to gather:

**From Obsidian**:
1. Search for notes containing project name
2. Find related notes by keywords/tags
3. Get recent notes mentioning project
4. Read full content of most relevant notes

**From Linear**:
1. Get project details (if formal Linear project)
2. Get all issues labeled with project tag
3. Get issues across all states (Done, In Progress, Blocked, etc.)
4. Get project team members and metadata

**From Graphiti**:
1. Search for project-related facts
2. Find project entities and relationships
3. Get historical episodes about project
4. Query for decision history

### Step 5: Cross-System Analysis

Perform synthesis:
1. **Map relationships**: Connect Linear issues to Obsidian docs
2. **Timeline reconstruction**: Build project history from Graphiti
3. **Pattern detection**: Identify velocity, blockers, risks
4. **Health assessment**: Evaluate project across dimensions

### Step 6: Generate Comprehensive Project Brief

Output format:

```markdown
# Project Brief: [Project Name]

## 📊 Current Status
**Health**: [On Track / At Risk / Blocked]
**Last Updated**: [Most recent activity date]

## 🎯 Project Overview
[Brief description, goals, key stakeholders from notes/Linear]

## 📈 Progress Summary

**Metrics**:
- Total Tasks: [N]
- Completed: [X] ([X/N]%)
- In Progress: [Y]
- Blocked: [Z]
- Not Started: [N]

**Velocity**: [Tasks completed per week/sprint]

## 🗂️ Key Resources

### Documentation (Obsidian)
[Relevant notes with summaries and relevance]

1. **[[Project Architecture Document]]**
   - Last updated: [date]
   - Key info: [1-2 sentence summary]
   - Relevance: [Why this matters now]

2. **[[Implementation Notes]]**
   - Last updated: [date]
   - Key info: [summary]
   - Relevance: [context]

3. **[[Design Decisions]]**
   - Last updated: [date]
   - Key decisions: [list]
   - Relevance: [impact]

### Active Work (Linear)
[In-progress issues with details]

- **[PROJ-123] Implement authentication**
  - Assignee: [name]
  - Started: [date]
  - Status: 60% complete
  - Blockers: Waiting on API keys

- **[PROJ-124] Database migration**
  - Assignee: [name]
  - Started: [date]
  - Status: 30% complete
  - Blockers: None

### Completed Work
[Recent done issues with completion dates]

### Blocked/Stalled Work
[Issues that need attention]

### Historical Context (Graphiti)
[Important decisions and milestones]

- **[Date]**: Initial project kickoff, decided on architecture
- **[Date]**: Scope change - added mobile support
- **[Date]**: Major blocker resolved - database access granted

## 🚧 Current Blockers

1. **[Blocker title]**
   - Impact: [High/Medium/Low]
   - Source: [Which note/issue mentions this]
   - Owner: [Who can resolve]
   - Since: [How long blocked]

2. **[Another blocker]**
   - [Details]

## 🔗 Dependencies

**This project depends on**:
- [Other project/system] - [why]
- [External team] - [for what]

**These depend on this project**:
- [Dependent project] - [waiting for what]

## 💡 Insights & Analysis

[Key insights from cross-system analysis]

### Patterns Observed
- [Pattern 1: e.g., "Velocity has slowed in last 2 weeks"]
- [Pattern 2: e.g., "Documentation lags implementation"]
- [Pattern 3: e.g., "Testing phase taking longer than planned"]

### Risks Identified
- [Risk 1: e.g., "Key engineer out next week"]
- [Risk 2: e.g., "Dependency on blocked external project"]

### Opportunities
- [Opportunity 1: e.g., "Can parallelize tasks X and Y"]
- [Opportunity 2: e.g., "Similar pattern solved in other project"]

### Health Assessment

**Velocity**: [🟢 Good / 🟡 Slowing / 🔴 Stalled]
- [Explanation with evidence]

**Clarity**: [🟢 Clear / 🟡 Some ambiguity / 🔴 Unclear]
- [Status of requirements and scope]

**Blockers**: [🟢 None / 🟡 Some / 🔴 Critical]
- [Current blocker status]

**Resources**: [🟢 Adequate / 🟡 Tight / 🔴 Insufficient]
- [Team capacity and availability]

**Communication**: [🟢 Good / 🟡 Could improve / 🔴 Poor]
- [Documentation and update frequency]

## 🎯 Recommended Next Steps

1. **[Priority 1]**
   - Action: [Specific action to take]
   - Owner: [Who should do it]
   - Timeline: [When]
   - Impact: [Why this matters]

2. **[Priority 2]**
   - [Details]

3. **[Priority 3]**
   - [Details]

## 📅 Key Milestones & Timeline

**Completed**:
- ✅ [Milestone 1] - [Date completed]
- ✅ [Milestone 2] - [Date completed]

**Upcoming**:
- 🎯 [Next milestone] - [Target date]
- 🎯 [Future milestone] - [Target date]

**Estimated Completion**: [Date based on current velocity]

## 📊 Timeline Visualization

```
[Date 1]: Project kickoff
    |
[Date 2]: Phase 1 complete
    |
[Date 3]: Blocker encountered
    |
[Today]: Current state
    |
[Future]: Estimated completion
```

## 🔄 Recent Activity (Last 7 Days)

- [Activity 1: e.g., "Note updated: Architecture decisions"]
- [Activity 2: e.g., "3 issues moved to Done"]
- [Activity 3: e.g., "Blocker resolved"]
```

## MCP Tool Usage Examples

From **obsidian-reader** skill:
```
# Search for project notes
mcp__MCP_DOCKER__obsidian_simple_search(
    query="[project name] OR [related keywords]",
    context_length=200
)

# Get specific note
mcp__MCP_DOCKER__obsidian_get_file_contents(filepath="[path]")

# Get batch of notes
mcp__MCP_DOCKER__obsidian_batch_get_file_contents(
    filepaths=["note1.md", "note2.md"]
)
```

From **linear-integration** skill:
```
# Get project details
mcp__linear__get_project(query="[project name]")

# Get all project issues
mcp__linear__list_issues(
    project="[project name]",
    includeArchived=false
)

# Get issues by label
mcp__linear__list_issues(label="[project-tag]")

# Get specific issue details
mcp__linear__get_issue(id="PROJ-123")
```

From **graphiti-memory** skill:
```python
# ⚠️ CRITICAL: Always pass group_ids=["work"] for all Chief of Staff operations!

# Search project facts - include project name in query
mcp__graphiti__search_memory_facts(
    query="[project name] decisions milestones blockers",
    group_ids=["work"],  # Always use "work" for Chief of Staff
    max_facts=20
)

# Find project entities
mcp__graphiti__search_nodes(
    query="[project name]",
    group_ids=["work"],  # Always use "work" for Chief of Staff
    max_nodes=10
)

# Get project episodes from "work" graph
mcp__graphiti__get_episodes(
    group_ids=["work"],  # Always use "work" for Chief of Staff
    max_episodes=15
)
```

## Best Practices

1. **Comprehensive**: Don't miss any data source
2. **Balanced**: Show both progress and problems honestly
3. **Evidence-Based**: Reference specific notes/issues/facts
4. **Actionable**: End with clear, prioritized next steps
5. **Historical**: Provide context on project evolution
6. **Visual**: Use structure that's easy to scan

## Optional Parameters

User can customize the brief:

```
/project-brief [name]              # Standard comprehensive brief
/project-brief [name] --quick      # Quick status only
/project-brief [name] --timeline   # Focus on timeline/history
/project-brief [name] --health     # Focus on health assessment
/project-brief [name] --blockers   # Focus on blockers/risks
```

## Error Handling

- If project name ambiguous: List options, ask user to clarify
- If project not found in Linear: Search Obsidian only, note limitation
- If Graphiti has no history: Note it, focus on current state
- If too many results: Prioritize by recency and relevance
- If conflicting info: Present both versions, note inconsistency

## After the Brief

Ask user:
```
Would you like me to:
- Dive deeper into any specific aspect?
- Update Linear issue priorities based on this analysis?
- Create action items for blockers?
- Generate a summary to share with your team?
```

## Related Commands

- `/morning-brief` - Daily briefing across all projects
- `/evening-sync` - End-of-day review with Linear updates
