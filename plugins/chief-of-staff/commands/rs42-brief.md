---
description: Generate comprehensive morning brief for RS42 startup work. Synthesizes Linear issues, Obsidian notes, Graphiti memory, and GitHub activity for project status.
---

# /rs42-brief

Generate a comprehensive RS42 startup briefing using parallel data gathering and synthesis.

## What This Command Does

This command:

1. **Gathers data from 4 sources in parallel** (spawned directly from main Claude):
   - Linear - RS42 team issues and projects
   - Obsidian - RS42 operations notes and documentation
   - Graphiti - Work knowledge graph and memory
   - GitHub - RS42 repository activity and PRs

2. **Synthesizes following Source of Truth Hierarchy**:
   ```
   PRIMARY:    Linear (defines work and projects)
   CONTEXT:    Obsidian + Graphiti (enriches understanding)
   ACTIVITY:   GitHub (code and PR status)
   ```

3. **Performs gap analysis** to identify:
   - Stale issues needing attention
   - Projects without recent activity
   - Documentation gaps
   - PR review needs

4. **Generates prioritized recommendations** for today's focus

## Usage

Simply run:
```
/rs42-brief
```

No parameters required. The brief covers all RS42 projects and recent activity.

## Workflow

When you run this command, spawn **all 4 gatherer agents in PARALLEL** using a SINGLE message with multiple Task tool calls:

```
Use the Task tool 4 times in the SAME message:

1. Task(
     subagent_type="chief-of-staff:rs42-linear-gatherer",
     description="Gather RS42 Linear issues",
     prompt="Gather RS42 Linear issues and projects. Get all active projects and in-progress/todo issues. Return structured JSON with projects, issues, priorities, and recent updates.",
     run_in_background=true
   )

2. Task(
     subagent_type="chief-of-staff:rs42-obsidian-gatherer",
     description="Gather RS42 Obsidian notes",
     prompt="Gather RS42-related notes from Obsidian. Search for RS42 operations, projects, and documentation. Return structured JSON.",
     run_in_background=true
   )

3. Task(
     subagent_type="chief-of-staff:rs42-graphiti-gatherer",
     description="Gather RS42 Graphiti memory",
     prompt="Gather RS42-related context from Graphiti memory. Search for project decisions, patterns, and historical context. Return structured JSON.",
     run_in_background=true
   )

4. Task(
     subagent_type="chief-of-staff:rs42-github-gatherer",
     description="Gather RS42 GitHub activity",
     prompt="Gather RS42 GitHub activity. Get recent commits, open PRs, and repository status. Return structured JSON.",
     run_in_background=true
   )
```

**CRITICAL**: All 4 Task calls MUST be in the SAME message to run in parallel.

Then collect results using TaskOutput for each agent.

## Synthesis Instructions

After collecting all results, synthesize following the Source of Truth hierarchy:

### Step 1: Apply Hierarchy

1. **Start with Linear** - These are the official projects and issues
2. **Overlay GitHub activity** - Add code progress and PR status
3. **Enrich with Obsidian/Graphiti** - Add documentation and historical context
4. **Cross-reference** - Identify alignment between planned work and actual activity

### Step 2: Perform Gap Analysis

Compare data across sources to identify:

1. **Stale issues** - Linear issues with no recent activity
2. **Orphan commits** - GitHub activity not linked to issues
3. **Documentation gaps** - Projects without Obsidian notes
4. **Review needs** - PRs waiting for review
5. **Blocked items** - Issues marked as blocked or waiting

### Step 3: Generate Prioritized Recommendations

Based on synthesis, recommend:

1. **Focus areas for today** - Highest impact work
2. **Quick wins** - Small items that can be completed
3. **PRs to review** - Code reviews needed
4. **Documentation to update** - Notes that need refresh
5. **Issues to triage** - Stale items needing decisions

## Output Format

Generate the final briefing in this exact structure:

```markdown
# RS42 Startup Brief - [Today's Date]

## Project Overview

| Project | Status | Progress | Last Activity |
|---------|--------|----------|---------------|
| [name] | [status] | [%] | [date] |

---

## Active Issues (Linear)

### In Progress
| ID | Title | Priority | Project | Last Update |
|----|-------|----------|---------|-------------|
| [id] | [title] | [priority] | [project] | [date] |

### Todo (Ready to Start)
| ID | Title | Priority | Project |
|----|-------|----------|---------|
| [id] | [title] | [priority] | [project] |

### Blocked
- [Issue] - **Reason**: [why blocked]

---

## GitHub Activity

### Recent Commits (Last 7 Days)
| Repo | Commit | Message | Date |
|------|--------|---------|------|
| [repo] | [sha] | [message] | [date] |

### Open Pull Requests
| Repo | PR | Title | Status | Reviews |
|------|----|----- |--------|---------|
| [repo] | [#] | [title] | [status] | [review status] |

### Repository Health
| Repo | Last Commit | Open Issues | Open PRs |
|------|-------------|-------------|----------|
| [repo] | [date] | [count] | [count] |

---

## Documentation Context (Obsidian/Graphiti)

### Relevant Notes
- [[Note Title]] - [Brief relevance]

### Historical Decisions
- [Past decision relevant to current work]

### Patterns Identified
- [Recurring theme or pattern]

---

## Gap Analysis

### Stale Issues (>14 Days Inactive)
- [Issue] - Last activity: [date]

### Missing Documentation
- [Project/feature without docs]

### PRs Needing Review
- [PR] - Waiting: [days]

### Activity Misalignment
- [GitHub activity not linked to Linear issue]

---

## Recommended Focus Today

### Priority 1: [Title]
**Why**: [Reasoning based on synthesis]
**First Step**: [Specific action]
**Linear Issue**: [ID if applicable]

### Priority 2: [Title]
**Why**: [Reasoning]
**First Step**: [Action]

### Priority 3: [Title]
**Why**: [Reasoning]
**First Step**: [Action]

### Quick Wins
- [ ] [Small task that can be done quickly]
- [ ] [Another quick task]

### Code Reviews Needed
- [ ] [PR to review]

---

*Generated: [timestamp]*
*Sources: Linear (RS42), Obsidian, Graphiti, GitHub*
```

## Architecture

```
/rs42-brief (command)
       │
       ▼
┌─────────────────────────────────────┐
│  Main Claude (spawns directly)      │
│  - Spawns all 4 gatherers in parallel│
│  - Collects results with TaskOutput │
│  - Synthesizes following hierarchy  │
│  - Generates startup briefing       │
└──────────────┬──────────────────────┘
               │
    ┌──────────┼──────────┬──────────┐
    ▼          ▼          ▼          ▼
┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│ Linear │ │Obsidian│ │Graphiti│ │ GitHub │
│Gatherer│ │Gatherer│ │Gatherer│ │Gatherer│
│(haiku) │ │(haiku) │ │(haiku) │ │(haiku) │
└────────┘ └────────┘ └────────┘ └────────┘
    │          │          │          │
    ▼          ▼          ▼          ▼
  Linear     RS42        Work       GitHub
  RS42      Notes       Memory      Repos
  Team
```

## Error Handling

| Scenario | Handling |
|----------|----------|
| Linear agent fails | **CRITICAL** - Report error prominently, limited brief possible |
| GitHub agent fails | Continue, note "Code activity unavailable" |
| Obsidian agent fails | Continue, note "Documentation context unavailable" |
| Graphiti agent fails | Continue, note "Historical context unavailable" |
| Agent returns empty data | Include section with "No data found" note |

## Related Commands

- `/evonik-brief` - Evonik employment work briefing
- `/life-brief` - Personal life briefing
- `/morning-brief` - Cross-project synthesis briefing
- `/project-brief [name]` - Deep dive into specific project

## Source of Truth Note

For RS42 startup work:
- **Linear (RS42 team)** is the official source of projects and tasks
- **GitHub** shows actual code activity
- **Obsidian/Graphiti** provide context and historical decisions
