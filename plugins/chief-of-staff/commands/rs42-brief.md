---
description: Generate comprehensive morning brief for RS42 startup work. Synthesizes Linear issues, Obsidian notes, Graphiti memory, and GitHub activity for project status.
---

# /rs42-brief

Generate a comprehensive RS42 startup briefing using multi-agent orchestration.

## What This Command Does

This command spawns a multi-agent system that:

1. **Gathers data from 4 sources in parallel**:
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

When you run this command, spawn the **rs42-orchestrator** agent:

```
Task(
  subagent_type="chief-of-staff:rs42-orchestrator",
  description="Generate RS42 startup morning brief",
  prompt="Generate a comprehensive RS42 startup brief. Spawn all 4 data-gathering agents in parallel (rs42-linear-gatherer, rs42-obsidian-gatherer, rs42-graphiti-gatherer, rs42-github-gatherer), collect their results, and synthesize into a formatted briefing following the Source of Truth hierarchy. Include project status, gap analysis, and prioritized recommendations for today."
)
```

## Expected Output

The orchestrator will produce a formatted briefing including:

- **Active Projects** - Table of Linear projects with status
- **In-Progress Issues** - Current work items
- **Blockers & Dependencies** - What's preventing progress
- **GitHub Activity** - Recent commits, PRs, reviews needed
- **Documentation Context** - Relevant Obsidian notes
- **Historical Context** - Graphiti insights and patterns
- **Gap Analysis** - Stale items and missing updates
- **Recommended Focus** - Prioritized actions for today

## Architecture

```
/rs42-brief (command)
       │
       ▼
┌─────────────────────────────────────┐
│  rs42-orchestrator (sonnet)         │
│  - Spawns gatherers in parallel     │
│  - Collects and synthesizes data    │
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

- If Linear is unavailable, the brief will note this prominently (it's the primary source)
- If other sources fail, the brief continues with available data
- Partial data is clearly marked in the output

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
