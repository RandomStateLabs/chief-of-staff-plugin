---
name: linear-project-templates
description: Apply and maintain RS42 Linear project templates. Use when creating new projects, updating project descriptions to follow the template, or syncing project structure. Covers project overview, development roadmap with linked issues, success criteria, and related projects patterns.
---
# Linear Project Templates Skill

This skill documents the RS42 Linear project template system and provides patterns for applying consistent project structure across all Linear projects.

## When to Use This Skill

Activate this skill when:

- Creating a new Linear project - new Linear projects require proper template structure
- Updating an existing project to follow the RS42 template
- Reviewing project descriptions for template compliance
- Linking issues to project Development Roadmap sections this is what we wer
- Setting up success criteria or related projects

## RS42 Project Template Structure

### Canonical Template

```markdown
# Project Overview

### Repository
**GitHub**: [repository-url]

### **Goal**
[1-3 sentence description of what the project achieves]

### Target Users
- [User persona 1]
- [User persona 2]
- [User persona 3]

### **Platform & Distribution**
**Platform**: [Primary platform/technology]
**Distribution**: [How users access it]
[Additional deployment details if needed]

## Technology Stack
- **[Category 1]**: [Technology]
- **[Category 2]**: [Technology]
[List relevant technologies, MCP servers, frameworks]

## Development Roadmap

### Phase 1: [Phase Name] [Status]
- [RS4-XX](https://linear.app/rs42/issue/RS4-XX/slug) Issue Title
- [RS4-YY](https://linear.app/rs42/issue/RS4-YY/slug) Issue Title

### Phase 2: [Phase Name] [Status]
- [RS4-ZZ](https://linear.app/rs42/issue/RS4-ZZ/slug) Issue Title

### Phase 3: [Phase Name]
[Issues to be created]

### Phase 4: [Phase Name]
[Issues to be created]

## Success Criteria
- [ ] Criterion 1 (measurable outcome)
- [ ] Criterion 2 (measurable outcome)
- [x] Criterion 3 (completed)

## Related Projects
- **Linear Project**: [Project Name](project-url)
```

## Industry Best Practices Alignment

### What Linear Method Recommends


| Linear Principle                                          | RS42 Implementation                                                 | Status     |
| ----------------------------------------------------------- | --------------------------------------------------------------------- | ------------ |
| **Write project specs** - brief, communicate why/what/how | Goal section explains "why", Technology/Roadmap explains "what/how" | ✅ Aligned |
| **Single project owner**                                  | Lead field in Linear                                                | ✅ Aligned |
| **Clear outcomes/completion dates**                       | Success Criteria with checkboxes                                    | ✅ Aligned |
| **Connect daily work to larger goals**                    | Development Roadmap links issues                                    | ✅ Aligned |
| **Scope issues small**                                    | Sub-issues under phases                                             | ✅ Aligned |
| **Purpose-built workflows**                               | 4-phase standard structure                                          | ✅ Aligned |

### What Real-World Teams Do (from Research)


| Practice                          | Description               | RS42 Adoption                 |
| ----------------------------------- | --------------------------- | ------------------------------- |
| **Outcome-based project names**   | Name by goal, not feature | ✅ Use goal-oriented names    |
| **Consistent naming conventions** | Verb-first issue titles   | ⚠️ Partially adopted        |
| **Milestone tracking**            | Progress checkpoints      | ✅ Phases serve as milestones |
| **Cross-team visibility**         | Related Projects linking  | ✅ Implemented                |
| **Repository linking**            | GitHub/GitLab connections | ✅ Repository section         |

### Unique RS42 Patterns (Beyond Standard Linear)

1. **Phase Development Roadmap** - Number of phases may vary by project type
2. **Issue Linking in Description** - Full URLs with issue IDs in roadmap
3. **Status Indicators** - ✅ 🚧 for phase progress
4. **Success Criteria Checkboxes** - Measurable outcomes as markdown checkboxes
5. **Bidirectional Related Projects** - Cross-reference between projects

## Quick Patterns

### Pattern 1: Create Project with Template

```python
# Step 1: Create project in Linear
mcp__linear__create_project(
    name="[Project Name]",
    team="RS42",
    description="[Use template below]",
    summary="[1-line summary for list views]"
)

# Step 2: Apply template structure to description
# See "Template Application" section below
```

### Pattern 2: Link Issues to Roadmap

Issue link format:

```markdown
- [RS4-XX](https://linear.app/rs42/issue/RS4-XX/issue-slug) Issue Title
```

Full example:

```markdown
### Phase 2: Testing 🚧
- [RS4-54](https://linear.app/rs42/issue/RS4-54/phase-1-test-morning-brief-command) Test morning brief command
- [RS4-55](https://linear.app/rs42/issue/RS4-55/phase-2-test-evening-sync-command) Test evening sync command
```

### Pattern 3: Phase Status Indicators


| Indicator | Meaning     | When to Use                     |
| ----------- | ------------- | --------------------------------- |
| ✅        | Complete    | All issues in phase marked Done |
| 🚧        | In Progress | At least one issue started      |
| (none)    | Not Started | No issues started yet           |

Example:

```markdown
### Phase 1: Configuration ✅
### Phase 2: Testing 🚧
### Phase 3: Documentation
### Phase 4: Distribution
```

### Pattern 4: Success Criteria Format

```markdown
## Success Criteria
- [ ] Core functionality working (specific metric)
- [ ] Tests passing with >80% coverage
- [x] Documentation complete
- [ ] Ready for distribution
```

Rules:

- Use `- [ ]` for incomplete
- Use `- [x]` for complete
- Make criteria **measurable** (numbers, specific outcomes)
- Keep to 4-6 criteria

### Pattern 5: Related Projects Linking

```markdown
## Related Projects
- **Linear Project**: [Sourcing Agent - Claude Code Plugin](https://linear.app/rs42/project/sourcing-agent-claude-code-plugin-e1850a1331e0)
- **Linear Project**: [VC Sourcing Agent Claude Desktop Demo](https://linear.app/rs42/project/vc-sourcing-agent-claude-desktop-demo-c49336190dd1)
```

Rules:

- Use full Linear project URLs
- Bidirectional: if A links to B, B should link to A
- Only link truly related projects (shared domain/tech/user)

## Standard Phase Names by Project Type

### Claude Code Plugin Projects

```markdown
### Phase 1: Configuration/Architecture
### Phase 2: Testing & Validation
### Phase 3: Documentation
### Phase 4: Distribution/Marketplace
```

### Client Service Projects

```markdown
### Phase 1: Setup & Configuration
### Phase 2: Development/Implementation
### Phase 3: Testing & Validation
### Phase 4: Deployment & Handoff
```

### Internal Operations Projects

```markdown
### Phase 1: Design & Planning
### Phase 2: Implementation
### Phase 3: Testing
### Phase 4: Launch & Iteration
```

## Template Application Workflow

When applying template to a project:

### Step 1: Gather Information

```python
# Get project details
mcp__linear__get_project(query="Project Name")

# Get all project issues
mcp__linear__list_issues(project="Project Name", limit=100)
```

### Step 2: Organize Issues by Phase

Map issues to phases based on:

- Issue title keywords (setup, test, deploy, etc.)
- Issue labels
- Issue status (Done issues often Phase 1)
- Logical dependency order

### Step 3: Generate Template

Fill in template sections:

1. **Repository** - Get from project notes or ask
2. **Goal** - Summarize from existing description
3. **Target Users** - Identify from project context
4. **Platform** - Determine from tech stack
5. **Technology Stack** - List from issues/code
6. **Development Roadmap** - Map issues to phases with links
7. **Success Criteria** - Define measurable outcomes
8. **Related Projects** - Find related Linear projects

### Step 4: Apply Template

```python
# Update project description
mcp__linear__update_project(
    id="project-id",
    description="[Full template markdown]"
)
```

## Validation Checklist

When reviewing a project for template compliance:

- [ ] Has Repository section with GitHub link
- [ ] Has Goal section (1-3 sentences)
- [ ] Has Target Users section (2-4 user types)
- [ ] Has Platform & Distribution section
- [ ] Has Technology Stack section
- [ ] Has 4-phase Development Roadmap
- [ ] Issues are linked with full URLs in format `[RS4-XX](url) Title`
- [ ] Phase status indicators (✅ 🚧) are current
- [ ] Has Success Criteria with checkboxes
- [ ] Has Related Projects section (if applicable)

## Common Issues & Fixes

### Issue: No linked issues in roadmap

**Fix**: Get project issues and add links:

```markdown
### Phase 1: Setup ✅
- [RS4-24](https://linear.app/rs42/issue/RS4-24/...) Set up Google Cloud Project
- [RS4-25](https://linear.app/rs42/issue/RS4-25/...) Create service account
```

### Issue: Missing phase status indicators

**Fix**: Check issue statuses and add indicators:

- If all Done → ✅
- If any In Progress → 🚧
- If all Todo/Backlog → no indicator

### Issue: Success criteria not measurable

**Bad**: `- [ ] App works well`
**Good**: `- [ ] 100% test pass rate on core workflows`

### Issue: Related projects not bidirectional

**Fix**: Update both projects to link to each other

## Integration with Other Skills

This skill works with:

- **linear-integration** skill - for reading/writing project data
- **obsidian-reader** skill - for syncing with Obsidian project notes

## References

- Linear Setup Project: [RS4-59](https://linear.app/rs42/issue/RS4-59/project-template) - Project Template issue
- Obsidian Template: `3 - Template/Linear Project Template.md`
- Linear Method: https://linear.app/method
- Linear Docs - Projects: https://linear.app/docs/projects

## Areas for Alignment (EDITABLE)

> **Note**: This section captures areas where the template may need refinement based on real usage. Edit this section to document decisions.

### Open Questions

1. **Phase naming consistency** - Should all projects use exact same phase names or allow project-type variations?

   - Current: Allow variations by project type
   - Decision: _[To be decided]_
2. **Issue linking depth** - Should sub-issues be listed under parent issues in roadmap?

   - Current: Only top-level issues linked
   - Decision: _[To be decided]_
3. **Success criteria granularity** - How many criteria per project?

   - Current: 4-6 suggested
   - Decision: _[To be decided]_
4. **Related projects threshold** - When should projects be linked?

   - Current: Shared domain/tech/user
   - Decision: _[To be decided]_

### Template Revisions Log


| Date       | Change                 | Reason                        |
| ------------ | ------------------------ | ------------------------------- |
| 2025-11-28 | Initial skill creation | Standardize project templates |
| _[Date]_   | _[Change]_             | _[Reason]_                    |
