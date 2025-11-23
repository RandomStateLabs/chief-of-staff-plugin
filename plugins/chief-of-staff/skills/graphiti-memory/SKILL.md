---
name: graphiti-memory
description: Access and store knowledge in Graphiti temporal knowledge graph including project facts, decision history, and insights. Use when you need to query past context, store learnings, or understand relationships between projects and entities.
---

# Graphiti Memory Skill

This skill helps you interact with the Graphiti temporal knowledge graph - a dynamic memory system that stores facts, entities, and relationships with temporal metadata. Use this for retrieving historical context and storing insights for future reference.

## When to Use This Skill

Activate this skill when:
- You need historical context about projects or decisions
- User asks "what did we decide about X?"
- Storing insights from evening reflections
- Understanding relationships between projects/entities
- Finding patterns across time
- Enriching briefs with past context

## Core Workflow

### 1. Determine Operation Type

**QUERY Operations** (retrieve from memory):
- Search for facts about a topic
- Find entities (projects, people, concepts)
- Get recent episodes (memories)
- Understand historical context

**STORE Operations** (add to memory):
- Add insights from daily work
- Store decision history
- Capture learnings and reflections
- Record project milestones

### 2. Choose the Right Tool

**Quick Reference** (most common operations):

| Need | Tool | Operation |
|------|------|-----------|
| Find facts about topic | `search_memory_facts` | Query |
| Find entities/nodes | `search_nodes` | Query |
| Get recent memories | `get_episodes` | Query |
| Store insight/learning | `add_memory` | Store |
| Store structured data | `add_memory` (JSON) | Store |

## Quick Patterns (QUERY Operations)

### Pattern 1: Search for Project Facts

```
# Search for facts about a project
mcp__graphiti__search_memory_facts(
    query="Auth Service decisions OR blockers OR milestones",
    max_facts=10
)

# Returns: Relevant facts with temporal metadata
# - Fact content (relationship between entities)
# - When the fact was created
# - Whether fact is still valid (or invalidated by newer info)
# - Related entities

# Use for: Project briefs, understanding history
```

### Pattern 2: Find Related Entities

```
# Search for entities (projects, people, concepts)
mcp__graphiti__search_nodes(
    query="Auth Service",
    max_nodes=5
)

# Returns: Entity nodes with:
# - Entity name and type
# - Summary of the entity
# - Related facts

# Use for: Understanding project relationships, finding connections
```

### Pattern 3: Get Recent Episodes

```
# Get recent memories/episodes
mcp__graphiti__get_episodes(
    group_ids=["default"],  # Or specific project group
    max_episodes=10
)

# Returns: Recent episodes (memory snapshots) with:
# - Episode name and content
# - When it was added
# - Source type (text, message, JSON)

# Use for: Reviewing recent work, understanding context flow
```

### Pattern 4: Search with Filtering

```
# Search facts for specific group/project
mcp__graphiti__search_memory_facts(
    query="deployment blockers",
    group_ids=["auth-service-project"],
    max_facts=15
)

# Search nodes by entity type
mcp__graphiti__search_nodes(
    query="engineering team",
    entity_types=["person", "team"],
    max_nodes=8
)

# Returns: Filtered results
# Use for: Focused queries, reducing noise
```

## STORE Operations (Adding to Memory)

### Pattern 5: Store Text Insight

```
# Store learning or insight from today
mcp__graphiti__add_memory(
    name="Evening Reflection - 2025-11-23",
    episode_body="Completed auth service implementation. Discovered token expiry bug during testing. Team velocity good this week. Blocker on staging deployment resolved by infra team.",
    source="text",
    source_description="Daily reflection",
    group_id="default"
)

# Graphiti will:
# - Extract entities (auth service, team, infra team)
# - Create facts (relationships)
# - Add temporal metadata
# - Make queryable for future

# Use for: Evening syncs, capturing learnings
```

### Pattern 6: Store Structured Data

```
# Store structured project data as JSON
import json

project_data = {
    "project": {
        "name": "Auth Service",
        "status": "in_progress",
        "phase": "implementation"
    },
    "team": {
        "lead": "Alice",
        "members": ["Bob", "Carol"]
    },
    "milestones": [
        {"name": "Design Complete", "date": "2025-11-15"},
        {"name": "Implementation", "date": "2025-11-23"}
    ]
}

mcp__graphiti__add_memory(
    name="Auth Service Project Status - 2025-11-23",
    episode_body=json.dumps(project_data),
    source="json",
    source_description="Weekly project update"
)

# Graphiti automatically processes JSON to extract:
# - Entities from structured data
# - Relationships between entities
# - Temporal tracking

# Use for: Structured project updates, metrics, timelines
```

### Pattern 7: Store Decision History

```
# Capture important decisions for future reference
mcp__graphiti__add_memory(
    name="Architecture Decision - Auth Token Strategy",
    episode_body="Decided to use JWT tokens with 1-hour expiry and refresh tokens. Considered session-based auth but JWT provides better scalability. Trade-off: need to handle token refresh on client.",
    source="text",
    source_description="Architecture decision record",
    group_id="auth-service-project"
)

# Future queries like "why did we choose JWT?" will surface this
# Use for: Decision records, rationale preservation
```

## MCP Tools Available

### Query Tools

**`search_memory_facts`**
- Purpose: Search for facts (relationships between entities)
- Parameters:
  - `query`: Natural language search query (required)
  - `group_ids`: Filter by specific groups (optional)
  - `max_facts`: Max results to return (default 10)
  - `center_node_uuid`: Center search around specific entity (optional)
- Returns: Array of facts with:
  - Fact content (relationship description)
  - Temporal metadata (created date, validity)
  - Related entities
- Use for: Understanding context, finding decisions, historical queries

**`search_nodes`**
- Purpose: Search for entities/nodes in the graph
- Parameters:
  - `query`: Search query (required)
  - `group_ids`: Filter by groups (optional)
  - `max_nodes`: Max results (default 10)
  - `entity_types`: Filter by entity type like "person", "project" (optional)
- Returns: Array of entities with:
  - Entity name and type
  - Summary/description
  - Related facts
- Use for: Finding projects, people, concepts

**`get_episodes`**
- Purpose: Get recent memory episodes
- Parameters:
  - `group_ids`: Filter by groups (optional)
  - `max_episodes`: Max to return (default 10)
- Returns: Recent episodes with content and timestamps
- Use for: Reviewing recent activity, context flow

### Store Tools

**`add_memory`**
- Purpose: Add new memory/episode to graph
- Parameters:
  - `name`: Episode name/title (required)
  - `episode_body`: Content to store (required)
  - `source`: "text" | "json" | "message" (default "text")
  - `source_description`: Description of source (optional)
  - `group_id`: Group to store in (optional, defaults to global)
  - `uuid`: Optional UUID (auto-generated if not provided)
- Process: Graphiti automatically:
  - Extracts entities from content
  - Creates relationships (facts)
  - Adds temporal metadata
  - Makes queryable
- Use for: Storing insights, decisions, learnings, structured data

### Management Tools (Use Carefully)

**`delete_episode`**
- Purpose: Delete an episode
- Parameters:
  - `uuid`: Episode UUID
- ⚠️ Use sparingly - historical data is valuable

**`delete_entity_edge`**
- Purpose: Delete a fact/relationship
- Parameters:
  - `uuid`: Edge UUID
- ⚠️ Use sparingly

**`clear_graph`**
- Purpose: Clear all data for group(s)
- Parameters:
  - `group_ids`: Groups to clear (optional)
- ⚠️ DESTRUCTIVE - confirm with user first

**`get_status`**
- Purpose: Check Graphiti connection status
- Parameters: None
- Returns: Server and database status
- Use for: Debugging connection issues

## Common Workflows

### Morning Brief Workflow

1. Query for recent project context:
   ```
   mcp__graphiti__search_memory_facts(
       query="project status OR recent decisions",
       max_facts=10
   )
   ```

2. Process: Include historical context in brief, show decision continuity

### Evening Sync Workflow

1. Store today's insights:
   ```
   mcp__graphiti__add_memory(
       name="Evening Reflection - [Date]",
       episode_body="[Summary of day's work, learnings, blockers]",
       source="text"
   )
   ```

2. Store structured updates:
   ```
   mcp__graphiti__add_memory(
       name="Task Updates - [Date]",
       episode_body=json.dumps(task_data),
       source="json"
   )
   ```

### Project Brief Workflow

1. Search for all project context:
   ```
   mcp__graphiti__search_memory_facts(
       query="[Project Name] milestones OR decisions OR blockers",
       group_ids=["[project-group]"],
       max_facts=20
   )
   ```

2. Find related entities:
   ```
   mcp__graphiti__search_nodes(
       query="[Project Name]",
       max_nodes=10
   )
   ```

3. Get recent episodes:
   ```
   mcp__graphiti__get_episodes(
       group_ids=["[project-group]"],
       max_episodes=15
   )
   ```

4. Process: Build comprehensive timeline, identify patterns, show evolution

## Understanding Graphiti's Knowledge Graph

### Key Concepts

**Episodes**: Snapshots of information added to memory
- Text content, messages, or JSON data
- Timestamped memories
- Source of truth for what was captured

**Entities (Nodes)**: Things extracted from episodes
- Projects, people, concepts, systems
- Have types and summaries
- Connected by facts

**Facts (Edges)**: Relationships between entities
- "Auth Service uses JWT tokens"
- "Alice leads Engineering Team"
- "Staging deployment blocked by Infrastructure"
- Have temporal metadata (when created, when invalidated)

**Temporal Awareness**: Graphiti tracks time
- When facts were created
- When facts became invalid (superseded by new info)
- Query by time ranges
- Understand evolution of knowledge

### Why Use Graphiti?

1. **Memory Across Sessions**: Context persists beyond individual briefs
2. **Pattern Recognition**: See recurring themes, blockers, decisions
3. **Decision History**: Understand "why we did X"
4. **Relationship Mapping**: How projects/people/concepts connect
5. **Temporal Understanding**: How things changed over time
6. **Future Context**: Insights stored now help future briefs

## Best Practices

### For Queries
1. **Natural Language**: Graphiti handles conversational queries well
2. **Multiple Keywords**: Use OR for alternatives ("decisions OR milestones")
3. **Specific Groups**: Filter by group_id for focused results
4. **Reasonable Limits**: 10-20 facts usually sufficient
5. **Entity Types**: Filter by type when looking for specific categories

### For Storage
1. **Descriptive Names**: Use clear episode names with dates
2. **Rich Context**: Store enough detail for future understanding
3. **Regular Cadence**: Store insights daily (evening syncs)
4. **Structured When Appropriate**: Use JSON for metrics, timelines
5. **Group Organization**: Use group_ids to organize by project
6. **Decision Records**: Explicitly capture important decisions with rationale

### What to Store
✅ **DO Store**:
- Daily reflections and insights
- Decision rationale
- Project milestones and status
- Blockers and their resolutions
- Learnings and patterns
- Team interactions and context

❌ **DON'T Store**:
- Redundant data already in Obsidian/Linear
- Temporary scratch notes
- Sensitive/private information
- Overly detailed logs (store summaries instead)

## Advanced Features

For complex requirements, see:
- `references/graphiti-queries.md` - Advanced query patterns and examples
- `references/temporal-analysis.md` - Using temporal data for insights
- `references/knowledge-graph-patterns.md` - Effective graph usage patterns

## Integration with Other Skills

This skill commonly works with:
- **obsidian-reader** skill - store insights from notes, retrieve context for documentation
- **linear-integration** skill - store task context, retrieve decision history
- Commands that benefit from historical context (all briefing commands)

## Error Handling

- **No results found**: Normal - may not have stored relevant data yet
- **Connection error**: Check Graphiti MCP configuration
- **Group not found**: Group IDs are arbitrary strings, create on first use
- **Add memory fails**: Check episode_body format, especially for JSON source

## Token Optimization

- Use specific queries to reduce result size
- Set appropriate max_facts/max_nodes limits
- Filter by group_ids when possible
- Store summaries rather than full content from other sources
- Query before storing to avoid duplicates

## Building Graph Over Time

Graphiti becomes more valuable with use:

**Week 1**: Basic facts, starting to populate
**Month 1**: Patterns emerging, decision history building
**Quarter 1**: Rich context, cross-project insights visible
**Year 1**: Deep institutional knowledge, trend analysis possible

The more consistently you store insights (especially in evening syncs), the more powerful the memory system becomes for future briefs and decision-making.
