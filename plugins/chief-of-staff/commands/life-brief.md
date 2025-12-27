---
description: Generate comprehensive morning brief for personal life. Synthesizes Obsidian journals, Graphiti personal memory, and personal reflections for daily alignment.
---

# /life-brief

Generate a comprehensive personal life briefing using parallel data gathering and synthesis.

## What This Command Does

This command:

1. **Gathers data from 2 sources in parallel** (spawned directly from main Claude):
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

When you run this command, spawn **both gatherer agents in PARALLEL** using a SINGLE message with multiple Task tool calls:

```
Use the Task tool 2 times in the SAME message:

1. Task(
     subagent_type="chief-of-staff:life-obsidian-gatherer",
     description="Gather personal journal entries",
     prompt="Gather recent personal journal entries from Obsidian. Focus on Personal/Journal/ folder. Extract gratitude entries, morning thoughts, habit tracking, top priorities, and evening reflections. Return structured JSON.",
     run_in_background=true
   )

2. Task(
     subagent_type="chief-of-staff:life-graphiti-gatherer",
     description="Gather personal memory context",
     prompt="Gather personal context from Graphiti memory. Search for personal insights, patterns, and historical reflections. Return structured JSON.",
     run_in_background=true
   )
```

**CRITICAL**: Both Task calls MUST be in the SAME message to run in parallel.

Then collect results using TaskOutput for each agent.

## Synthesis Instructions

After collecting all results, synthesize following these steps:

### Step 1: Extract Recurring Themes

From journal entries, identify:
- What topics appear across multiple entries?
- What gratitude themes recur?
- What priorities are consistently mentioned?

### Step 2: Identify Alignment Gaps

Compare journal entries against stated vision:
1. **Vision statement** - "Who do you want to be?" from journals
2. **Current habits** - Meditation, AA, gym, sobriety tracking
3. **Alignment score** - Are daily actions matching vision?
4. **Gap identification** - What's not aligned?

### Step 3: Celebrate Wins

Note positive patterns and growth:
- Habits maintained
- Goals achieved
- Positive emotional trends

### Step 4: Surface Action Items

Identify personal tasks mentioned but not completed in journals.

### Step 5: Generate Personal Focus

Based on synthesis, recommend:
1. **Today's intention** - Single focus for the day
2. **Habit priorities** - Which habits to focus on
3. **Gratitude reminder** - Surface recent gratitude themes
4. **Growth opportunity** - One area for improvement

## Output Format

Generate the final briefing in this exact structure:

```markdown
# Personal Life Brief - [Today's Date]

## Morning Reflection

> [Inspirational quote from recent journal if found]

### How You've Been Feeling
[Summary of recent emotional themes from journals]

### Gratitude Themes
1. [Recurring gratitude topic 1]
2. [Recurring gratitude topic 2]
3. [Recurring gratitude topic 3]

---

## Habit Tracking

### Current Focus Habits
| Habit | Recent Status | Streak/Notes |
|-------|---------------|--------------|
| Meditation | [status] | [notes] |
| AA | [status] | [notes] |
| Gym | [status] | [notes] |
| Journaling | [status] | [notes] |

### Alignment Check
**Vision**: "[Who do you want to be quote]"
**Current Alignment**: [assessment]

---

## Personal Priorities

### From Recent Journals
- [ ] [Priority 1 from journals]
- [ ] [Priority 2 from journals]
- [ ] [Priority 3 from journals]

### Pending Personal Tasks
- [Task mentioned but not completed]

---

## Growth Insights

### Recurring Themes
- **[Theme 1]**: [What journals reveal about this]
- **[Theme 2]**: [Pattern observed]

### Wins to Celebrate
- [Positive pattern or accomplishment noted]

### Areas for Growth
- [Gentle observation about improvement opportunity]

---

## Today's Focus

### Intention
**[Single clear intention for today]**

### Affirmation
[Supportive message based on journal themes]

### First Step
[One concrete action to start the day aligned]

---

*Generated: [timestamp]*
*Sources: Obsidian Personal Journals, Graphiti Personal Memory*
```

## Architecture

```
/life-brief (command)
       │
       ▼
┌─────────────────────────────────────┐
│  Main Claude (spawns directly)      │
│  - Spawns both gatherers in parallel│
│  - Collects results with TaskOutput │
│  - Synthesizes personal insights    │
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

| Scenario | Handling |
|----------|----------|
| Obsidian agent fails | **CRITICAL** - Report error, limited brief possible |
| Graphiti agent fails | Continue with journal data only |
| No recent journals | Note gap, encourage journaling |
| Agent returns empty data | Include section with supportive message |

## Tone Guidelines

When generating the brief:
1. **Supportive, not judgmental** - This is personal reflection
2. **Encouraging** - Celebrate small wins
3. **Honest** - Surface alignment gaps gently
4. **Actionable** - Always end with concrete next step
5. **Personal** - Use "you" language, reference their actual words

## Related Commands

- `/evonik-brief` - Evonik employment work briefing
- `/rs42-brief` - RS42 startup projects briefing
- `/morning-brief` - Cross-project synthesis briefing

## Personal Context Note

This brief focuses on:
- **Personal/Journal/** folder in Obsidian
- **Graphiti personal group** for memory
- Daily alignment with personal vision and habits
