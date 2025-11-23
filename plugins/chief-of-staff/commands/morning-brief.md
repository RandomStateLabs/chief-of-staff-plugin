---
description: Start your day with an intelligent morning briefing that synthesizes overnight context from Obsidian, Linear, and Graphiti to show project status and priorities
---

# Morning Brief Command

You are now in **Morning Brief Mode**. Help the user start their day with clarity by gathering and synthesizing their current project status across all systems.

## Command Purpose

Generate a comprehensive morning briefing that:
1. Reviews overnight context (journal, recent notes, task updates)
2. Shows current project status and momentum
3. Highlights priorities and blockers for today
4. Provides actionable insights to start the day

## Workflow

### Step 1: Activate Relevant Skills

This command should leverage:
- **obsidian-reader** skill - to access today's journal and recent notes
- **linear-integration** skill - to get in-progress tasks
- **graphiti-memory** skill - to query recent project context

### Step 2: Consider Agent Spawning

For complex morning briefs with many projects, consider spawning the **morning-planner agent**:

```
When to use agent:
- User has 5+ active projects
- Need cross-system synthesis across 10+ data points
- Want isolated context for detailed analysis
- Brief requires complex pattern recognition

When to use skills directly:
- Simple brief with 1-3 projects
- Quick status check
- User wants fast response
- Limited context needed
```

### Step 3: Gather Morning Context

Use skills to:
1. Check for today's daily journal in Obsidian
2. Get notes modified in last 48 hours
3. Fetch all "In Progress" Linear issues assigned to user
4. Query Graphiti for recent project facts

### Step 4: Synthesize and Present

Generate morning brief following this structure:

```markdown
# Morning Brief - [Date]

## 📊 Project Status Review
[AI synthesis of where you are across active projects]
- Project 1: [Status, momentum, key updates]
- Project 2: [Status, momentum, key updates]
- Project 3: [Status, momentum, key updates]

## 📔 Today's Journal
[Today's journal excerpt if exists, or "No journal entry yet - consider starting one!"]

## 📝 Recent Activity (48h)
[Modified notes with relevance and context]
- [[Note Title 1]] - [Why this matters]
- [[Note Title 2]] - [Why this matters]
- [[Note Title 3]] - [Why this matters]

## ✅ Active Work
[Linear issues in progress with brief status]
- [PROJ-123] Task title - [Current state]
- [PROJ-124] Task title - [Current state]

## 🎯 What This Means
[Synthesis section with insights]
- **Momentum**: [What's moving forward]
- **Blockers**: [What needs attention]
- **Priorities**: [What to focus on today]

## 💡 Suggested Focus
1. [Priority action item 1]
2. [Priority action item 2]
3. [Priority action item 3]
```

## MCP Tool Usage Examples

From **obsidian-reader** skill:
```
# Get today's journal
mcp__MCP_DOCKER__obsidian_get_periodic_note(period="daily")

# Get recent changes
mcp__MCP_DOCKER__obsidian_get_recent_changes(days=2, limit=10)
```

From **linear-integration** skill:
```
# Get in-progress issues
mcp__linear__list_issues(assignee="me", state="In Progress")
```

From **graphiti-memory** skill:
```
# Search for recent project context
mcp__graphiti__search_memory_facts(query="project status", max_facts=10)
```

## Best Practices

1. **Be Concise**: Morning briefs should be scannable in 2-3 minutes
2. **Prioritize Overnight Changes**: Focus on what's new or changed
3. **Cross-Reference**: Connect related info across systems
4. **Action-Oriented**: End with clear priorities
5. **Honest Assessment**: Show both progress and blockers

## Optional Parameters

User can customize the brief:

```
/morning-brief                    # Standard morning brief
/morning-brief --project "Auth"   # Focus on specific project
/morning-brief --detailed         # More comprehensive analysis
/morning-brief --quick            # Quick status only, skip synthesis
```

## Error Handling

- If Obsidian not accessible: Use Linear + Graphiti only
- If Linear API fails: Use cached Graphiti data if available
- If no recent activity: Note it and suggest focus areas
- If multiple projects: Prioritize by recent activity or user input

## After the Brief

Ask user:
```
Would you like me to:
- Dive deeper into any specific project? (use /project-brief)
- Create a focus plan for today?
- Update any task priorities in Linear?
```

## Related Commands

- `/evening-sync` - End-of-day review and Linear updates
- `/project-brief [name]` - Deep dive into specific project
