---
description: Generate comprehensive morning brief for Evonik employment work. Synthesizes Azure DevOps tasks, Granola meetings, Obsidian notes, Graphiti memory, and Linear tracking.
---

# /evonik-brief

Generate a comprehensive Evonik employment morning briefing using parallel data gathering and synthesis.

## What This Command Does

This command:

1. **Gathers data from 4 sources in parallel** (spawned directly from main Claude):
   - Azure DevOps (CS Enterprise AI project) - Official work items
   - Granola - Recent meeting notes and action items
   - Obsidian + Graphiti - Project documentation and memory
   - Linear - Personal tracking (context only)

2. **Synthesizes following Source of Truth Hierarchy**:
   ```
   PRIMARY:    Azure DevOps + Granola (defines work)
   CONTEXT:    Obsidian + Graphiti (enriches understanding)
   TRACKING:   Linear (personal reflection only)
   ```

3. **Performs gap analysis** to identify discrepancies and missing items

4. **Generates prioritized recommendations** for today's focus

## Usage

Simply run:
```
/evonik-brief
```

No parameters required. The brief covers the last 30 days of meeting context.

## Workflow

When you run this command, spawn **all 4 gatherer agents in PARALLEL** using a SINGLE message with multiple Task tool calls:

```
Use the Task tool 4 times in the SAME message:

1. Task(
     subagent_type="chief-of-staff:evonik-azure-gatherer",
     description="Gather Azure DevOps work items",
     prompt="Gather my active Azure DevOps work items from CS Enterprise AI project. Return structured JSON with work items, states, priorities, and recent comments.",
     run_in_background=true
   )

2. Task(
     subagent_type="chief-of-staff:evonik-granola-gatherer",
     description="Gather Granola meetings",
     prompt="Gather recent Evonik meetings from the last 30 days. Extract decisions, action items, commitments, and blockers. Return structured JSON.",
     run_in_background=true
   )

3. Task(
     subagent_type="chief-of-staff:evonik-context-gatherer",
     description="Gather Obsidian/Graphiti context",
     prompt="Gather Evonik-related context from Obsidian notes and Graphiti memory. Search for project status, decisions, and documentation. Return structured JSON.",
     run_in_background=true
   )

4. Task(
     subagent_type="chief-of-staff:evonik-linear-gatherer",
     description="Gather Linear tracking",
     prompt="Gather my Evonik Linear issues for personal tracking context. Note: This is NOT source of truth. Return structured JSON.",
     run_in_background=true
   )
```

**CRITICAL**: All 4 Task calls MUST be in the SAME message to run in parallel.

Then collect results using TaskOutput for each agent.

## Synthesis Instructions

After collecting all results, synthesize following the Source of Truth hierarchy:

### Step 1: Apply Hierarchy

1. **Start with Azure DevOps** - These are the official work items
2. **Overlay Granola context** - Add meeting decisions and action items
3. **Enrich with Obsidian/Graphiti** - Add documentation and historical context
4. **Note Linear tracking** - Include as personal reflections only

### Step 2: Perform Gap Analysis

Compare data across sources to identify:

1. **Meeting commitments not in Azure DevOps** - Action items from meetings without corresponding work items
2. **Work items without recent context** - Azure DevOps items with no meeting discussion or documentation
3. **Conflicting information** - Discrepancies between sources (Azure DevOps wins)
4. **Stale items** - Work items with no recent activity

### Step 3: Generate Prioritized Recommendations

Based on synthesis, recommend:

1. **Focus areas for today** - Highest impact work
2. **Blockers to address** - What's preventing progress
3. **Meetings to prepare for** - Upcoming discussions requiring prep
4. **Updates to make** - Azure DevOps items needing status changes

## Output Format

Generate the final briefing in this exact structure:

```markdown
# Evonik Morning Brief - [Today's Date]

## 📋 Active Work Items (Azure DevOps)

| ID | Title | Status | Priority | Recent Activity |
|----|-------|--------|----------|-----------------|
| [id] | [title] | [state] | [priority] | [last update] |

### ⚠️ Blockers & Dependencies
- [blocker with context and suggested resolution]

---

## 📅 Recent Meeting Context (Granola)

### Key Decisions (Last 7 Days)
- **[Decision]** - [Meeting Date] - [Context]

### 🎯 Action Items Assigned to Me
- [ ] [Action item] - Due: [date if specified]
- [ ] [Action item] - From: [meeting name]

### Commitments Made
- [Commitment] - To: [person/team]

### Upcoming Meetings
| Meeting | Date/Time | Preparation Needed |
|---------|-----------|-------------------|
| [name] | [datetime] | [prep notes] |

---

## 📚 Project Documentation (Obsidian/Graphiti)

### Relevant Notes
- [[Note Title]] - [Brief relevance summary]

### Historical Context
- [Relevant fact or past decision]
- [Pattern identified from history]

---

## 📝 Personal Tracking (Linear)
*Note: Personal reflection only - not source of truth*

| Issue | Status | My Notes |
|-------|--------|----------|
| [issue] | [status] | [personal context] |

---

## 🔍 Gap Analysis

### Meeting Commitments Without Work Items
- ⚠️ [Commitment from meeting with no ADO work item]

### Work Items Lacking Recent Context
- ⚠️ [Work item with no recent meeting/documentation]

### Potential Conflicts
- ⚠️ [Any discrepancy between sources]

---

## 🎯 Recommended Focus Today

### Priority 1: [Title]
**Why**: [Reasoning based on synthesis]
**First Step**: [Specific action]

### Priority 2: [Title]
**Why**: [Reasoning]
**First Step**: [Action]

### Priority 3: [Title]
**Why**: [Reasoning]
**First Step**: [Action]

### Blockers to Address
1. [Blocker] → [Suggested resolution]

### Meetings to Prepare For
- [Meeting] at [time] - Need to: [preparation]

---

*Generated: [timestamp]*
*Sources: Azure DevOps (CS Enterprise AI), Granola, Obsidian, Graphiti, Linear*
```

## Architecture

```
/evonik-brief (command)
       │
       ▼
┌─────────────────────────────────────┐
│  Main Claude (spawns directly)      │
│  - Spawns all 4 gatherers in parallel│
│  - Collects results with TaskOutput │
│  - Synthesizes following hierarchy  │
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

| Scenario | Handling |
|----------|----------|
| Azure DevOps agent fails | **CRITICAL** - Report error prominently, cannot generate reliable brief |
| Granola agent fails | Continue, note "Meeting context unavailable" |
| Context agent fails | Continue, note "Documentation/memory context unavailable" |
| Linear agent fails | Continue, note "Personal tracking unavailable" |
| Agent returns empty data | Include section with "No data found" note |

## Related Commands

- `/life-brief` - Personal life briefing
- `/rs42-brief` - RS42 startup projects briefing
- `/morning-brief` - Cross-project synthesis briefing
- `/project-brief [name]` - Deep dive into specific project

## Source of Truth Note

For Evonik employment work:
- **Azure DevOps (CS Enterprise AI)** is the official source of work assignments
- **Granola meetings** capture decisions and commitments
- **Linear** is for personal tracking/reflection only - never treat as source of truth
