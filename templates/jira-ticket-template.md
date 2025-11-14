# JIRA Ticket Template

**Purpose**: Defines the canonical 4-field structure for JIRA tickets generated from tasks.md

**Usage**: This template is used by `/speckit.jira-tasks` command to generate properly structured tickets with AI-friendly documentation style

---

## Field 1: Subject

**Format**: 50-100 characters, verb+object+purpose

**Structure**: `[Verb] [Object] [Purpose/Outcome]`

**Examples**:

- "Implement task grouping algorithm for ticket generation"
- "Create JIRA tickets via jira-db skill with team assignment"
- "Add interactive review mode for ticket refinement"

**Guidelines**:

- Start with action verb (Implement, Create, Add, Build, Configure, etc.)
- Include the primary object being worked on
- Briefly state the purpose or outcome
- Keep under 100 characters for JIRA summary field compatibility
- Avoid technical jargon when possible
- Be specific enough that the work scope is clear

---

## Field 2: Description

**Purpose**: Provides comprehensive context for the work, answers "what" and "why"

**Structure**: 6 subsections with explicit bold labels

### Subsection 1: **Purpose**

- 1-2 sentences explaining why this work is needed
- Connect to user value or system capability
- Example: "This work enables automatic conversion of granular AI tasks into human-friendly JIRA tickets, eliminating manual grouping effort and reducing ticket creation time from 30+ minutes to under 30 seconds."

### Subsection 2: **Tasks Covered**

- List original task IDs from tasks.md that this ticket encompasses
- Format: Comma-separated list or bullet points
- Example: "T012, T013, T014, T015, T016" or:
  - T012: Implement Pass 1 - separate by phase
  - T013: Implement Pass 2 - group by story label
  - T014: Implement Pass 3 - cluster by file paths

### Subsection 3: **Affected Systems**

- List systems, modules, or components touched by this work
- Include file paths or directory names
- Example:
  - `templates/commands/jira-tasks.md` (command implementation)
  - `templates/jira-ticket-template.md` (ticket structure)
  - Task grouping algorithm (5-pass multi-dimensional)

### Subsection 4: **Key Workflows**

- Describe the primary workflows or user interactions affected
- Explain how users will interact with the implemented functionality
- Example: "Users run `/speckit.jira-tasks` command, which parses tasks.md, groups tasks by story label and file paths, generates tickets with 4-field structure, outputs to jira-tickets.md for review"

### Subsection 5: **Stakeholders**

- Identify who benefits from or is affected by this work
- Include user roles from spec.md user stories
- Example: "Project managers (ticket creation automation), Developers (AI-friendly ticket content), Technical leads (review capabilities)"

### Subsection 6: **Edge Cases**

- List edge cases or special scenarios to handle
- Reference edge cases from spec.md if applicable
- Example:
  - Empty tasks.md: Provide clear error message
  - Tasks without file paths: Flag as incomplete, warn in output
  - Large task count (200+): Warn about scope, suggest splitting

**AI-Friendly Style Principles**:

- Use **bold labels** for all subsection headers
- Write in bullet points or short paragraphs
- Avoid ambiguous terms ("robust", "intuitive" without quantification)
- Use canonical terminology consistently
- Provide explicit structure rather than prose

---

## Field 3: Test Plan

**Purpose**: Describes how to verify the implementation works correctly

**Structure**: 4 subsections with explicit bold labels

### Subsection 1: **Verification Approach**

- High-level testing strategy for this ticket
- What needs to be verified and how
- Example: "Manual verification by running command on spec-kit's own tasks.md (meta-test), examining generated jira-tickets.md for proper grouping, structure, and content quality"

### Subsection 2: **Component Tests**

- List individual components or functions to test
- Specify expected behavior for each
- Example:
  - Task parsing: Verify all 71 tasks from tasks.md are parsed with id, parallel, story_label, description, file_path
  - Pass 1 grouping: Verify tasks separated into setup, foundational, user_story, polish phases
  - Pass 2 grouping: Verify user_story tasks grouped by [US1], [US2], [US3], [US4] labels

### Subsection 3: **Integration Test**

- End-to-end test scenarios combining multiple components
- Specify inputs and expected outputs
- Example: "Run `/speckit.jira-tasks` on specs/001-jira-tasks-integration/tasks.md, verify output file specs/001-jira-tasks-integration/jira-tickets.md contains 8-12 tickets, each with Subject (50-100 chars), Description (6 subsections), Test Plan (4 subsections), Technical Details (4 subsections)"

### Subsection 4: **Edge Case Validation**

- Test edge cases identified in Description field
- Verify error handling and warning messages
- Example:
  - Test with empty tasks.md: Verify error message "tasks.md is empty or missing. Run /speckit.tasks first."
  - Test with 200+ tasks: Verify warning about large scope
  - Test with tasks missing file paths: Verify warning in output

**Testing Guidelines**:

- If automated tests requested: Specify test file locations and test names
- If manual verification only: Provide clear step-by-step instructions
- Include acceptance criteria from spec.md acceptance scenarios
- Reference user story independent test criteria where applicable

---

## Field 4: Technical Details

**Purpose**: Provides implementation guidance for developers and AI agents

**Structure**: 4 subsections with explicit bold labels

### Subsection 1: **Files to Modify**

- List exact file paths that need changes
- Include new files to create
- Format: Bullet list with file paths and brief purpose
- Example:
  - `templates/commands/jira-tasks.md` - Add grouping algorithm passes 1-5
  - `templates/jira-ticket-template.md` - Define 4-field structure (this file)
  - `templates/jira-tickets-template.md` - Define output format

### Subsection 2: **Functions to Add/Modify**

- List functions, methods, or sections to implement/change
- Include signatures or interfaces if applicable
- For markdown commands: List workflow steps or sections
- Example:
  - Add "Pass 1: Separate by phase" section with logic to categorize tasks
  - Add "Pass 2: Group by story label" section with clustering algorithm
  - Implement task parsing logic in "Step 2: Parse tasks.md" section

### Subsection 3: **Implementation Sequence**

- Order of implementation steps
- Dependencies between steps
- Recommended approach
- Example:
  1. Create jira-ticket-template.md (this task, no dependencies)
  2. Create jira-tickets-template.md (parallel with T001)
  3. Create command file with YAML frontmatter (parallel with T001-T002)
  4. Implement task parsing (depends on command file existing)
  5. Implement grouping passes 1-5 (depends on task parsing)

### Subsection 4: **Dependencies**

- External dependencies (libraries, skills, tools)
- Internal dependencies (other tasks, modules, configurations)
- Data dependencies (required files, formats)
- Example:
  - **External**: jira-db skill (for JIRA integration in later phases)
  - **Internal**: tasks.md must exist (prerequisite)
  - **Data**: Task checklist format: `- [ ] T### [P?] [Story?] Description with file path`
  - **Cross-ticket**: Ticket G02 (JIRA integration) depends on this ticket's grouping algorithm

**Implementation Guidelines**:

- Provide enough detail for an LLM to implement without asking questions
- Include exact file paths (absolute when possible, relative to repo root otherwise)
- Reference data model entities from data-model.md where applicable
- Note any assumptions or constraints
- Specify error handling requirements

---

## Usage Notes

**For Command Implementation**:

- Use this template to generate ticket content in `/speckit.jira-tasks` command
- Replace placeholders with actual content from TaskGroup entities
- Maintain all 4 fields in every generated ticket
- Apply AI-friendly style: bold labels, bullet points, explicit structure

**For JIRA Integration**:

- **Subject** → JIRA `summary` field (string)
- **Description** → JIRA `description` field (ADF format)
- **Test Plan** → JIRA `customfield_10332` (ADF format)
- **Technical Details** → JIRA `customfield_10301` (ADF format)

**For Human Review**:

- All fields should be readable in jira-tickets.md (markdown format)
- Tickets can be edited in jira-tickets.md before JIRA creation
- Interactive mode allows field-by-field editing

---

## AI-Friendly Principles Applied

1. **Explicit Labeling**: All subsections use bold labels (**Purpose:**, **Tasks Covered:**, etc.)
2. **Reduced Ambiguity**: Specific formats, examples, and structures provided
3. **Canonical Terminology**: Consistent terms (TaskGroup, User Story, Phase, etc.)
4. **Structured Data**: Bullet points, numbered lists, clear hierarchies
5. **Self-Contained**: Each field provides complete context independently
6. **Quantified Requirements**: Specific counts (6 subsections, 4 subsections, 50-100 chars)
7. **Clear Examples**: Multiple examples demonstrate expected output

---

**Version**: 1.0.0
**Created**: 2025-11-14
**Last Updated**: 2025-11-14
