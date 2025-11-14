# Feature Specification: JIRA Tasks Integration

**Feature Branch**: `001-jira-tasks-integration`
**Created**: 2025-11-12
**Status**: Draft
**Input**: User description: "I'd like the task command (task.md template) to be able to create JIRA tickets using jira-db skill. Please create a new command '/speckit.jira-tasks' to avoid a conflict with the original '/speckit.tasks'. The new command should be able to split work on fragments that require 1-3 Story Points (aproximately days). The original tasks command generates very small tasks easy to handle by AI without exhausting context and are well scoped. The tasks closely related to each other should be grouped in a slightly bigger chunk of work and described precisely in a JIRA ticket."

## Clarifications

### Session 2025-11-12

- Q: How should the command handle Epic assignment for generated tickets? → A: Support both modes via `--epic JIRA_EPIC_NUMBER` flag - when provided, associate all tickets with that Epic as parent; when omitted, tickets created without Epic association (manual assignment in JIRA)
- Q: What JIRA issue type should be used for generated tickets? → A: Infer issue type from content - setup/foundational phase tasks become "Task" issue type, user story phase tasks become "Story" issue type
- Q: Should the command set Story Points field in JIRA? → A: No - leave story points unspecified, allowing teams to estimate during their refinement process

### Session 2025-11-14

- Q: How should the `/speckit.jira-tasks` command be distributed to users? → A: Add to release build process - command integrated into GitHub release ZIP templates so it's automatically included during `specify init` for all new projects, making it a standard spec-kit command alongside `/speckit.tasks`
- Q: Where should the jira-ticket-template.md and jira-tickets-template.md files be stored? → A: Create in repo's templates/ directory (templates/jira-ticket-template.md and templates/jira-tickets-template.md) that get copied to .specify/templates/ during installation, following the same pattern as tasks-template.md
- Q: Should a new jira-tasks-setup.sh script be created or should existing check-prerequisites.sh be extended? → A: Extend existing check-prerequisites.sh in scripts/bash/ to handle both tasks and jira-tasks commands via flags (e.g., --for-jira-tasks), keeping validation logic centralized and avoiding duplication
- Q: Should this be implemented as a Python script, shell script, or Claude Code slash command? → A: Implement as Claude Code slash command (markdown file in templates/commands/jira-tasks.md) - Claude interprets instructions to parse tasks, group them, generate tickets, and call jira-db skill for JIRA operations, following same pattern as /speckit.tasks
- Q: How should the command's YAML frontmatter specify the prerequisite script call? → A: Use scripts section with --for-jira-tasks flag (e.g., `scripts: { sh: "scripts/bash/check-prerequisites.sh --json --for-jira-tasks" }`) so script returns both TASKS_FILE input path and JIRA_TICKETS_FILE output path in JSON response

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Generate JIRA-sized Work Units (Priority: P1)

As a project manager using Spec-Driven Development, I want to convert the granular AI-friendly tasks from tasks.md into JIRA-sized work tickets (1-3 story points) so that I can track feature implementation in JIRA while maintaining the detailed task breakdown for AI agents.

**Why this priority**: This is the core value proposition - bridging the gap between AI-friendly micro-tasks and human project management in JIRA. Without this, users must manually group and create tickets, which is time-consuming and error-prone.

**Independent Test**: Can be fully tested by running `/speckit.jira-tasks` on an existing tasks.md file and verifying it produces a jira-tickets.md file with properly grouped work units. Success is achieved when grouped tickets preserve task boundaries and contain AI-friendly descriptions.

**Acceptance Scenarios**:

1. **Given** a feature has tasks.md with 50+ granular tasks organized by user story, **When** I run `/speckit.jira-tasks`, **Then** the system groups related tasks into 8-12 JIRA-sized work units that map to logical functional areas
2. **Given** tasks.md contains Phase 2 (Foundational) tasks, **When** grouping occurs, **Then** foundational tasks are grouped separately from user story implementation tasks
3. **Given** tasks marked with [P] for parallel execution, **When** grouping occurs, **Then** these tasks can be in separate tickets if they represent different functional areas
4. **Given** tasks with [US1], [US2] story labels, **When** grouping occurs, **Then** tasks are primarily grouped by their user story association
5. **Given** grouped tasks, **When** output is generated, **Then** each group shows original task IDs for traceability

---

### User Story 2 - Create Tickets with AI-Friendly 4-Field Structure (Priority: P2)

As a developer using AI assistance, I want JIRA tickets to follow the 4-field structure (Subject, Description, Test Plan, Technical Details) with AI-friendly documentation style so that both AI agents and human developers can efficiently understand and implement the work.

**Why this priority**: The ticket structure is what makes tickets useful for AI-assisted development. Without proper structure following AI-friendly principles, tickets become vague project management artifacts rather than actionable implementation guides.

**Independent Test**: Can be tested by examining generated jira-tickets.md and verifying each ticket has all 4 fields properly filled with content following AI-friendly principles (clear structure, explicit labeling, reduced ambiguity, canonical terminology).

**Acceptance Scenarios**:

1. **Given** a group of related tasks, **When** generating ticket content, **Then** Subject is a single clear sentence describing the work purpose
2. **Given** a group of tasks from tasks.md with file paths and descriptions, **When** generating Description field, **Then** it includes: what to do, why it's needed, affected systems, and references original task IDs
3. **Given** tasks that create testable functionality, **When** generating Test Plan field, **Then** it describes how to verify each major component works correctly
4. **Given** tasks with specific file paths and implementation details, **When** generating Technical Details field, **Then** it lists modules/files to modify, function signatures to add/change, and implementation approach
5. **Given** all 4 fields, **When** content is written, **Then** it follows AI-friendly style: clear headers, explicit labels, structured data where appropriate, minimal ambiguity

---

### User Story 3 - Auto-Create JIRA Tickets via jira-db Skill (Priority: P3)

As a project manager, I want the command to automatically create JIRA tickets using the jira-db skill so that I can populate JIRA with properly structured work items without manual data entry.

**Why this priority**: Automation of ticket creation saves time and ensures consistency, but it depends on having proper ticket content (P1, P2). This is an enhancement that adds convenience but isn't required for the core value of generating well-structured ticket content.

**Independent Test**: Can be tested by running `/speckit.jira-tasks --create --project PM --team "Team Name" --epic PM-1000` and verifying tickets appear in JIRA with correct content, assignee (team), Epic parent association, and metadata. Success requires jira-db skill configured with valid credentials.

**Acceptance Scenarios**:

1. **Given** jira-tickets.md exists with properly structured tickets, **When** I run with `--create` flag and provide project key and team name, **Then** tickets are created in JIRA via jira-db skill
2. **Given** ticket creation is requested, **When** team name is provided, **Then** system resolves team name to team ID using jira-db's team resolution
3. **Given** tickets grouped from Setup or Foundational phase tasks, **When** creating in JIRA, **Then** these tickets are created with "Task" issue type
4. **Given** tickets grouped from User Story phase tasks (US1, US2, etc.), **When** creating in JIRA, **Then** these tickets are created with "Story" issue type
5. **Given** ticket creation is requested, **When** `--epic PM-1000` flag is provided, **Then** all created tickets have PM-1000 set as their Epic parent link
6. **Given** ticket creation is requested without `--epic` flag, **When** tickets are created, **Then** tickets have no Epic parent association (can be manually assigned later in JIRA)
7. **Given** ticket creation is requested, **When** `--dry-run` flag is provided, **Then** system shows what tickets would be created (including issue type and Epic association if specified) without actually creating them
8. **Given** ticket content with Subject, Description, Test Plan, Technical Details, **When** creating in JIRA, **Then** these fields map correctly to JIRA issue fields (summary, description, customfield_10332, customfield_10301)
9. **Given** successful ticket creation, **When** command completes, **Then** output shows mapping between original task IDs and created JIRA ticket keys with issue types (e.g., "Tasks T012-T018 → PM-12345 [Story]")

---

### User Story 4 - Interactive Review and Refinement (Priority: P4)

As a technical lead, I want to review and modify generated ticket content before JIRA creation so that I can adjust groupings, edit descriptions, or add context based on team-specific knowledge.

**Why this priority**: While automated generation is valuable, human oversight ensures tickets match team conventions and capture domain knowledge. This is a quality-of-life enhancement after core functionality works.

**Independent Test**: Can be tested by running command in interactive mode, modifying ticket content when prompted, and verifying changes are preserved in both jira-tickets.md and created JIRA tickets.

**Acceptance Scenarios**:

1. **Given** jira-tickets.md is generated, **When** I run with `--interactive` flag, **Then** system presents each ticket for review before proceeding
2. **Given** a ticket is presented for review, **When** I choose to edit, **Then** system allows editing each field (Subject, Description, Test Plan, Technical Details)
3. **Given** I modify ticket content during review, **When** I save changes, **Then** modified content is written back to jira-tickets.md
4. **Given** I modify task groupings (split or merge tickets), **When** I confirm changes, **Then** task ID mappings are updated accordingly
5. **Given** review is complete, **When** I approve tickets, **Then** system proceeds with JIRA creation using final reviewed content

---

### Edge Cases

- **Empty or missing tasks.md**: Command should detect this and provide clear error message explaining that `/speckit.tasks` must be run first
- **Tasks without file paths**: System should flag these as incomplete tasks that need more detail before grouping into JIRA tickets
- **Very large number of tasks (200+)**: System should still group effectively but may warn that feature scope is large and suggest breaking into multiple features
- **Conflicting user story labels**: If a task has multiple story labels or belongs to multiple phases, system should use primary label (first occurrence) and note the ambiguity
- **jira-db skill not configured**: When `--create` flag is used but jira-db credentials are missing, provide helpful error with setup instructions
- **Custom field IDs vary by JIRA instance**: System should allow configuration of field mappings or detect them from JIRA metadata
- **Team name not found**: When team name doesn't resolve to a team ID, provide list of available teams from jira-db cache
- **Invalid Epic key**: When `--epic` flag is provided with non-existent Epic key, fail fast with clear error message and suggest verifying Epic exists in JIRA
- **Epic key from different project**: When Epic key belongs to different project than `--project`, warn user about cross-project linkage (may be intentional for portfolio management)
- **Polish phase tasks**: When tasks belong to Polish/final phase (not setup/foundational/user story), default to "Task" issue type
- **Mixed phase grouping**: If a ticket group accidentally contains tasks from multiple phases, use the majority phase to determine issue type
- **Task dependencies across tickets**: When tasks have dependencies that span multiple JIRA tickets, Technical Details should note these cross-ticket dependencies

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Command YAML frontmatter MUST specify scripts section with --for-jira-tasks flag: `scripts: { sh: "scripts/bash/check-prerequisites.sh --json --for-jira-tasks" }`
- **FR-002**: check-prerequisites.sh MUST return JSON with TASKS_FILE (input path to tasks.md) and JIRA_TICKETS_FILE (output path to jira-tickets.md) when called with --for-jira-tasks flag
- **FR-003**: System MUST read tasks.md from path specified in TASKS_FILE from check-prerequisites.sh JSON output
- **FR-004**: System MUST parse task checklist format: `- [ ] [TaskID] [P?] [Story?] Description with file path`
- **FR-005**: System MUST group tasks into work units targeting 1-3 story points (approximately 1-3 days of work)
- **FR-006**: System MUST preserve original task IDs within grouped tickets for traceability
- **FR-007**: System MUST generate tickets with 4-field structure: Subject, Description, Test Plan, Technical Details
- **FR-008**: System MUST follow AI-friendly documentation style as defined in ~/.claude/AI-FRIENDLY.md
- **FR-009**: System MUST prefer grouping by user story label ([US1], [US2], etc.) as primary organization
- **FR-010**: System MUST create separate groups for Setup (Phase 1), Foundational (Phase 2), and Polish (final phase) tasks
- **FR-011**: System MUST output results to path specified in JIRA_TICKETS_FILE from check-prerequisites.sh JSON output
- **FR-012**: System MUST support `--dry-run` flag to preview ticket generation without creating in JIRA
- **FR-013**: System MUST support `--create` flag with `--project`, `--team`, and optional `--epic` options to auto-create JIRA tickets via jira-db skill
- **FR-014**: System MUST resolve team names to team IDs using jira-db's team resolution capability
- **FR-015**: System MUST map 4-field structure to JIRA fields: Subject→summary, Description→description, Test Plan→customfield_10332, Technical Details→customfield_10301
- **FR-016**: System MUST use ADF (Atlassian Document Format) for rich text fields (description, Test Plan, Technical Details)
- **FR-017**: System MUST report mapping between task IDs and created JIRA ticket keys
- **FR-018**: When `--epic JIRA_EPIC_NUMBER` flag is provided, system MUST set the Epic parent link field for all created tickets
- **FR-019**: When `--epic` flag is provided, system MUST validate that the Epic key exists in JIRA before creating tickets
- **FR-020**: System MUST infer JIRA issue type from task phase: Setup/Foundational phase tickets use "Task" type, User Story phase tickets use "Story" type
- **FR-021**: Description field MUST include: purpose, tasks covered (by ID), affected systems/files, workflows, stakeholders, corner cases
- **FR-022**: Test Plan field MUST describe verification approach for each major component in the ticket
- **FR-023**: Technical Details field MUST list: specific files/modules, functions to add/modify, implementation sequence, dependencies
- **FR-024**: System MUST handle missing or optional tasks.md sections gracefully (e.g., if no Test Plan section exists in tasks.md)
- **FR-025**: System MUST validate that tasks.md follows expected format before attempting to group

### Key Entities

- **Task**: Individual work item from tasks.md with ID, optional parallel marker [P], optional story label [US#], description, and file path
- **Task Group**: Collection of related tasks that form a JIRA-sized work unit (1-3 SP), organized by functional area or user story
- **JIRA Ticket**: Work item with 4-field structure (Subject, Description, Test Plan, Technical Details) ready for creation in JIRA
- **Epic**: JIRA issue type that serves as parent container for User Stories and Tasks, used for grouping related work and tracking timelines (optional parent for generated tickets)
- **Field Mapping**: Configuration that maps 4-field structure to JIRA issue fields (including custom field IDs)
- **Task-to-Ticket Mapping**: Traceability record linking original task IDs to created JIRA ticket keys
- **JIRA Ticket Template**: Markdown template file (templates/jira-ticket-template.md in repo, copied to .specify/templates/ during installation) defining canonical 4-field structure for individual tickets
- **JIRA Tickets Output Template**: Markdown template file (templates/jira-tickets-template.md in repo, copied to .specify/templates/ during installation) defining structure for generated jira-tickets.md output file with all tickets, mappings, and statistics

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can generate jira-tickets.md from tasks.md in under 30 seconds for typical features (50-100 tasks)
- **SC-002**: Generated tickets require minimal manual editing - 80% of tickets are usable as-generated without modifications
- **SC-003**: Each generated ticket contains all 4 required fields with substantive content (not placeholder text)
- **SC-004**: Task grouping produces 8-15 tickets for a typical feature, with average group size of 5-8 tasks
- **SC-005**: 90% of tasks are successfully grouped based on user story label or functional area
- **SC-006**: JIRA ticket creation via jira-db skill succeeds without errors when valid credentials and project info provided
- **SC-007**: Generated Description fields follow AI-friendly style and receive positive feedback from AI agents attempting to implement them
- **SC-008**: Users can trace from any JIRA ticket back to original tasks.md tasks via task IDs referenced in Description field
- **SC-009**: Command execution produces clear progress output showing: tasks loaded, groups formed, tickets generated, JIRA creation status
- **SC-010**: Dry-run mode accurately previews what tickets would be created without side effects

## Assumptions

- Command implemented as Claude Code slash command (markdown specification file) that Claude interprets at runtime, not as compiled Python/shell script
- Claude has natural language understanding sufficient to parse tasks.md format, group tasks by heuristics, and generate well-structured ticket content
- Users have already run `/speckit.tasks` to generate tasks.md before running `/speckit.jira-tasks`
- Feature follows standard Spec Kit directory structure (specs/###-feature-name/)
- tasks.md follows the standard template format with proper task IDs, story labels, and file paths
- When JIRA creation is requested, jira-db skill is properly configured with valid credentials and accessible via skill invocation
- JIRA instance has custom fields for Test Plan (customfield_10332) and Technical Details (customfield_10301)
- Task grouping targets work units suitable for 1-3 days of development effort (story points will be estimated by team during refinement)
- AI-friendly documentation style from ~/.claude/AI-FRIENDLY.md is applicable to JIRA ticket content
- Team names in jira-db cache are current and accurate for team assignment
- Users understand that this command is supplementary to `/speckit.tasks`, not a replacement

## Dependencies

- **Build System**: Command files must be added to templates/commands/ and integrated into GitHub release workflow (.github/workflows/release.yml and create-release-packages.sh) to be included in spec-kit distribution ZIPs
- **Distribution**: Command distributed via GitHub release ZIP (e.g., spec-kit-template-claude-sh-v{version}.zip) and installed during `specify init` into .claude/commands/ directory
- **Template Files**: New templates must be created in templates/ directory (jira-ticket-template.md and jira-tickets-template.md) and will be copied to .specify/templates/ during installation, following same pattern as tasks-template.md
- **Prerequisite**: `/speckit.tasks` must have been run to generate tasks.md
- **External**: jira-db skill must be available and configured (for P3 auto-creation feature)
- **External**: ~/.claude/AI-FRIENDLY.md must exist to guide documentation style
- **File**: tasks.md must exist in current feature's spec directory
- **File**: spec.md should exist to provide context about feature and user stories (optional but helpful)
- **Infrastructure**: JIRA instance must be accessible and have required custom fields configured
