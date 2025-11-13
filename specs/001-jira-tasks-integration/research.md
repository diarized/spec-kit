# Research: JIRA Tasks Integration

**Feature**: [spec.md](spec.md) | **Plan**: [plan.md](plan.md)
**Date**: 2025-11-12

## Overview

This document consolidates research findings and technical decisions for implementing the `/speckit.jira-tasks` command.

## Decision 1: Slash Command Architecture Pattern

**Context**: Need to implement a new Claude Code slash command that follows existing Spec Kit patterns.

**Decision**: Use Markdown-based command file in `.claude/commands/` with structured sections

**Rationale**:
- Existing commands (speckit.tasks, speckit.plan, etc.) use this pattern successfully
- Claude Code natively executes Markdown files as prompts
- Allows rich documentation alongside executable logic
- Enables version control and diffing of command logic

**Pattern from speckit.tasks.md**:
```markdown
---
description: [Brief command description]
---

## User Input
$ARGUMENTS

## Outline
1. Setup: Run bash script, parse JSON paths
2. Load context: Read required files
3. Execute workflow: Process and transform data
4. Generate output: Write to template-based file
5. Report: Summary and validation
```

**Implementation Approach**:
- Frontmatter for command metadata
- User Input section to capture optional arguments
- Outline section for step-by-step workflow
- Separate sections for complex logic (Task Grouping Rules, JIRA Creation, etc.)

**Alternatives Considered**:
- Python CLI extension: Rejected because adds complexity, requires package changes
- Bash script only: Rejected because insufficient for complex text processing and LLM reasoning
- Inline prompts: Rejected because less maintainable than structured Markdown

---

## Decision 2: Task Grouping Algorithm Strategy

**Context**: Need to group 50-100 granular tasks into 8-15 JIRA-sized work units.

**Decision**: Multi-pass grouping using story labels, phase markers, and semantic clustering

**Rationale**:
- User story labels ([US1], [US2]) provide primary organizational structure
- Phase markers (Setup, Foundational, Polish) require separate handling
- File path similarity indicates related work
- Task descriptions may indicate cross-cutting concerns

**Algorithm Outline**:
```text
Pass 1: Separate by phase
  - Setup tasks → separate group
  - Foundational tasks → separate group (may split if >8 tasks)
  - Polish tasks → separate group

Pass 2: Group by user story label
  - Tasks with [US1] → US1 groups
  - Tasks with [US2] → US2 groups
  - Etc.

Pass 3: Within each story, cluster by similarity
  - Group tasks targeting same file/module
  - Keep groups to 5-8 tasks (1-3 days work estimate)
  - Preserve task execution order (earlier task IDs stay together)

Pass 4: Validate and adjust
  - Ensure no group exceeds ~12 tasks
  - Check that dependencies stay within group when possible
  - Verify all tasks are assigned to exactly one group
```

**Alternatives Considered**:
- Simple file-based grouping: Rejected because ignores user story structure
- Manual configuration: Rejected because loses automation benefit
- Machine learning clustering: Rejected as over-engineered for this problem size

---

## Decision 3: AI-Friendly Documentation Style Application

**Context**: JIRA tickets must follow AI-friendly principles from ~/.claude/AI-FRIENDLY.md

**Decision**: Apply principles to all 4 ticket fields (Subject, Description, Test Plan, Technical Details)

**Rationale from AI-FRIENDLY.md**:
1. **Clear Structure with Consistent Hierarchy**: Use markdown headers in Description/Test Plan/Technical Details
2. **Explicit Labeling**: Use bold labels (e.g., **Purpose:**, **Affected Files:**)
3. **Reduced Ambiguity**: Structured lists instead of prose where appropriate
4. **Canonical Terminology**: Use consistent terms (e.g., "task ID" not "task number/identifier/ref")
5. **Self-Contained Sections**: Each field complete without requiring cross-references
6. **Explicit Relationships**: List task IDs covered, dependencies on other tickets if any

**Field Structure Template**:

```markdown
## Subject
[Single sentence: verb + object + purpose]

## Description
**Purpose**: [Why this work is needed]
**Tasks Covered**: [T001, T002, T005, T008]
**Affected Systems**: [List of modules/areas]
**Key Workflows**: [What user/system workflows change]
**Stakeholders**: [Who cares about this work]
**Edge Cases**: [Notable corner cases to handle]

## Test Plan
**Verification Approach**: [How to verify this works]
**Component Tests**:
- [Component 1]: [How to test it]
- [Component 2]: [How to test it]
**Integration Test**: [End-to-end verification]
**Edge Case Validation**: [How to test edge cases]

## Technical Details
**Files to Modify**:
- `path/to/file1.ext`: [What to change]
- `path/to/file2.ext`: [What to change]

**Functions to Add/Modify**:
- `functionName(params)`: [Purpose and implementation notes]

**Implementation Sequence**:
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Dependencies**:
- Internal: [Dependencies on other tasks/tickets]
- External: [Dependencies on libraries, services]
```

**Alternatives Considered**:
- Free-form text: Rejected because less parseable by AI
- JIRA's default structure: Rejected because lacks AI-friendly explicit labeling
- Separate template per issue type: Rejected because adds maintenance burden

---

## Decision 4: JIRA Integration via jira-db Skill

**Context**: Need to create tickets in JIRA programmatically with custom fields.

**Decision**: Use existing jira-db skill's ticket creation capability with ADF format

**Rationale**:
- jira-db skill already available in Spec Kit environment (referenced in spec)
- Handles authentication and JIRA API complexity
- Supports custom fields (customfield_10332 for Test Plan, customfield_10301 for Technical Details)
- Provides team name → team ID resolution
- Handles Epic parent link field assignment

**Required Integration Points**:
1. Field mapping to JIRA:
   - Subject → `summary` (plain text)
   - Description → `description` (ADF format)
   - Test Plan → `customfield_10332` (ADF format)
   - Technical Details → `customfield_10301` (ADF format)

2. Issue type inference:
   - Setup/Foundational/Polish phase → "Task"
   - User Story phase ([US1], [US2], etc.) → "Story"

3. Optional Epic association:
   - If `--epic` flag provided → set Epic Link field
   - Validate Epic exists before batch creation

4. Team assignment:
   - Resolve team name to team ID using jira-db capability
   - Assign to resolved team ID

**ADF (Atlassian Document Format) Conversion**:
- Markdown → ADF conversion required for rich text fields
- Structure: Paragraphs, headings, lists, bold/italic inline marks
- Reference: https://developer.atlassian.com/cloud/jira/platform/apis/document/structure/

**Alternatives Considered**:
- Direct JIRA REST API calls: Rejected because reinvents authentication, rate limiting
- JIRA CLI tools: Rejected because adds external dependency
- Manual ticket creation: Rejected because defeats automation goal

---

## Decision 5: Error Handling Strategy

**Context**: Multiple failure modes (missing files, skill unavailable, JIRA errors).

**Decision**: Fail fast with actionable error messages; graceful degradation where possible

**Error Scenarios and Handling**:

1. **Missing tasks.md**:
   - Detection: Check file existence in setup script
   - Response: Error message: "tasks.md not found. Run /speckit.tasks first."
   - Recovery: Cannot proceed - FATAL

2. **jira-db skill not configured** (when --create used):
   - Detection: Attempt skill invocation, catch error
   - Response: Error with setup instructions for jira-db skill
   - Recovery: Cannot create tickets, but jira-tickets.md still generated

3. **Team name not found**:
   - Detection: Empty response from team resolution
   - Response: List available teams from jira-db cache
   - Recovery: User must re-run with valid team name

4. **Epic key invalid** (when --epic used):
   - Detection: Validation call before batch creation
   - Response: Error: "Epic PM-1000 not found in JIRA"
   - Recovery: User must verify Epic exists or omit --epic flag

5. **Partial JIRA creation failure**:
   - Detection: Track successful vs. failed ticket creations
   - Response: Report which tickets created, which failed with reasons
   - Recovery: Manual retry or investigation of failures

6. **Invalid task format in tasks.md**:
   - Detection: Regex validation of checklist format
   - Response: Warning about malformed tasks, continue with valid ones
   - Recovery: User should fix tasks.md and re-run

**Logging/Observability**:
- Progress messages: "Loading tasks.md...", "Grouping 87 tasks...", "Creating 12 tickets..."
- Dry-run output: Show what would be created without side effects
- Task mapping output: "Tasks T001-T008 → PM-12345 [Story]"

**Alternatives Considered**:
- Retry logic: Rejected as over-engineered for manual command execution
- Transaction rollback: Rejected because JIRA ticket creation not transactional
- Silent failures: Rejected because violates Observability constitution principle

---

## Decision 6: Output Format for jira-tickets.md

**Context**: Need intermediate file format for review and traceability.

**Decision**: Markdown format with structured sections, human-readable and version-controllable

**File Structure**:
```markdown
# JIRA Tickets: [Feature Name]

**Generated**: [ISO timestamp]
**Source**: [path to tasks.md]
**Ticket Count**: [N]
**Task Count**: [M]

## Ticket 1: [Subject]

**Phase**: [Setup/Foundational/User Story N/Polish]
**Issue Type**: [Task/Story]
**Tasks Covered**: [T001, T002, T005]

### Description
[Full description content]

### Test Plan
[Full test plan content]

### Technical Details
[Full technical details content]

---

## Ticket 2: [Subject]
...

---

## Task-to-Ticket Mapping

| Task IDs | JIRA Key | Issue Type | Subject |
|----------|----------|------------|---------|
| T001-T008 | PM-12345 | Story | [Subject] |
| T009-T015 | PM-12346 | Task | [Subject] |

---

## Statistics

- Total Tasks: [N]
- Total Tickets: [M]
- Average Tasks per Ticket: [N/M]
- Setup Phase Tickets: [X]
- Foundational Phase Tickets: [Y]
- User Story Tickets: [Z]
- Polish Phase Tickets: [W]
```

**Rationale**:
- Human-readable format for review before JIRA creation
- Version controllable (can track ticket evolution)
- Easy to parse if automation needed
- Clear traceability with task mapping table

**Alternatives Considered**:
- JSON format: Rejected because less human-readable
- Direct JIRA creation only: Rejected because removes review step
- Separate file per ticket: Rejected because harder to review collectively

---

## Decision 7: Dry-Run Mode Implementation

**Context**: Users need to preview tickets without JIRA side effects.

**Decision**: `--dry-run` flag shows full ticket content and what would be created

**Behavior**:
```text
$ /speckit.jira-tasks --dry-run

Loading tasks.md... ✓ (87 tasks loaded)
Grouping tasks by story and phase... ✓ (12 groups formed)
Generating ticket content... ✓ (12 tickets)

DRY-RUN MODE: The following tickets would be created:

Ticket 1 [Story]: Implement User Authentication
  Tasks: T012-T020 (9 tasks)
  Files: src/auth/service.py, src/auth/middleware.py, src/models/user.py
  Epic: PM-1000 (if --epic provided)
  Team: Engineering (if --team provided)

Ticket 2 [Task]: Setup Project Infrastructure
  Tasks: T001-T006 (6 tasks)
  Files: pyproject.toml, src/__init__.py, tests/conftest.py
  Epic: None (Setup phase, no Epic association)
  Team: Engineering

... [remaining tickets]

Output written to: /path/to/specs/###-feature/jira-tickets.md
To create these tickets in JIRA, run with --create flag
```

**Alternatives Considered**:
- No dry-run: Rejected because risky for bulk operations
- Minimal preview: Rejected because users need full context to review
- Interactive confirmation: Rejected because adds friction for automation

---

## Best Practices from Existing Commands

**From speckit.tasks.md**:
1. Use bash setup script for path validation and JSON output
2. Quote handling for arguments: `'I'\''m Groot'` syntax
3. Report format: Clear summary with counts and validations
4. Template-based generation for consistency

**From AI-FRIENDLY.md**:
1. Clear structure with consistent hierarchy
2. Explicit labeling (use bold markers)
3. Canonical terminology (use same terms throughout)
4. Self-contained sections (no forward references)

**From Constitution**:
1. Explicit error messages with recovery guidance
2. Progress output for observability
3. Independent user story delivery (P1 MVP, then P2, P3, P4)
4. Simplicity bias (no unnecessary abstractions)

---

## Open Questions for Task Generation Phase

None - all technical decisions resolved. Ready for Phase 1 (Design & Contracts).

---

## References

- [spec.md](spec.md) - Feature specification
- [plan.md](plan.md) - Technical implementation plan
- [~/.claude/AI-FRIENDLY.md](~/.claude/AI-FRIENDLY.md) - AI-friendly documentation principles
- [.specify/memory/constitution.md](../../.specify/memory/constitution.md) - Project governance
- [.claude/commands/speckit.tasks.md](../../.claude/commands/speckit.tasks.md) - Reference command pattern
