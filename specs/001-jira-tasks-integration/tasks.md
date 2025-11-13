# Tasks: JIRA Tasks Integration

**Input**: Design documents from `/specs/001-jira-tasks-integration/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, quickstart.md

**Tests**: Tests are NOT requested in the specification. Manual verification against acceptance scenarios will be used.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3, US4)
- Include exact file paths in descriptions

## Path Conventions

This feature adds files to existing Spec Kit repository structure:
- **Commands**: `.claude/commands/`
- **Templates**: `.specify/templates/`
- **Scripts**: `.specify/scripts/bash/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Create template files and setup script infrastructure

- [X] T001 Create jira-ticket-template.md defining 4-field structure in .specify/templates/jira-ticket-template.md
- [X] T002 [P] Create jira-tickets-template.md defining output file format in .specify/templates/jira-tickets-template.md
- [X] T003 [P] Create jira-tasks-setup.sh bash script for path validation in .specify/scripts/bash/jira-tasks-setup.sh

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core command file structure that user stories will build upon

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [X] T004 Create speckit.jira-tasks.md command file with frontmatter and basic structure in .claude/commands/speckit.jira-tasks.md
- [X] T005 Add User Input section to capture optional arguments ($ARGUMENTS) in .claude/commands/speckit.jira-tasks.md
- [X] T006 Add Outline section with 5-step workflow structure in .claude/commands/speckit.jira-tasks.md

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - Generate JIRA-sized Work Units (Priority: P1) 🎯 MVP

**Goal**: Convert granular tasks from tasks.md into JIRA-sized work tickets (8-12 tickets for typical feature)

**Independent Test**: Run `/speckit.jira-tasks` on existing tasks.md and verify jira-tickets.md contains properly grouped tickets with task IDs preserved for traceability

### Implementation for User Story 1

- [X] T007 [P] [US1] Implement Setup step: Run jira-tasks-setup.sh and parse JSON paths in .claude/commands/speckit.jira-tasks.md
- [X] T008 [P] [US1] Implement tasks.md parsing logic: Read file, extract task checklist items in .claude/commands/speckit.jira-tasks.md
- [X] T009 [US1] Implement Task entity parsing: Extract ID, parallel marker, story label, description, file path in .claude/commands/speckit.jira-tasks.md
- [X] T010 [US1] Implement phase detection logic: Infer setup/foundational/user_story/polish from context in .claude/commands/speckit.jira-tasks.md
- [X] T011 [US1] Implement Pass 1 grouping: Separate tasks by phase (setup, foundational, polish) in .claude/commands/speckit.jira-tasks.md
- [X] T012 [US1] Implement Pass 2 grouping: Group user_story phase tasks by story label [US1], [US2], etc. in .claude/commands/speckit.jira-tasks.md
- [X] T013 [US1] Implement Pass 3 clustering: Within each story, cluster by file path similarity in .claude/commands/speckit.jira-tasks.md
- [X] T014 [US1] Implement Pass 4 validation: Enforce 5-8 task target per group, preserve task order in .claude/commands/speckit.jira-tasks.md
- [X] T015 [US1] Implement TaskGroup entity creation with group_id, tasks array, phase, story_label attributes in .claude/commands/speckit.jira-tasks.md
- [X] T016 [US1] Add grouping statistics output: "12 groups formed - Setup: 1, Foundational: 1, US1: 3, US2: 4, US3: 2, Polish: 1" in .claude/commands/speckit.jira-tasks.md

**Checkpoint**: At this point, User Story 1 should be fully functional - command can group tasks and output shows grouping statistics

---

## Phase 4: User Story 2 - Create Tickets with AI-Friendly 4-Field Structure (Priority: P2)

**Goal**: Generate ticket content following AI-friendly documentation style (Subject, Description, Test Plan, Technical Details)

**Independent Test**: Examine generated jira-tickets.md and verify each ticket has all 4 fields with content following AI-friendly principles (explicit labeling, reduced ambiguity, canonical terminology)

### Implementation for User Story 2

- [X] T017 [P] [US2] Implement Subject generation: Single clear sentence from TaskGroup context in .claude/commands/speckit.jira-tasks.md
- [X] T018 [P] [US2] Implement Description field generation with AI-friendly structure (Purpose, Tasks Covered, Affected Systems, etc.) in .claude/commands/speckit.jira-tasks.md
- [X] T019 [P] [US2] Implement Test Plan field generation with verification approach and component tests in .claude/commands/speckit.jira-tasks.md
- [X] T020 [P] [US2] Implement Technical Details field generation with files, functions, implementation sequence in .claude/commands/speckit.jira-tasks.md
- [X] T021 [US2] Load AI-friendly documentation style from ~/.claude/AI-FRIENDLY.md for reference in .claude/commands/speckit.jira-tasks.md
- [X] T022 [US2] Apply AI-friendly principles: Explicit labeling with bold markers (**Purpose:**, **Files to Modify:**) in .claude/commands/speckit.jira-tasks.md
- [X] T023 [US2] Apply AI-friendly principles: Reduced ambiguity using structured lists instead of prose in .claude/commands/speckit.jira-tasks.md
- [X] T024 [US2] Apply AI-friendly principles: Canonical terminology (consistent use of "task ID", "file path", etc.) in .claude/commands/speckit.jira-tasks.md
- [X] T025 [US2] Create JiraTicket entities from TaskGroups with all 4 fields populated in .claude/commands/speckit.jira-tasks.md
- [X] T026 [US2] Infer issue_type from phase: setup/foundational/polish → "Task", user_story → "Story" in .claude/commands/speckit.jira-tasks.md
- [X] T027 [US2] Populate task_ids array with original task IDs from TaskGroup in .claude/commands/speckit.jira-tasks.md
- [X] T028 [US2] Load jira-tickets-template.md and fill with generated ticket content in .claude/commands/speckit.jira-tasks.md
- [X] T029 [US2] Write jira-tickets.md output file to FEATURE_DIR using template structure in .claude/commands/speckit.jira-tasks.md
- [X] T030 [US2] Add ticket generation progress output: "12 tickets created" in .claude/commands/speckit.jira-tasks.md

**Checkpoint**: At this point, User Story 2 should be fully functional - command generates jira-tickets.md with AI-friendly 4-field structure

---

## Phase 5: User Story 3 - Auto-Create JIRA Tickets via jira-db Skill (Priority: P3)

**Goal**: Automatically create tickets in JIRA when --create flag is provided, with team assignment and Epic association

**Independent Test**: Run `/speckit.jira-tasks --create --project PM --team "Team Name" --epic PM-1000` and verify tickets appear in JIRA with correct content, team assignment, issue types, and Epic parent link

### Implementation for User Story 3

- [X] T031 [P] [US3] Parse command flags: --create, --project, --team, --epic, --dry-run in .claude/commands/speckit.jira-tasks.md
- [X] T032 [P] [US3] Implement dry-run mode: Show what would be created without JIRA side effects in .claude/commands/speckit.jira-tasks.md
- [X] T033 [US3] Validate --epic flag: Check Epic exists in JIRA before ticket creation via jira-db skill in .claude/commands/speckit.jira-tasks.md
- [X] T034 [US3] Warn if Epic belongs to different project than --project flag in .claude/commands/speckit.jira-tasks.md
- [X] T035 [US3] Resolve team name to team ID using jira-db skill's team resolution capability in .claude/commands/speckit.jira-tasks.md
- [X] T036 [US3] Handle team not found: List available teams from jira-db cache in .claude/commands/speckit.jira-tasks.md
- [X] T037 [US3] Convert Description markdown to ADF (Atlassian Document Format) for JIRA field in .claude/commands/speckit.jira-tasks.md
- [X] T038 [US3] Convert Test Plan markdown to ADF for customfield_10332 in .claude/commands/speckit.jira-tasks.md
- [X] T039 [US3] Convert Technical Details markdown to ADF for customfield_10301 in .claude/commands/speckit.jira-tasks.md
- [X] T040 [US3] Map JiraTicket fields to JIRA API format: subject→summary, description→description (ADF) in .claude/commands/speckit.jira-tasks.md
- [X] T041 [US3] Set Epic Link field when --epic flag provided (all tickets except Setup phase) in .claude/commands/speckit.jira-tasks.md
- [X] T042 [US3] Create tickets via jira-db skill with correct issue type (Task vs Story) in .claude/commands/speckit.jira-tasks.md
- [X] T043 [US3] Assign tickets to resolved team ID in .claude/commands/speckit.jira-tasks.md
- [X] T044 [US3] Capture JIRA ticket keys (PM-12345, PM-12346, etc.) from creation response in .claude/commands/speckit.jira-tasks.md
- [X] T045 [US3] Update JiraTicket entities with jira_key after creation in .claude/commands/speckit.jira-tasks.md
- [X] T046 [US3] Handle jira-db skill unavailable: Graceful error with setup instructions in .claude/commands/speckit.jira-tasks.md
- [X] T047 [US3] Handle partial creation failure: Report which tickets succeeded/failed in .claude/commands/speckit.jira-tasks.md
- [X] T048 [US3] Update jira-tickets.md with JIRA ticket keys in mapping table in .claude/commands/speckit.jira-tasks.md
- [X] T049 [US3] Add creation progress output: "Created PM-12345 [Story]: Subject (Tasks T001-T008)" in .claude/commands/speckit.jira-tasks.md
- [X] T050 [US3] Generate task-to-ticket mapping output showing all created tickets in .claude/commands/speckit.jira-tasks.md

**Checkpoint**: At this point, User Story 3 should be fully functional - command can create tickets in JIRA with all metadata correct

---

## Phase 6: User Story 4 - Interactive Review and Refinement (Priority: P4)

**Goal**: Allow review and modification of ticket content before JIRA creation

**Independent Test**: Run with --interactive flag, modify ticket content when prompted, verify changes preserved in jira-tickets.md and created JIRA tickets

### Implementation for User Story 4

- [X] T051 [P] [US4] Parse --interactive flag from command arguments in .claude/commands/speckit.jira-tasks.md
- [X] T052 [P] [US4] Present each ticket for review with numbered menu: 1) Accept, 2) Edit, 3) Split, 4) Merge in .claude/commands/speckit.jira-tasks.md
- [X] T053 [US4] Implement Edit option: Allow editing each field (Subject, Description, Test Plan, Technical Details) in .claude/commands/speckit.jira-tasks.md
- [X] T054 [US4] Implement Split option: Allow breaking ticket into multiple smaller tickets in .claude/commands/speckit.jira-tasks.md
- [X] T055 [US4] Implement Merge option: Allow combining tickets with manual task ID reassignment in .claude/commands/speckit.jira-tasks.md
- [X] T056 [US4] Update JiraTicket entities with modified content from user edits in .claude/commands/speckit.jira-tasks.md
- [X] T057 [US4] Update TaskToTicketMapping when tickets are split or merged in .claude/commands/speckit.jira-tasks.md
- [X] T058 [US4] Write modified content back to jira-tickets.md before proceeding to creation in .claude/commands/speckit.jira-tasks.md
- [X] T059 [US4] Proceed with JIRA creation using final reviewed content if --create flag also provided in .claude/commands/speckit.jira-tasks.md

**Checkpoint**: At this point, User Story 4 should be fully functional - command supports interactive ticket review and editing

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Error handling, edge cases, documentation, and user experience improvements

- [X] T060 [P] Add error handling for missing tasks.md: "tasks.md not found. Run /speckit.tasks first." in .claude/commands/speckit.jira-tasks.md
- [X] T061 [P] Add error handling for invalid task format: Warn about malformed tasks, continue with valid ones in .claude/commands/speckit.jira-tasks.md
- [X] T062 [P] Add error handling for tasks without file paths: Flag as incomplete, suggest adding detail in .claude/commands/speckit.jira-tasks.md
- [X] T063 [P] Add warning for very large task count (200+): Suggest breaking into multiple features in .claude/commands/speckit.jira-tasks.md
- [X] T064 [P] Add handling for conflicting story labels: Use first occurrence, note ambiguity in output in .claude/commands/speckit.jira-tasks.md
- [X] T065 [P] Add handling for mixed phase grouping: Use majority phase for issue type in .claude/commands/speckit.jira-tasks.md
- [X] T066 [P] Add handling for cross-ticket dependencies: Note in Technical Details field in .claude/commands/speckit.jira-tasks.md
- [X] T067 [P] Document command usage in frontmatter description in .claude/commands/speckit.jira-tasks.md
- [X] T068 [P] Add examples section showing common usage patterns (basic, dry-run, with Epic) in .claude/commands/speckit.jira-tasks.md
- [X] T069 [P] Add clear progress indicators for each workflow step in .claude/commands/speckit.jira-tasks.md
- [X] T070 Validate all file paths are absolute (cross-platform compatibility) in .claude/commands/speckit.jira-tasks.md
- [X] T071 Add final summary output: tasks loaded, tickets generated, JIRA keys if created in .claude/commands/speckit.jira-tasks.md

---

## Dependencies

### User Story Dependencies

```text
Phase 1 (Setup) → Phase 2 (Foundational) → All User Stories can run in parallel

User Story 1 (P1): Independent - No dependencies
  ↓ provides: Grouped tasks and jira-tickets.md structure

User Story 2 (P2): Depends on P1 completing
  ↓ provides: 4-field AI-friendly ticket content

User Story 3 (P3): Depends on P2 completing
  ↓ provides: JIRA ticket creation capability

User Story 4 (P4): Depends on P2 completing (can run parallel to P3)
  ↓ provides: Interactive review before creation
```

### Story Completion Order

**Minimum Viable Product (MVP)**: P1 + P2
- Delivers ticket generation with AI-friendly structure
- Manual JIRA creation using jira-tickets.md content

**Full Automation**: P1 + P2 + P3
- Adds automatic JIRA ticket creation
- Most valuable for frequent use

**Enhanced UX**: P1 + P2 + P3 + P4
- Adds interactive review capability
- Best for quality-sensitive workflows

---

## Parallel Execution Examples

### Within User Story 1 (Task Grouping)

These tasks can run in parallel after T006 completes:
```text
T007 [Setup step implementation]
T008 [Parsing logic]
```

After T008-T010 complete, these can run in parallel:
```text
T011 [Phase-based grouping]
T012 [Story label grouping]
T013 [File similarity clustering]
```

### Within User Story 2 (Ticket Generation)

After T016 completes, these field generation tasks can run in parallel:
```text
T017 [Subject generation]
T018 [Description generation]
T019 [Test Plan generation]
T020 [Technical Details generation]
T021 [Load AI-friendly style guide]
```

After T021-T024 complete, these can run in parallel:
```text
T022 [Apply explicit labeling]
T023 [Apply reduced ambiguity]
T024 [Apply canonical terminology]
```

### Within User Story 3 (JIRA Integration)

After T030 completes, these flag parsing and validation tasks can run in parallel:
```text
T031 [Parse command flags]
T032 [Dry-run mode]
T033 [Epic validation]
```

After T035 completes, these ADF conversion tasks can run in parallel:
```text
T037 [Convert Description to ADF]
T038 [Convert Test Plan to ADF]
T039 [Convert Technical Details to ADF]
```

### Within User Story 4 (Interactive Review)

After T050 completes, these UI option implementations can run in parallel:
```text
T052 [Review menu]
T053 [Edit option]
T054 [Split option]
T055 [Merge option]
```

### Polish Phase

After all user stories complete, these tasks can ALL run in parallel:
```text
T060 [Missing tasks.md error]
T061 [Invalid format error]
T062 [Missing file paths error]
T063 [Large task count warning]
T064 [Conflicting labels handling]
T065 [Mixed phase handling]
T066 [Cross-ticket dependencies]
T067 [Documentation]
T068 [Examples]
T069 [Progress indicators]
```

---

## Implementation Strategy

### MVP First (P1 + P2)

**Deliverable**: Command that generates jira-tickets.md with AI-friendly structure

**Value**: Automates 80% of the work (grouping and content generation)

**Tasks**: T001-T030 (30 tasks)

**Timeline**: ~3-5 days for core functionality

### Full Automation (Add P3)

**Deliverable**: Command that creates tickets directly in JIRA

**Value**: Eliminates all manual work for ticket creation

**Tasks**: T031-T050 (20 additional tasks)

**Timeline**: ~2-3 days for JIRA integration

### Enhanced UX (Add P4)

**Deliverable**: Interactive review before creation

**Value**: Quality control for ticket content

**Tasks**: T051-T059 (9 additional tasks)

**Timeline**: ~1-2 days for interactive features

### Polish (Cross-Cutting)

**Deliverable**: Production-ready error handling and UX

**Tasks**: T060-T071 (12 tasks)

**Timeline**: ~1-2 days for polish

### Total Scope

**Total Tasks**: 71
- Setup: 3 tasks
- Foundational: 3 tasks
- P1 (MVP core): 10 tasks
- P2 (AI-friendly structure): 14 tasks
- P3 (JIRA creation): 20 tasks
- P4 (Interactive): 9 tasks
- Polish: 12 tasks

**Estimated Timeline**:
- MVP (P1+P2): 3-5 days
- Full automation (+P3): 5-8 days total
- Complete feature (+P4+Polish): 7-12 days total

**Parallel Opportunities**: ~35 tasks marked [P] (49% parallelizable)
