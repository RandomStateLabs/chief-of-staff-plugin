# Chief of Staff Plugin

**Your AI Chief of Staff for Daily Briefings and Project Clarity**

A Claude Code plugin that provides intelligent daily briefings by synthesizing your Obsidian notes, Linear tasks, and Graphiti memory to give you comprehensive project status and momentum insights.

---

## 🎯 What This Plugin Does

This plugin acts as your AI Chief of Staff, helping you:

- **Start your day with clarity**: Morning briefs that synthesize overnight context
- **Close your day efficiently**: Evening syncs that propose Linear updates based on actual work
- **Deep dive into projects**: Comprehensive project briefs with cross-system analysis
- **Build institutional memory**: Automatic storage of insights in Graphiti knowledge graph

### The Killer Feature: Evening Sync

The `/evening-sync` command analyzes your Obsidian notes from today, compares with Linear task status, and **proposes updates** (mark complete, mark blocked, create new tasks) based on actual work done. You approve the batch, and your Linear workspace stays in sync effortlessly.

---

## 📊 Development Status

**Status**: 🚧 **Template Scaffold Complete** - Ready for customization 🚧

### ✅ What's Complete

- [x] Plugin manifest and structure
- [x] 3 specialized agents (morning-planner, evening-reviewer, project-analyst)
- [x] 3 commands (/morning-brief, /evening-sync, /project-brief)
- [x] 3 skills (obsidian-reader, linear-integration, graphiti-memory)
- [x] Progressive disclosure pattern implemented
- [x] Batch approval pattern for Linear writes
- [x] Comprehensive documentation

### 🚧 What You Need to Customize

- [ ] Test with your Obsidian vault structure
- [ ] Verify Linear workspace integration
- [ ] Configure Graphiti group_ids for projects
- [ ] Adjust briefing output formats to your preferences
- [ ] Test with 1-2 projects first
- [ ] Iterate on synthesis prompts based on your needs

---

## 📂 Plugin Structure

```
plugins/chief-of-staff/
├── .claude-plugin/
│   └── plugin.json                    # Plugin manifest
├── agents/
│   ├── morning-planner.md             # Morning brief specialist
│   ├── evening-reviewer.md            # Evening sync specialist
│   └── project-analyst.md             # Project deep-dive specialist
├── commands/
│   ├── morning-brief.md               # /morning-brief command
│   ├── evening-sync.md                # /evening-sync command ⭐
│   └── project-brief.md               # /project-brief [name] command
├── skills/
│   ├── obsidian-reader/
│   │   ├── SKILL.md                   # Obsidian access patterns
│   │   └── references/
│   ├── linear-integration/
│   │   ├── SKILL.md                   # Linear read/write patterns
│   │   └── references/
│   │       └── batch-update-patterns.md
│   └── graphiti-memory/
│       ├── SKILL.md                   # Knowledge graph patterns
│       └── references/
└── README.md                          # This file
```

---

## 🚀 Setup Instructions

### Prerequisites

1. **Claude Code** installed globally
2. **MCP Servers** configured:
   - Obsidian MCP (via Docker)
   - Linear MCP (via Docker)
   - Graphiti MCP (via Docker)

### Step 1: Verify MCP Servers

```bash
# Check your MCP configuration
cat ~/.claude/settings/mcpServers.json

# Should include:
# - Obsidian MCP (pointing to your vault)
# - Linear MCP (with API key)
# - Graphiti MCP (with database connection)
```

If not configured, see MCP setup guides:
- [Obsidian MCP Setup](https://github.com/anthropics/claude-code/docs/mcp/obsidian)
- [Linear MCP Setup](https://github.com/anthropics/claude-code/docs/mcp/linear)
- [Graphiti MCP Setup](https://github.com/anthropics/claude-code/docs/mcp/graphiti)

### Step 2: Test MCP Connections

```bash
# Start Claude Code
claude code

# Test each MCP server:
# Obsidian
mcp__MCP_DOCKER__obsidian_list_files_in_vault

# Linear
mcp__linear__list_teams

# Graphiti
mcp__graphiti__get_status
```

If all work, MCP servers are configured correctly!

### Step 3: Install the Plugin

```bash
# Navigate to the ai-assistant-plugin directory
cd ~/RS42/claude-code-plugins/ai-assistant-plugin

# Start Claude Code
claude code

# The plugin will auto-load from plugins/chief-of-staff/
# Verify plugin loaded - you should see commands and skills available
```

### Step 4: Test with Simple Command

```bash
# In Claude Code, try:
/morning-brief

# This will:
# 1. Get today's journal from Obsidian (if exists)
# 2. Get recent notes from last 48h
# 3. Get in-progress Linear tasks
# 4. Generate morning briefing
```

---

## 📖 Usage Guide

### Quick Start: Three Core Commands

#### 1. Morning Brief - Start Your Day

```bash
/morning-brief
```

**What it does**:
- Gets today's daily journal from Obsidian
- Shows notes modified in last 48 hours
- Lists all "In Progress" Linear tasks
- Synthesizes project status and momentum
- Provides priorities for the day

**Example Output**:
```markdown
# Morning Brief - 2025-11-23

## 📊 Project Status Review
- Auth Service: Implementation phase, 60% complete, on track
- Mobile App: Design review scheduled, waiting on stakeholder feedback

## 📔 Today's Journal
No journal entry yet - consider starting one!

## 📝 Recent Activity (48h)
- [[Auth Service Implementation]] - Core features complete
- [[API Design Doc]] - Finalized endpoints schema

## ✅ Active Work
- [PROJ-123] Implement auth service - 60% complete
- [PROJ-124] Review API design - Pending feedback

## 🎯 What This Means
**Momentum**: Auth service progressing well
**Blockers**: API design waiting on stakeholder
**Priorities**: Focus on auth testing, follow up on API review
```

#### 2. Evening Sync - Close Your Day ⭐

```bash
/evening-sync
```

**What it does**:
- Reviews today's notes from Obsidian
- Compares with Linear task status
- **Proposes Linear updates** based on actual work done
- Waits for your approval
- Executes approved updates
- Stores insights in Graphiti

**Example Output**:
```markdown
# Evening Sync - 2025-11-23

## ✅ What Got Done Today
- Completed auth service implementation
- Discovered token expiry bug during testing
- Finalized API design review

## 📝 Proposed Linear Updates

### Tasks to Mark Complete
- [ ] [PROJ-123] "Implement auth service"
      Evidence: "Auth Service Implementation.md" shows completion
- [ ] [PROJ-124] "Review API design"
      Evidence: "API Design Review.md" finalized

### New Tasks to Create
- [ ] "Fix auth token expiry bug"
      Source: "Testing Session.md" - Critical bug found

**Review these changes. Would you like me to apply them to Linear?**
```

You say: "Yes, apply them"

```markdown
✅ **Linear Updates Applied**
- PROJ-123 → Done
- PROJ-124 → Done
- Created PROJ-456: Fix auth token expiry bug

Your Linear workspace is now in sync!
```

#### 3. Project Brief - Deep Dive

```bash
/project-brief "Auth Service"
```

**What it does**:
- Searches all Obsidian notes related to project
- Gets all Linear issues for the project (all states)
- Queries Graphiti for project history and decisions
- Builds comprehensive timeline
- Analyzes health, velocity, blockers
- Provides actionable recommendations

**Example Output**:
```markdown
# Project Brief: Auth Service

## 📊 Current Status
**Health**: 🟢 On Track
**Last Updated**: 2025-11-23

## 🎯 Project Overview
Authentication service for mobile and web apps using JWT tokens.
Lead: Alice | Team: Engineering | Timeline: 6 weeks

## 📈 Progress Summary
- Total Tasks: 15
- Completed: 8 (53%)
- In Progress: 5
- Blocked: 1
- Not Started: 1

## 🗂️ Key Resources
### Documentation (Obsidian)
- [[Auth Service Architecture]] - System design, last updated 2025-11-20
- [[API Endpoints Spec]] - Complete API reference, last updated 2025-11-22

### Active Work (Linear)
- [PROJ-125] Deploy staging - BLOCKED by infra team
- [PROJ-126] Integration tests - 40% complete

### Historical Context (Graphiti)
- 2025-11-01: Decided on JWT vs session-based auth
- 2025-11-15: Architecture review approved
- 2025-11-20: Discovered token expiry issue, now resolved

## 💡 Insights & Analysis
- Velocity: Consistent progress, completing ~2 tasks/week
- Risk: Dependency on infra team may delay deployment
- Opportunity: Can parallelize testing and documentation

## 🎯 Recommended Next Steps
1. Follow up on staging deployment blocker (High priority)
2. Complete integration tests (Target: end of week)
3. Begin production deployment planning
```

---

## 🎨 Customization

### Adjusting Briefing Formats

Edit the agent files to customize output:

```bash
# Modify morning brief structure
open plugins/chief-of-staff/agents/morning-planner.md

# Modify evening sync format
open plugins/chief-of-staff/agents/evening-reviewer.md

# Modify project brief sections
open plugins/chief-of-staff/agents/project-analyst.md
```

### Configuring Group IDs

For project-specific memory in Graphiti, use group_ids:

```markdown
# In evening sync, store with project group
mcp__graphiti__add_memory(
    name="Auth Service Update",
    episode_body="...",
    group_id="auth-service-project"
)
```

### Adding Optional Parameters

Commands can accept parameters:

```bash
/morning-brief --project "Auth"     # Focus on specific project
/evening-sync --no-linear-updates   # Skip Linear proposals
/project-brief "Auth" --health      # Focus on health assessment
```

Edit command files to add parameter handling.

---

## 🛠️ Troubleshooting

### Plugin Not Loading

```bash
# Check directory structure
ls -la plugins/chief-of-staff/.claude-plugin/
cat plugins/chief-of-staff/.claude-plugin/plugin.json

# Restart Claude Code in plugin directory
cd ~/RS42/claude-code-plugins/ai-assistant-plugin
claude code
```

### MCP Servers Not Working

```bash
# Test each server
# Obsidian
mcp__MCP_DOCKER__obsidian_list_files_in_vault

# Linear
mcp__linear__list_teams

# Graphiti
mcp__graphiti__get_status

# Check Docker
docker ps
```

### Commands Not Found

```bash
# Verify plugin loaded
# You should see commands listed when typing /

# Check command files exist
ls plugins/chief-of-staff/commands/
```

### Linear Updates Not Working

Check the batch approval pattern:
1. Are proposed updates being shown?
2. Did you explicitly approve?
3. Check Linear permissions (can you update issues manually?)
4. Check API key in MCP config

---

## 💡 Best Practices

### Daily Workflow

**Morning**:
```bash
1. Open Claude Code
2. Run /morning-brief
3. Review project status and priorities
4. Start working on top priority items
```

**Evening**:
```bash
1. Review today's work in notes
2. Run /evening-sync
3. Review proposed Linear updates
4. Approve updates to sync Linear
5. Reflect on insights captured
```

**Weekly**:
```bash
1. Run /project-brief for each active project
2. Review health assessments
3. Address blockers
4. Adjust priorities
```

### Building Graphiti Memory

The more consistently you use `/evening-sync` to store daily insights, the more valuable Graphiti becomes:

- **Week 1**: Basic facts, starting to populate
- **Month 1**: Patterns emerging, decision history building
- **Quarter 1**: Rich context, cross-project insights visible
- **Year 1**: Deep institutional knowledge, trend analysis possible

### Safety with Linear Writes

The **batch approval pattern** is critical:

✅ **DO**:
- Review all proposed changes before approving
- Check evidence makes sense
- Approve in batches (efficient)

❌ **DON'T**:
- Auto-approve without reading
- Skip checking evidence
- Approve if unsure

---

## 📚 Architecture Overview

### Three-Layer Design

This plugin demonstrates Claude Code's three-layer architecture:

**Layer 1: Skills** (Auto-activated, ~70 tokens)
- `obsidian-reader` - loads when Obsidian access needed
- `linear-integration` - loads when Linear access needed
- `graphiti-memory` - loads when memory access needed
- Progressive disclosure: skills describe operations without loading full MCP tools

**Layer 2: Commands** (User-triggered entry points)
- `/morning-brief` - sets morning planning mode
- `/evening-sync` - sets evening review mode
- `/project-brief` - sets project analysis mode
- Commands orchestrate multiple skills together

**Layer 3: Agents** (Specialized subprocesses with 200K context)
- `morning-planner` - dedicated context for morning synthesis
- `evening-reviewer` - dedicated context for evening analysis
- `project-analyst` - dedicated context for deep project dives
- Spawn when complex multi-step analysis needed

### Progressive Disclosure Pattern

**Token Efficiency**:
- Skill description loaded: ~70 tokens
- Full MCP tools loaded: ~4,200 tokens
- Result: 98% reduction when MCP not needed

**How it works**:
1. User asks question
2. Skill loads (just description)
3. If MCP needed, tools load on demand
4. If just guidance needed, tools never load

### Cross-System Synthesis

The plugin's value comes from connecting three systems:

```
Obsidian (Documentation) + Linear (Tasks) + Graphiti (Memory)
                           ↓
            Comprehensive Project Understanding
```

Example:
- Obsidian: "Auth Service Implementation.md" shows work complete
- Linear: PROJ-123 still marked "In Progress"
- Graphiti: Stores decision that JWT was chosen over sessions
- **Synthesis**: Update Linear task to Done, reference decision history in report

---

## 🔗 Related Documentation

- **PRD**: See `prd.md` for complete product requirements
- **Development Notes**: See `notes.md` for brainstorming history
- **MCP Guides**: Check Claude Code docs for MCP server setup
- **Plugin Architecture**: See Obsidian notes for architecture patterns

---

## ✅ Final Checklist

Before using this plugin in production:

- [ ] Configured Obsidian MCP pointing to your vault
- [ ] Configured Linear MCP with valid API key
- [ ] Configured Graphiti MCP with database connection
- [ ] Tested `/morning-brief` command successfully
- [ ] Tested `/evening-sync` command (reviewed batch approval)
- [ ] Tested `/project-brief [name]` for one project
- [ ] Customized agent output formats if needed
- [ ] Set up Graphiti group_ids for project organization
- [ ] Verified Linear write permissions work
- [ ] Read batch-update-patterns.md for safety guidelines

---

## 🏆 Why This Plugin?

**Based on Claude Code's extensibility patterns**:

✅ **Three-layer architecture** - Skills, Commands, Agents working together
✅ **Progressive disclosure** - 98% token efficiency when MCP not needed
✅ **MCP integration** - Obsidian + Linear + Graphiti working in harmony
✅ **Safety first** - Batch approval pattern for Linear writes
✅ **Real value** - Cross-system synthesis that saves hours of manual work

**Transform your workflow**:
- From: Manually checking notes, Linear, remembering context
- To: AI Chief of Staff that synthesizes everything and keeps systems in sync

---

**Version**: 0.1.0 (Template Scaffold)
**Status**: 🚧 Ready for customization and testing
**License**: MIT
**Author**: RandomStateLabs

**Next Steps**: Test with your data, iterate on synthesis quality, build that Graphiti memory!
