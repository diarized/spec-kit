# JIRA Tickets: [FEATURE_NAME]

**Generated**: [TIMESTAMP]
**Source**: [TASKS_FILE_PATH]
**Feature**: [FEATURE_NUMBER]-[FEATURE_NAME]
**Total Tasks**: [TOTAL_TASKS]
**Total Tickets**: [TOTAL_TICKETS]

---

## Overview

This document contains JIRA-sized work tickets generated from the detailed task breakdown in `tasks.md`. Each ticket groups related tasks (approximately 1-3 days of work) and follows the 4-field AI-friendly structure:

1. **Subject**: Brief description (50-100 characters)
2. **Description**: Comprehensive context (6 subsections)
3. **Test Plan**: Verification approach (4 subsections)
4. **Technical Details**: Implementation guidance (4 subsections)

**Usage**:
- Review tickets below for completeness and accuracy
- Use `--interactive` flag to edit tickets before JIRA creation
- Use `--create` flag to automatically create tickets in JIRA via jira-db skill

---

## Ticket [GROUP_ID]: [TICKET_SUBJECT]

**Phase**: [PHASE_NAME]
**Story**: [STORY_LABEL or N/A]
**Tasks Covered**: [TASK_IDS]
**Issue Type**: [Task or Story]

### Subject

[50-100 character summary in verb+object+purpose format]

### Description

**Purpose**: [Why this work is needed - 1-2 sentences]

**Tasks Covered**: [Comma-separated task IDs or bullet list with brief descriptions]

**Affected Systems**:
- [System/module/component 1]
- [System/module/component 2]

**Key Workflows**:
[Description of primary workflows or user interactions affected]

**Stakeholders**:
[User roles who benefit from or are affected by this work]

**Edge Cases**:
- [Edge case 1 to handle]
- [Edge case 2 to handle]

### Test Plan

**Verification Approach**: [High-level testing strategy]

**Component Tests**:
- [Component 1 test description and expected behavior]
- [Component 2 test description and expected behavior]

**Integration Test**: [End-to-end scenario with inputs and expected outputs]

**Edge Case Validation**:
- [Edge case 1 validation steps]
- [Edge case 2 validation steps]

### Technical Details

**Files to Modify**:
- `[file/path/1]` - [Brief purpose]
- `[file/path/2]` - [Brief purpose]

**Functions to Add/Modify**:
- [Function/section 1] - [What it does]
- [Function/section 2] - [What it does]

**Implementation Sequence**:
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Dependencies**:
- **External**: [External dependencies like libraries, skills, tools]
- **Internal**: [Internal dependencies like other tasks, modules]
- **Data**: [Required files, formats, or data structures]
- **Cross-ticket**: [Dependencies on other tickets in this batch]

---

<!-- Repeat above Ticket section for each generated ticket -->

---

## Task-to-Ticket Mapping

**Purpose**: Traceability between original tasks.md and generated JIRA tickets

| Task IDs | Group ID | Phase | Story | Subject | JIRA Key |
|----------|----------|-------|-------|---------|----------|
| T001-T003 | G01 | Setup | N/A | [Subject] | [JIRA-KEY or Not Created] |
| T004-T008 | G02 | Foundational | N/A | [Subject] | [JIRA-KEY or Not Created] |
| T009-T020 | G03 | User Story 1 | US1 | [Subject] | [JIRA-KEY or Not Created] |
| T021-T028 | G04 | User Story 2 | US2 | [Subject] | [JIRA-KEY or Not Created] |
| ... | ... | ... | ... | ... | ... |

**Notes**:
- Task IDs reference original line items in tasks.md
- Group ID is internal identifier for this grouping
- JIRA Key populated after --create flag execution
- "Not Created" indicates dry-run mode or creation not requested

---

## Statistics

### Tickets by Phase

| Phase | Ticket Count | Task Count | Avg Tasks/Ticket |
|-------|--------------|------------|------------------|
| Setup | [COUNT] | [COUNT] | [AVG] |
| Foundational | [COUNT] | [COUNT] | [AVG] |
| User Story 1 (P1) | [COUNT] | [COUNT] | [AVG] |
| User Story 2 (P2) | [COUNT] | [COUNT] | [AVG] |
| User Story 3 (P3) | [COUNT] | [COUNT] | [AVG] |
| User Story 4 (P4) | [COUNT] | [COUNT] | [AVG] |
| Polish | [COUNT] | [COUNT] | [AVG] |
| **Total** | **[TOTAL]** | **[TOTAL]** | **[AVG]** |

### Tickets by Issue Type

| Issue Type | Count | Percentage |
|------------|-------|------------|
| Task | [COUNT] | [PERCENT]% |
| Story | [COUNT] | [PERCENT]% |
| **Total** | **[TOTAL]** | **100%** |

### Grouping Effectiveness

- **Target group size**: 5-8 tasks per ticket
- **Actual range**: [MIN]-[MAX] tasks per ticket
- **Median group size**: [MEDIAN] tasks
- **Tasks successfully grouped**: [COUNT]/[TOTAL] ([PERCENT]%)
- **Tasks requiring single tickets**: [COUNT] (isolated or unique)

---

## Usage Instructions

### Manual JIRA Creation

1. Review tickets above for accuracy and completeness
2. For each ticket, create a JIRA issue manually:
   - **Summary**: Copy Subject field content
   - **Description**: Copy Description field content (convert to ADF if needed)
   - **Test Plan** (customfield_10332): Copy Test Plan field content
   - **Technical Details** (customfield_10301): Copy Technical Details field content
   - **Issue Type**: Use specified type (Task or Story)
   - **Epic Link**: Set if applicable
   - **Team**: Assign if applicable
3. Update Task-to-Ticket Mapping table with created JIRA keys

### Automatic JIRA Creation

**Basic command** (dry-run, no JIRA creation):
```bash
/speckit.jira-tasks
```

**With JIRA creation**:
```bash
/speckit.jira-tasks --create --project PM --team "Platform Team"
```

**With Epic association**:
```bash
/speckit.jira-tasks --create --project PM --team "Platform Team" --epic PM-1000
```

**With interactive review**:
```bash
/speckit.jira-tasks --interactive --create --project PM
```

**Available Flags**:
- `--dry-run`: Preview tickets without creating (default behavior)
- `--create`: Create tickets in JIRA via jira-db skill
- `--project PROJECT_KEY`: JIRA project key (required with --create)
- `--team "Team Name"`: Assign tickets to team (optional)
- `--epic EPIC_KEY`: Associate all tickets with Epic (optional)
- `--interactive`: Review and edit tickets before creation (optional)

**Prerequisites for --create**:
- jira-db skill must be installed and configured
- Valid JIRA credentials in jira-db cache
- Project key must exist in JIRA
- Team name must resolve to valid team_id (if provided)
- Epic key must exist in JIRA (if provided)

---

## Notes

### AI-Friendly Documentation Style

All tickets follow AI-friendly principles:
- **Explicit labeling**: Bold subsection headers (**Purpose:**, **Tasks Covered:**, etc.)
- **Reduced ambiguity**: Specific formats, clear examples, structured data
- **Canonical terminology**: Consistent terms across all tickets
- **Self-contained sections**: Each field provides complete context
- **Quantified requirements**: Specific counts and targets where applicable

### Ticket Editing

**Before JIRA creation**, you can:
1. Edit any ticket field directly in this file (markdown format)
2. Add clarifications or context based on team knowledge
3. Adjust groupings (split large tickets, merge small ones)
4. Update Task-to-Ticket Mapping if groupings change
5. Run `/speckit.jira-tasks --interactive` for guided editing

**After JIRA creation**:
- Update this file with JIRA keys in Task-to-Ticket Mapping table
- Use JIRA interface for further ticket modifications
- This file serves as archival record of original generation

### Troubleshooting

**Empty output or missing tickets**:
- Verify tasks.md exists and follows proper format: `- [ ] T### [P?] [Story?] Description with file path`
- Check that tasks have story labels ([US1], [US2], etc.) for user story phases
- Ensure file paths are present in task descriptions

**Incorrect grouping**:
- Review grouping algorithm parameters (target 5-8 tasks per ticket)
- Check if task story labels match intended user stories
- Use --interactive flag to manually adjust groupings

**JIRA creation failures**:
- Verify jira-db skill configuration: `Skill: "jira-db"`
- Check project key exists: Epic validation logs will show errors
- Verify team name resolves: Error messages list available teams
- Ensure Epic key is valid: Epic validation runs before creation

---

## References

- **Tasks Source**: [TASKS_FILE_PATH]
- **Feature Spec**: [SPECS_DIR]/spec.md
- **Data Model**: [SPECS_DIR]/data-model.md
- **Ticket Template**: templates/jira-ticket-template.md
- **Command**: /speckit.jira-tasks

---

**Document Version**: 1.0.0
**Generated By**: /speckit.jira-tasks command
**Last Updated**: [TIMESTAMP]
