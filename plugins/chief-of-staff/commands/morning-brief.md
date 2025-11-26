---
description: Start your day with a project status briefing - synthesizes Linear tasks, Obsidian project notes, and Graphiti memory to show what you're working on and what needs attention
---

# Morning Brief Command

You are now in **Morning Brief Mode**. Help the user start their day with clarity by gathering and synthesizing their current **project status** across all systems.

## Command Purpose

Generate a project-focused morning briefing that:
1. **Loads context from memory** (Graphiti) - understand what you were working on
2. Shows all active Linear projects and their current status
3. Lists in-progress and blocked tasks across projects
4. Surfaces relevant project documentation from Obsidian
5. Highlights priorities and blockers for today


## Workflow

### Step 1: Activate Relevant Skills

This command should leverage (in this order):
- **graphiti-memory** skill - FIRST: Load prior context, decisions, patterns
- **linear-integration** skill - SECOND: Get current projects, issues, status
- **obsidian-reader** skill - THIRD: Get project-related notes and documentation

### Step 2: Spawn Morning Planner Agent

**ALWAYS spawn the `morning-planner` agent** for morning briefs. This is a context-heavy operation that benefits from the agent's dedicated 200K context window.

```
Use the Task tool:
- subagent_type: "chief-of-staff:morning-planner"
- prompt: Include user's request and any parameters (--project, --quick, etc.)
```

**Why always spawn?**
- Graphiti queries return substantial context (facts, episodes, patterns)
- Linear queries across multiple teams (RS42, Evonik) generate many issues
- Obsidian searches surface multiple relevant notes
- Cross-system synthesis requires holding all this context simultaneously
- Keeps the main conversation clean for follow-up questions

**DO NOT** attempt to run the morning brief workflow directly in the main conversation.

### Step 3: Gather Project Data (Graphiti → Linear → Obsidian)

**Step 3a: Get Project Context from Graphiti (FIRST)**
```python
# ⚠️ CRITICAL: Always pass group_ids=["work"] for all Chief of Staff operations!
# Query Graphiti FIRST to prime context before looking at Linear tasks

# Query project facts - decisions, blockers, patterns
mcp__graphiti__search_memory_facts(
    query="project status decisions blockers milestones priorities work",
    group_ids=["work"],
    max_facts=15
)

# Get recent work episodes - what was being worked on
mcp__graphiti__get_episodes(
    group_ids=["work"],
    max_episodes=10
)
```

**Why first?** Graphiti memory tells you:
- What project/task was the focus recently
- Key decisions already made
- Blockers that were encountered
- Patterns from past work (what worked, what didn't)

This context helps you interpret Linear data more effectively.

**Step 3b: Get Linear Projects and Issues (SECOND)**
```
# Get all projects for RS42 team
mcp__linear__list_projects(team="RS42")

# Get all issues for RS42 team (filter by status after retrieval)
mcp__linear__list_issues(team="RS42")

# If user also has Evonik work, get those too
mcp__linear__list_issues(team="Evonik")
```

**Note**: The Linear MCP queries by TEAM, not by assignee. Filter results locally for "In Progress", "Todo", "Blocked" statuses after retrieval.

**Step 3c: Get Project Documentation from Obsidian (THIRD)**
```
# Search for project-related notes (use project names from Linear)
mcp__MCP_DOCKER__obsidian_simple_search(query="[project name]", context_length=150)

# Get recently modified project notes
mcp__MCP_DOCKER__obsidian_get_recent_changes(days=7, limit=10)

# Focus on "1 - Main Notes/" folder for project documentation
mcp__MCP_DOCKER__obsidian_list_files_in_dir(dirpath="1 - Main Notes")
```

### Step 4: Synthesize and Present

Generate morning brief using **Eisenhower Matrix** format to prioritize work:

```markdown
# Morning Brief - [Date]

## 📊 Active Projects Overview
[Quick list of in-progress projects with completion %]

---

# 🎯 EISENHOWER MATRIX

## 🔴 DO FIRST (Urgent + Important)
*Blockers, deadlines today, critical issues*

| Project | Task | Why Urgent |
|---------|------|------------|
| [Project] | [ISSUE-XXX] Task name | [Deadline/Blocker reason] |

**Action Items:**
1. [Specific action to take NOW]
2. [Next critical action]

---

## 🟡 SCHEDULE (Important + Not Urgent)
*Strategic work, project advancement, no immediate deadline*

| Project | Task | Target Date |
|---------|------|-------------|
| [Project] | [ISSUE-XXX] Task name | [When to work on it] |

**This Week's Focus:**
- [Key deliverable 1]
- [Key deliverable 2]

---

## 🟠 DELEGATE (Urgent + Not Important)
*Tasks that need doing but could be handed off or automated*

| Task | Suggestion |
|------|------------|
| [Task] | [Who/what could handle this] |

---

## ⚪ ELIMINATE (Not Urgent + Not Important)
*Low-value tasks, distractions, consider dropping*

- [Task that's consuming time but not delivering value]

---

## 📝 Project Context
[Key Obsidian notes and Graphiti facts relevant to today's work]

## 💡 Memory Insights
[Patterns from Graphiti - past decisions, recurring blockers, etc.]
```

### Eisenhower Classification Rules

When classifying tasks:

**URGENT** = Has deadline within 48h OR is blocking other work OR external dependency waiting
**IMPORTANT** = Directly advances project goals OR has significant business impact

- **Blocked issues** → Always DO FIRST (urgent + important)
- **In Progress with deadline** → DO FIRST if <48h, else SCHEDULE
- **In Progress no deadline** → SCHEDULE (important, not urgent)
- **Backlog items** → Usually SCHEDULE or ELIMINATE
- **Meetings/admin** → Often DELEGATE or ELIMINATE

## Best Practices

1. **Project-First**: Always lead with Linear project status
2. **Be Specific**: Show actual issue IDs and task names
3. **Surface Blockers**: Make blocked items highly visible
4. **Actionable Priorities**: End with concrete next steps
5. **Connect the Dots**: Link Obsidian notes to relevant projects

## Optional Parameters

```
/morning-brief                    # Full project status brief
/morning-brief --project "Auth"   # Focus on specific project only
/morning-brief --quick            # Just show blocked items and priorities
```

## Error Handling

- If Linear API fails: Note it, use Graphiti cached data if available
- If no Linear projects: Suggest creating project structure
- If Obsidian not accessible: Continue with Linear + Graphiti only
- If no in-progress work: Highlight backlog items to consider starting

## After the Brief

Ask user:
```
Would you like me to:
- Dive deeper into a specific project? (use /project-brief [name])
- Update any task statuses in Linear?
- Help prioritize the backlog?
```

## Related Commands

- `/evening-sync` - End-of-day review and Linear updates
- `/project-brief [name]` - Deep dive into specific project
- `/project-sync` - Sync project status across all systems
