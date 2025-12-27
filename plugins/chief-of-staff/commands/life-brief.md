---
description: Generate comprehensive morning brief for personal life. Synthesizes Obsidian journals, Graphiti personal memory, and personal reflections for daily alignment.
---

# /life-brief

Generate a comprehensive personal life briefing using multi-agent orchestration.

## What This Command Does

This command spawns a multi-agent system that:

1. **Gathers data from 2 sources in parallel**:
   - Obsidian - Personal journals, daily notes, habit tracking
   - Graphiti - Personal knowledge graph and memory

2. **Synthesizes following Personal Focus Areas**:
   ```
   REFLECTION:   Journal entries, gratitude, morning thoughts
   HABITS:       Habit tracking, streak data, alignment
   PRIORITIES:   Top 3 priorities, pending tasks
   GROWTH:       Patterns, insights, personal development
   ```

3. **Performs alignment analysis** to identify:
   - Vision alignment gaps
   - Habit consistency patterns
   - Recurring themes in journals
   - Growth opportunities

4. **Generates prioritized focus** for today's personal goals

## Usage

Simply run:
```
/life-brief
```

No parameters required. The brief covers recent journal entries and personal context.

## Workflow

When you run this command, spawn the **life-orchestrator** agent:

```
Task(
  subagent_type="chief-of-staff:life-orchestrator",
  description="Generate personal life morning brief",
  prompt="Generate a comprehensive personal life brief. Spawn the 2 data-gathering agents in parallel (life-obsidian-gatherer, life-graphiti-gatherer), collect their results, and synthesize into a formatted briefing focused on personal alignment, habits, and growth. Include reflection analysis and prioritized focus for today."
)
```

## Expected Output

The orchestrator will produce a formatted briefing including:

- **Recent Journal Insights** - Key themes from recent entries
- **Gratitude & Mindset** - Gratitude patterns and morning thoughts
- **Habit Tracking** - Current streaks and alignment
- **Top Priorities** - Personal priorities from journals
- **Growth Patterns** - Recurring themes and insights
- **Vision Alignment** - How current actions align with long-term vision
- **Today's Focus** - Prioritized personal goals

## Architecture

```
/life-brief (command)
       │
       ▼
┌─────────────────────────────────────┐
│  life-orchestrator (sonnet)         │
│  - Spawns gatherers in parallel     │
│  - Collects and synthesizes data    │
│  - Generates personal briefing      │
└──────────────┬──────────────────────┘
               │
    ┌──────────┴──────────┐
    ▼                     ▼
┌────────────┐     ┌────────────┐
│  Obsidian  │     │  Graphiti  │
│  Gatherer  │     │  Gatherer  │
│  (haiku)   │     │  (haiku)   │
└────────────┘     └────────────┘
    │                     │
    ▼                     ▼
  Personal             Personal
  Journals             Memory
```

## Error Handling

- If Obsidian is unavailable, the brief will note this prominently
- If Graphiti fails, the brief continues with journal data
- Partial data is clearly marked in the output

## Related Commands

- `/evonik-brief` - Evonik employment work briefing
- `/rs42-brief` - RS42 startup projects briefing
- `/morning-brief` - Cross-project synthesis briefing

## Personal Context Note

This brief focuses on:
- **Personal/Journal/** folder in Obsidian
- **Graphiti personal group** for memory
- Daily alignment with personal vision and habits
