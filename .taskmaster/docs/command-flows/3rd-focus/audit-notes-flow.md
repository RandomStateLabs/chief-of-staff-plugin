# /audit-notes Command Flow Trace

> **Purpose**: Reviews project notes before syncing to Obsidian vault. Checks atomicity (single idea per note), detects duplicates via semantic similarity, suggests placement (folder, tags, wikilinks), then executes moves after batch approval.

---

## Overview

| Property | Value |
|----------|-------|
| **Command** | `/audit-notes` |
| **Spawns Agent** | Yes - `note-auditor` |
| **Complexity** | High (semantic analysis, batch operations) |
| **Write Operations** | Obsidian vault writes (batch approval required) |

| Feature                  | Obsidian Read | Obsidian Write | Linear Read | Linear Write | Graphiti Read | Graphiti Write | GitHub Read | GitHub Write | Git Read |
| -------------------------- | --------------- | ---------------- | ------------- | -------------- | --------------- | ---------------- | ------------- | -------------- | ---------- |
| `/audit-notes`           | ✅            | ✅             |             |              |               | ✅             |             |              |          |

---

## Flow Diagram

```
User: /audit-notes
         │
         ▼
┌─────────────────────────────────────────┐
│  COMMAND: audit-notes.md                │
│                                         │
│  1. Load skill: project-context         │
│  2. Detect current project              │
│  3. Locate local notes directory        │
│  4. Spawn agent: note-auditor           │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  AGENT: note-auditor                    │
│  (200K context)                         │
│                                         │
│  Activated Skills:                      │
│  - obsidian-reader                      │
│  - obsidian-writer                      │
│  - graphiti-memory                      │
│  - semantic-analysis (proposed)         │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 1: Discover Local Notes          │
│                                         │
│  Tool: Bash(ls:*)                       │
│  Params: ./notes/                       │
│  → List all notes in project folder     │
│                                         │
│  For each note file:                    │
│  Tool: Read                             │
│  → Load full content                    │
│                                         │
│  Build notes inventory:                 │
│  [                                      │
│    {                                    │
│      file: "auth-decision.md",          │
│      title: "Auth Decision: JWT",       │
│      content: "...",                    │
│      word_count: 250,                   │
│      headings: 2,                       │
│      created: "2025-12-04"              │
│    },                                   │
│    ...                                  │
│  ]                                      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 2: Atomicity Check               │
│                                         │
│  For each note, evaluate:               │
│                                         │
│  Atomic indicators (PASS):              │
│  - Single main topic                    │
│  - Word count < 500                     │
│  - 1-2 headings max                     │
│  - Cohesive content flow                │
│                                         │
│  Non-atomic indicators (SPLIT):         │
│  - Multiple distinct topics             │
│  - Word count > 800                     │
│  - Many unrelated headings              │
│  - Natural split points                 │
│                                         │
│  Generate atomicity report:             │
│  {                                      │
│    "auth-decision.md": {                │
│      status: "atomic",                  │
│      reason: "Single decision topic"    │
│    },                                   │
│    "project-notes.md": {                │
│      status: "split-recommended",       │
│      reason: "Contains 3 distinct topics│
│      suggested_splits: [                │
│        "architecture-choice.md",        │
│        "deployment-plan.md",            │
│        "timeline-estimate.md"           │
│      ]                                  │
│    }                                    │
│  }                                      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 3: Duplicate Detection           │
│                                         │
│  Search Obsidian vault for similar:     │
│                                         │
│  For each local note:                   │
│                                         │
│  Tool: obsidian_simple_search           │
│  Params: {query: "[note title]"}        │
│  → Find vault notes with similar titles │
│                                         │
│  Tool: obsidian_simple_search           │
│  Params: {query: "[key phrases]"}       │
│  → Find notes with similar content      │
│                                         │
│  For potential matches:                 │
│  Tool: obsidian_get_file_contents       │
│  → Read full content for comparison     │
│                                         │
│  Semantic similarity check:             │
│  - Extract key concepts from both       │
│  - Compare topic coverage               │
│  - Check for overlapping content        │
│                                         │
│  Generate duplicate report:             │
│  {                                      │
│    "auth-decision.md": {                │
│      duplicates: [],                    │
│      similar: [                         │
│        {                                │
│          file: "Projects/Auth Design.md"│
│          similarity: 0.65,              │
│          recommendation: "link"         │
│        }                                │
│      ]                                  │
│    },                                   │
│    "api-notes.md": {                    │
│      duplicates: [                      │
│        {                                │
│          file: "Projects/API Notes.md", │
│          similarity: 0.92,              │
│          recommendation: "merge"        │
│        }                                │
│      ]                                  │
│    }                                    │
│  }                                      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 4: Placement Suggestions         │
│                                         │
│  For each note, determine:              │
│                                         │
│  FOLDER:                                │
│  - Match to vault folder structure      │
│  - Project notes → Projects/[name]/     │
│  - Decisions → Decisions/               │
│  - Reference → Reference/               │
│                                         │
│  TAGS:                                  │
│  - Extract key concepts                 │
│  - Match to existing vault tags         │
│  - Suggest new tags if needed           │
│                                         │
│  WIKILINKS:                             │
│  - Identify linkable concepts           │
│  - Find existing notes to link to       │
│  - Suggest bidirectional links          │
│                                         │
│  Generate placement suggestions:        │
│  {                                      │
│    "auth-decision.md": {                │
│      folder: "Projects/PE Evaluation/", │
│      tags: ["decision", "auth", "jwt"], │
│      wikilinks: [                       │
│        "[[PE Evaluation]]",             │
│        "[[Authentication]]"             │
│      ],                                 │
│      backlinks_to_add: [                │
│        "PE Evaluation.md" → add link    │
│      ]                                  │
│    }                                    │
│  }                                      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 5: Generate Audit Report         │
│  ═══════════════════════════════════════│
│  BATCH APPROVAL PATTERN                 │
│  ═══════════════════════════════════════│
│                                         │
│  ## Note Audit Report                   │
│                                         │
│  **Project**: PE Evaluation             │
│  **Notes Found**: 5                     │
│  **Issues Detected**: 2                 │
│                                         │
│  ### Atomicity Issues                   │
│                                         │
│  - [ ] **project-notes.md** - SPLIT     │
│        Contains 3 topics. Suggested:    │
│        1. architecture-choice.md        │
│        2. deployment-plan.md            │
│        3. timeline-estimate.md          │
│                                         │
│  ### Duplicates Found                   │
│                                         │
│  - [ ] **api-notes.md** - MERGE         │
│        92% similar to existing:         │
│        Projects/API Notes.md            │
│        Action: Merge into existing      │
│                                         │
│  ### Ready to Sync (4 notes)            │
│                                         │
│  - [ ] **auth-decision.md**             │
│        → Projects/PE Evaluation/        │
│        Tags: #decision #auth #jwt       │
│        Links: [[PE Evaluation]]         │
│                                         │
│  - [ ] **db-choice.md**                 │
│        → Projects/PE Evaluation/        │
│        Tags: #decision #database        │
│        Links: [[PE Evaluation]]         │
│                                         │
│  **Proceed with sync? (yes/no/edit):**  │
│                                         │
│  ⏸️  WAITING FOR USER CONFIRMATION      │
└────────────────┬────────────────────────┘
                 │
        User confirms
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 6: Execute Approved Actions      │
│                                         │
│  FOR SPLITS:                            │
│                                         │
│  Generate split note files locally:     │
│  Tool: Write                            │
│  Path: ./notes/architecture-choice.md   │
│  Content: [extracted content]           │
│                                         │
│  Delete original (after splits created):│
│  Tool: Bash(rm:*)                       │
│  Path: ./notes/project-notes.md         │
│                                         │
│  FOR MERGES:                            │
│                                         │
│  Tool: obsidian_patch_content           │
│  Params: {                              │
│    filepath: "Projects/API Notes.md",   │
│    operation: "append",                 │
│    target_type: "heading",              │
│    target: "## Updates",                │
│    content: "[merged content from local]│
│  }                                      │
│                                         │
│  FOR SYNC TO VAULT:                     │
│                                         │
│  For each approved note:                │
│                                         │
│  1. Add front matter:                   │
│     Tool: (generate locally)            │
│                                         │
│  2. Write to Obsidian:                  │
│     Tool: obsidian_append_content       │
│     Params: {                           │
│       filepath: "[vault-path]/[note].md"│
│       content: "[full note with fm]"    │
│     }                                   │
│                                         │
│  3. Add backlinks to related notes:     │
│     Tool: obsidian_patch_content        │
│     Params: {                           │
│       filepath: "PE Evaluation.md",     │
│       operation: "append",              │
│       target: "## Related Notes",       │
│       content: "- [[Auth Decision]]"    │
│     }                                   │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 7: Archive Processed Notes       │
│                                         │
│  Move synced notes to archive:          │
│  Tool: Bash(mv:*)                       │
│  From: ./notes/auth-decision.md         │
│  To: ./notes/.archived/auth-decision.md │
│                                         │
│  OR delete if fully merged:             │
│  Tool: Bash(rm:*)                       │
│  Path: ./notes/api-notes.md             │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  PHASE 8: Store to Graphiti             │
│                                         │
│  For each synced note:                  │
│                                         │
│  Tool: mcp__graphiti__add_memory        │
│  Params: {                              │
│    name: "Note: Auth Decision",         │
│    episode_body: "Synced note about JWT │
│      authentication decision. Located   │
│      at Projects/PE Evaluation/",       │
│    group_id: "work:pe-evaluation",      │
│    source: "text",                      │
│    source_description: "audit-notes"    │
│  }                                      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  OUTPUT: Audit Summary                  │
│                                         │
│  ## Note Audit Complete                 │
│                                         │
│  ### Actions Taken                      │
│                                         │
│  **Split**:                             │
│  ✅ project-notes.md → 3 atomic notes   │
│                                         │
│  **Merged**:                            │
│  ✅ api-notes.md → Projects/API Notes.md│
│                                         │
│  **Synced to Vault**:                   │
│  ✅ auth-decision.md                    │
│     → Projects/PE Evaluation/           │
│  ✅ db-choice.md                        │
│     → Projects/PE Evaluation/           │
│  ✅ architecture-choice.md (from split) │
│     → Projects/PE Evaluation/           │
│                                         │
│  **Archived**: 4 local notes            │
│  **Memory**: 4 facts stored to Graphiti │
│                                         │
│  ### Vault Changes                      │
│  - 3 new notes created                  │
│  - 1 note updated (merge)               │
│  - 2 backlinks added                    │
└─────────────────────────────────────────┘
```

---

## Tool Calls Summary

### Read Operations (Auto-Allowed)

| Tool | Purpose | Parameters |
|------|---------|------------|
| `Bash(ls:*)` | List local notes | `./notes/` |
| `Read` | Read local notes | `{file_path: "..."}` |
| `mcp__MCP_DOCKER__obsidian_simple_search` | Find similar vault notes | `{query: "..."}` |
| `mcp__MCP_DOCKER__obsidian_get_file_contents` | Read vault notes | `{filepath: "..."}` |
| `mcp__MCP_DOCKER__obsidian_list_files_in_dir` | Check vault structure | `{dirpath: "..."}` |

### Write Operations (Require Batch Approval)

| Tool | Purpose | Approval |
|------|---------|----------|
| `mcp__MCP_DOCKER__obsidian_append_content` | Create vault notes | **BATCH APPROVAL** |
| `mcp__MCP_DOCKER__obsidian_patch_content` | Update existing notes | **BATCH APPROVAL** |
| `Write` | Create split notes locally | Auto (local) |
| `Bash(mv:*)` | Archive processed notes | Auto (local) |
| `Bash(rm:*)` | Delete merged notes | Auto (local) |
| `mcp__graphiti__add_memory` | Store sync facts | Auto (low risk) |

---

## Skills Required

| Skill | Why Needed |
|-------|------------|
| `obsidian-reader` | Search vault for duplicates, read existing |
| `obsidian-writer` | Write to vault, update notes |
| `graphiti-memory` | Store sync events |
| `semantic-analysis` (proposed) | Duplicate detection, atomicity check |
| `project-context` | Detect current project |

---

## Atomicity Criteria

A note is considered **atomic** if:

| Criterion | Atomic | Non-Atomic |
|-----------|--------|------------|
| Topics | Single main topic | Multiple distinct topics |
| Word count | < 500 words | > 800 words |
| Headings | 1-2 headings | Many unrelated headings |
| Flow | Cohesive narrative | Disconnected sections |
| Purpose | One clear purpose | Mixed purposes |

---

## Duplicate Detection Thresholds

| Similarity | Recommendation |
|------------|----------------|
| > 90% | **Merge**: Content is essentially the same |
| 70-90% | **Link**: Related but distinct, add wikilinks |
| 50-70% | **Review**: May be related, show to user |
| < 50% | **Ignore**: Different enough to be separate |

---

## Error Handling

| Scenario | Handling |
|----------|----------|
| No local notes | Report "no notes to audit" |
| Vault unreachable | Report error, offer local-only audit |
| Split fails | Keep original, report error |
| Merge conflict | Show diff, ask user to resolve |
| Backlink target missing | Create minimal stub note |

---

## Example Output

```markdown
## Note Audit Complete: PE Evaluation

### Summary
- **Notes Audited**: 5
- **Issues Found**: 2
- **Actions Taken**: 7

---

### Split: project-notes.md

Original note contained 3 distinct topics. Split into:

1. ✅ `architecture-choice.md` → Projects/PE Evaluation/
   - Topic: Microservices vs monolith decision
   - Tags: #decision #architecture

2. ✅ `deployment-plan.md` → Projects/PE Evaluation/
   - Topic: AWS ECS deployment strategy
   - Tags: #deployment #aws

3. ✅ `timeline-estimate.md` → Projects/PE Evaluation/
   - Topic: Q1 milestone timeline
   - Tags: #planning #timeline

---

### Merged: api-notes.md

Merged into existing `Projects/API Notes.md`:
- Added section: "December Updates"
- Added content: Rate limiting implementation notes
- Local file archived

---

### Synced to Vault

| Note | Location | Tags | Links |
|------|----------|------|-------|
| auth-decision.md | Projects/PE Evaluation/ | #decision #auth | [[PE Evaluation]] |
| db-choice.md | Projects/PE Evaluation/ | #decision #database | [[PE Evaluation]] |

---

### Backlinks Added

- PE Evaluation.md: Added links to 4 new notes

---

### Memory Updated

4 facts stored to Graphiti group `work:pe-evaluation`

---

### Local Notes Status

- **Archived**: 4 notes moved to ./notes/.archived/
- **Remaining**: 0 notes in ./notes/
```
