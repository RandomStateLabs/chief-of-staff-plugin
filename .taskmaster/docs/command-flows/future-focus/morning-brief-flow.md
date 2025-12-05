# /morning-brief Command Flow Trace

> **Purpose**: Daily status synthesis across ALL projects. Provides a comprehensive briefing to start the day.

---

## Overview

| Property | Value |
|----------|-------|
| **Command** | `/morning-brief` |
| **Spawns Agent** | Yes - `morning-planner` |
| **Complexity** | High (cross-project synthesis, multiple systems) |
| **Write Operations** | None (read-only briefing) |

| Feature                  | Obsidian Read | Obsidian Write | Linear Read | Linear Write | Graphiti Read | Graphiti Write | GitHub Read | GitHub Write | Git Read |
| -------------------------- | --------------- | ---------------- | ------------- | -------------- | --------------- | ---------------- | ------------- | -------------- | ---------- |
| `/morning-brief`         | ✅            |                | ✅          |              | ✅            |                | ✅          |              | ✅       |

---

## Flow Diagram

```
User: /morning-brief
         │
         ▼
┌─────────────────────────────────────────┐
│  COMMAND: morning-brief.md              │
│                                         │
│  1. Load skill: project-context         │
│  2. Get current date (Bash date:*)      │
│  3. Spawn agent: morning-planner        │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  AGENT: morning-planner                 │
│  (200K context)                         │
│                                         │
│  Activated Skills:                      │
│  - obsidian-reader                      │
│  - linear-integration (read only)       │
│  - graphiti-memory (read only)          │
│  - github-integration (read only)       │
│  - git-operations (read only)           │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 1: Gather Obsidian Context       │
│                                         │
│  Tool: obsidian_get_periodic_note       │
│  Params: {period: "daily"}              │
│  → Get today's journal (if exists)      │
│                                         │
│  Tool: obsidian_get_recent_changes      │
│  Params: {days: 2, limit: 20}           │
│  → Get notes modified in last 48h       │
│                                         │
│  Tool: obsidian_simple_search           │
│  Params: {query: "project status"}      │
│  → Find project-related notes           │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 2: Gather Linear Context         │
│                                         │
│  Tool: mcp__linear__list_issues         │
│  Params: {team: "RS42", status: "in-progress"}
│  → Get all in-progress tasks            │
│                                         │
│  Tool: mcp__linear__list_issues         │
│  Params: {team: "RS42", status: "blocked"}
│  → Get blocked tasks                    │
│                                         │
│  Tool: mcp__linear__list_projects       │
│  Params: {}                             │
│  → Get all active projects              │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 3: Gather Graphiti Context       │
│                                         │
│  Tool: mcp__graphiti__search_memory_facts
│  Params: {                              │
│    query: "decisions blockers progress",│
│    group_ids: ["work"],                 │
│    max_facts: 20                        │
│  }                                      │
│  → Get cross-project facts              │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 4: Gather GitHub Context         │
│  (Remote repository state)              │
│                                         │
│  For each active project:               │
│                                         │
│  Tool: mcp__MCP_DOCKER__list_pull_requests
│  Params: {owner: "...", repo: "...",    │
│           state: "open"}                │
│  → Get open PRs                         │
│                                         │
│  Tool: mcp__MCP_DOCKER__list_commits    │
│  Params: {owner: "...", repo: "...",    │
│           perPage: 5}                   │
│  → Get recent commits                   │
│                                         │
│  Tool: mcp__MCP_DOCKER__get_pull_request_status
│  → Check CI/CD status                   │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 5: Gather Local Git Context      │
│                                         │
│  For each project directory:            │
│                                         │
│  Tool: Bash(git status:*)               │
│  → Check for uncommitted changes        │
│                                         │
│  Tool: Bash(git log:*)                  │
│  Params: --oneline -5                   │
│  → Recent local commits                 │
│                                         │
│  Tool: Bash(git diff:*)                 │
│  Params: --stat                         │
│  → Pending changes summary              │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 6: Synthesis                     │
│                                         │
│  Cross-reference all gathered data:     │
│  - Notes ↔ Linear tasks                 │
│  - Decisions ↔ Progress                 │
│  - Blockers across projects             │
│  - Open PRs needing attention           │
│                                         │
│  Generate briefing:                     │
│  - Projects with activity               │
│  - Blockers and priorities              │
│  - Pending PRs/reviews                  │
│  - Suggested focus for today            │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  OUTPUT: Morning Briefing               │
│                                         │
│  ## Morning Briefing - [Date]           │
│                                         │
│  ### Active Projects                    │
│  - Project A: [status summary]          │
│  - Project B: [status summary]          │
│                                         │
│  ### Blockers                           │
│  - [blocker 1]                          │
│  - [blocker 2]                          │
│                                         │
│  ### Open PRs Needing Attention         │
│  - [PR link]: [status]                  │
│                                         │
│  ### Suggested Focus Today              │
│  1. [priority item]                     │
│  2. [priority item]                     │
│                                         │
│  ### Notes from Yesterday               │
│  - [relevant note summary]              │
└─────────────────────────────────────────┘
```

---

## Tool Calls Summary

### Read Operations (All Auto-Allowed)

| Tool | Purpose | Parameters |
|------|---------|------------|
| `Bash(date:*)` | Get current date | - |
| `mcp__MCP_DOCKER__obsidian_get_periodic_note` | Today's journal | `{period: "daily"}` |
| `mcp__MCP_DOCKER__obsidian_get_recent_changes` | Recent notes | `{days: 2, limit: 20}` |
| `mcp__MCP_DOCKER__obsidian_simple_search` | Project notes | `{query: "..."}` |
| `mcp__linear__list_issues` | In-progress tasks | `{team: "RS42", status: "..."}` |
| `mcp__linear__list_projects` | Active projects | `{}` |
| `mcp__graphiti__search_memory_facts` | Cross-project facts | `{group_ids: ["work"]}` |
| `mcp__MCP_DOCKER__list_pull_requests` | Open PRs | `{state: "open"}` |
| `mcp__MCP_DOCKER__list_commits` | Recent commits | `{perPage: 5}` |
| `mcp__MCP_DOCKER__get_pull_request_status` | CI/CD status | - |
| `Bash(git status:*)` | Local uncommitted | - |
| `Bash(git log:*)` | Local commits | `--oneline -5` |
| `Bash(git diff:*)` | Pending changes | `--stat` |

### Write Operations

**None** - This is a read-only briefing command.

---

## Skills Required

| Skill | Why Needed |
|-------|------------|
| `obsidian-reader` | Read journal and recent notes |
| `linear-integration` | Read issues and projects |
| `graphiti-memory` | Query cross-project facts |
| `github-integration` | Read remote repo state (PRs, commits, CI) |
| `git-operations` | Check local repo state |
| `project-context` | Detect project directories |

---

## Error Handling

| Scenario | Handling |
|----------|----------|
| No journal for today | Note absence, continue with other sources |
| Linear API failure | Report error, continue with available data |
| GitHub rate limit | Report, suggest manual check |
| No active projects | Report empty state |
| Graphiti unreachable | Continue without memory context |

---

## Future Enhancements

1. **Calendar Integration**: Add meeting context for the day
2. **Email/Slack Summary**: Communication requiring response
3. **Trend Analysis**: Compare with previous briefings
4. **Focus Mode**: Filter to single project with `--project` flag

---

## Example Output

```markdown
## Morning Briefing - December 4, 2025

### Active Projects (3)

**PE Evaluation** (High Priority)
- Status: 2 tasks in-progress, 1 blocked
- Last commit: 2h ago - "Fix edge case in auth flow"
- Open PR: #42 awaiting review (CI passing)
- Blocker: Waiting on API spec from external team

**Chief of Staff Plugin** (Medium Priority)
- Status: 3 tasks in-progress
- Last commit: Yesterday - "Add architecture docs"
- Notes: "Architecture overview created" (yesterday)

**Claude Code Plugins** (Low Priority)
- Status: Maintenance mode
- No recent activity

### Blockers Requiring Attention
1. PE Eval: API spec from external team (3 days waiting)
2. Chief of Staff: Graphiti hierarchical groups untested

### Open PRs
- [pe-eval#42](link): Auth flow fix - Awaiting review
- [chief-of-staff#15](link): Morning brief - Draft

### Suggested Focus Today
1. Review and merge PE Eval PR #42
2. Follow up on external API spec
3. Continue Chief of Staff architecture docs

### Recent Notes
- "Architecture overview for Chief of Staff" - captures skill design decisions
- "PE Eval blocking issue" - documents the external dependency
```
