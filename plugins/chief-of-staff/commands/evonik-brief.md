---
description: Generate comprehensive morning brief for Evonik employment work. Synthesizes Azure DevOps tasks, Granola meetings, Obsidian notes, Graphiti memory, and Linear tracking.
---

# /evonik-brief

Generate a comprehensive morning briefing for Evonik employment work using multi-agent orchestration.

## What This Command Does

This command spawns a multi-agent system that:

1. **Gathers data from 5 sources in parallel**:
   - Azure DevOps (CS Enterprise AI project) - Official work items
   - Granola - Recent meeting notes and action items
   - Obsidian - Project documentation and notes
   - Graphiti - Knowledge graph and memory
   - Linear - Personal tracking (context only)

2. **Synthesizes following Source of Truth Hierarchy**:
   ```
   PRIMARY:    Azure DevOps + Granola (defines work)
   CONTEXT:    Obsidian + Graphiti (enriches understanding)
   TRACKING:   Linear (personal reflection only)
   ```

3. **Performs gap analysis** to identify:
   - Meeting commitments not tracked as work items
   - Work items lacking recent context
   - Potential conflicts between sources

4. **Generates prioritized recommendations** for today's focus

## Usage

Simply run:
```
/evonik-brief
```

No parameters required. The brief covers the last 7 days of context.

## Workflow

When you run this command, spawn the **evonik-orchestrator** agent:

```
Task(
  subagent_type="evonik-orchestrator",
  description="Generate Evonik morning brief",
  prompt="Generate a comprehensive Evonik morning brief. Spawn all 4 data-gathering agents in parallel (evonik-azure-gatherer, evonik-granola-gatherer, evonik-context-gatherer, evonik-linear-gatherer), collect their results, and synthesize into a formatted briefing following the Source of Truth hierarchy. Include gap analysis and prioritized recommendations for today."
)
```

## Expected Output

The orchestrator will produce a formatted briefing including:

- **Active Work Items** - Table of Azure DevOps items with status
- **Blockers & Dependencies** - What's preventing progress
- **Meeting Context** - Recent decisions, action items, commitments
- **Documentation Context** - Relevant notes and historical facts
- **Personal Tracking** - Linear issues (context only)
- **Gap Analysis** - Discrepancies and missing items
- **Recommended Focus** - Prioritized actions for today

## Architecture

```
/evonik-brief (command)
       │
       ▼
┌─────────────────────────────────────┐
│  evonik-orchestrator (sonnet)       │
│  - Spawns gatherers in parallel     │
│  - Collects and synthesizes data    │
│  - Generates final briefing         │
└──────────────┬──────────────────────┘
               │
    ┌──────────┼──────────┬──────────┐
    ▼          ▼          ▼          ▼
┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│ Azure  │ │Granola │ │Context │ │ Linear │
│Gatherer│ │Gatherer│ │Gatherer│ │Gatherer│
│(haiku) │ │(haiku) │ │(haiku) │ │(haiku) │
└────────┘ └────────┘ └────────┘ └────────┘
    │          │          │          │
    ▼          ▼          ▼          ▼
  Azure      Granola   Obsidian   Linear
  DevOps    Meetings  + Graphiti  Issues
```

## Error Handling

- If Azure DevOps is unavailable, the brief will note this prominently (it's the primary source)
- If other sources fail, the brief continues with available data
- Partial data is clearly marked in the output

## Related Commands

- `/morning-brief` - General morning briefing (projects context)
- `/evening-sync` - End-of-day sync with Linear updates
- `/project-brief [name]` - Deep dive into specific project

## Source of Truth Note

For Evonik employment work:
- **Azure DevOps (CS Enterprise AI)** is the official source of work assignments
- **Granola meetings** capture decisions and commitments
- **Linear** is for personal tracking/reflection only - never treat as source of truth
