# Tasks: JIRA Tasks Integration

**Input**: Design documents from `/specs/001-jira-tasks-integration/`
**Prerequisites**: plan.md (required), spec.md (required), research.md, data-model.md, quickstart.md

**Tests**: Tests are NOT requested in the specification. Manual verification against acceptance scenarios will be used.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

**UPDATED**: 2025-11-14 - Tasks regenerated based on clarification decisions (distribution via GitHub release workflow, extend check-prerequisites.sh, templates in repo's templates/ directory)

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3, US4)
- Include exact file paths in descriptions

## Path Conventions

This feature adds files to existing Spec Kit repository structure:
- **Template sources (repo)**: `templates/`, `templates/commands/`
- **Script modifications**: `scripts/bash/check-prerequisites.sh`
- **Workflow modifications**: `.github/workflows/`
- **Installed locations (user project)**: `.claude/commands/`, `.specify/templates/`

---

## Phase 1: Setup (Template Infrastructure)

**Purpose**: Create template files that will be distributed via GitHub release workflow

- [X] T001 Create jira-ticket-template.md defining 4-field structure (Subject, Description, Test Plan, Technical Details) in templates/jira-ticket-template.md
- [X] T002 [P] Create jira-tickets-template.md defining output file format with metadata, tickets, mapping table, statistics in templates/jira-tickets-template.md
- [X] T003 [P] Create command definition file jira-tasks.md with YAML frontmatter in templates/commands/jira-tasks.md

---

## Phase 2: Foundational (Script & Workflow Integration)

**Purpose**: Core infrastructure that MUST be complete before ANY user story implementation

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [X] T004 Modify check-prerequisites.sh to accept --for-jira-tasks flag in scripts/bash/check-prerequisites.sh
- [X] T005 Add JSON output fields TASKS_FILE and JIRA_TICKETS_FILE when --for-jira-tasks flag present in scripts/bash/check-prerequisites.sh
- [X] T006 [P] Modify release.yml to add templates/commands/jira-tasks.md to watch paths in .github/workflows/release.yml
- [X] T007 [P] Modify release.yml to add templates/jira-*.md to watch paths in .github/workflows/release.yml
- [X] T008 [P] Modify create-release-packages.sh to process new command template in .github/workflows/scripts/create-release-packages.sh

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - Generate JIRA-sized Work Units (Priority: P1) 🎯 MVP

**Goal**: Core task parsing and grouping functionality that converts tasks.md into jira-tickets.md with grouped work units

**Independent Test**: Run `/speckit.jira-tasks` on specs/001-jira-tasks-integration/tasks.md and verify it produces jira-tickets.md with 8-12 properly grouped tickets

### Implementation for User Story 1

- [X] T009 [US1] Add YAML frontmatter with scripts section (sh: scripts/bash/check-prerequisites.sh --json --for-jira-tasks) in templates/commands/jira-tasks.md
- [X] T010 [US1] Add Setup step (Step 1) to execute bash script and parse JSON output for FEATURE_DIR, TASKS_FILE, JIRA_TICKETS_FILE in templates/commands/jira-tasks.md
- [X] T011 [US1] Add task parsing logic (Step 2) to read tasks.md and extract Task entities (id, parallel, story_label, description, file_path, phase) in templates/commands/jira-tasks.md
- [X] T012 [US1] Implement Pass 1 of grouping algorithm - separate tasks by phase (setup/foundational/user_story/polish) in templates/commands/jira-tasks.md
- [X] T013 [US1] Implement Pass 2 of grouping algorithm - group user_story phase tasks by story label ([US1], [US2], etc.) in templates/commands/jira-tasks.md
- [X] T014 [US1] Implement Pass 3 of grouping algorithm - cluster by file path similarity within each story group in templates/commands/jira-tasks.md
- [X] T015 [US1] Implement Pass 4 of grouping algorithm - validate group sizes (target 5-8 tasks), split oversized groups in templates/commands/jira-tasks.md
- [X] T016 [US1] Implement Pass 5 of grouping algorithm - create TaskGroup entities with metadata (group_id, tasks, phase, story_label, task_ids) in templates/commands/jira-tasks.md
- [X] T017 [US1] Add file output logic (Step 3) to generate jira-tickets.md using jira-tickets-template.md structure in templates/commands/jira-tasks.md
- [X] T018 [US1] Add task-to-ticket mapping table generation with columns (Task IDs, Group ID, Phase, Story Label) in templates/commands/jira-tasks.md
- [X] T019 [US1] Add statistics generation (tickets by phase, tickets by story, average group size, grouping effectiveness) in templates/commands/jira-tasks.md
- [X] T020 [US1] Add progress output messages for each step (tasks loaded, groups formed, tickets generated, file written) in templates/commands/jira-tasks.md

**Checkpoint**: At this point, User Story 1 should be fully functional and testable independently - command generates jira-tickets.md with grouped tasks

---

## Phase 4: User Story 2 - AI-Friendly 4-Field Structure (Priority: P2)

**Goal**: Generate well-structured ticket content with all 4 fields following AI-friendly documentation principles

**Independent Test**: Examine generated jira-tickets.md and verify each ticket has Subject, Description, Test Plan, Technical Details with proper structure and explicit labeling

### Implementation for User Story 2

- [X] T021 [US2] Implement Subject generation (Step 4a) - create 50-100 char sentence with verb+object+purpose format from task descriptions in templates/commands/jira-tasks.md
- [X] T022 [US2] Implement Description field generation (Step 4b) with 6 subsections - Purpose, Tasks Covered (by ID), Affected Systems, Key Workflows, Stakeholders, Edge Cases in templates/commands/jira-tasks.md
- [X] T023 [US2] Implement Test Plan field generation (Step 4c) with 4 subsections - Verification Approach, Component Tests, Integration Test, Edge Case Validation in templates/commands/jira-tasks.md
- [X] T024 [US2] Implement Technical Details field generation (Step 4d) with 4 subsections - Files to Modify, Functions to Add/Modify, Implementation Sequence, Dependencies in templates/commands/jira-tasks.md
- [X] T025 [US2] Apply AI-friendly style principles - add bold labels for all subsection headers (**Purpose:**, **Tasks Covered:**, etc.) in templates/commands/jira-tasks.md
- [X] T026 [US2] Extract file paths from task descriptions and populate Affected Systems and Files to Modify subsections in templates/commands/jira-tasks.md
- [X] T027 [US2] Generate Implementation Sequence by ordering tasks within group based on dependencies and file relationships in templates/commands/jira-tasks.md
- [X] T028 [US2] Add cross-ticket dependency detection - scan for task dependencies spanning multiple groups, note in Technical Details/Dependencies in templates/commands/jira-tasks.md

**Checkpoint**: At this point, User Stories 1 AND 2 should both work independently - tickets have complete, AI-friendly content

---

## Phase 5: User Story 3 - Auto-Create via jira-db Skill (Priority: P3)

**Goal**: Automatic JIRA ticket creation using jira-db skill with team assignment and Epic association

**Independent Test**: Run `/speckit.jira-tasks --create --project PM_TEST --team "Test Team" --epic PM_TEST-1000` and verify tickets appear in JIRA with correct metadata

### Implementation for User Story 3

- [X] T029 [US3] Add flag parsing logic (Step 5a) - parse --dry-run, --create, --project, --team, --epic, --interactive flags from $ARGUMENTS in templates/commands/jira-tasks.md
- [X] T030 [US3] Add dry-run mode implementation - output preview of tickets with issue types and Epic associations without creating in JIRA in templates/commands/jira-tasks.md
- [X] T031 [US3] Implement Epic validation (Step 5b) - call jira-db skill to verify Epic key exists before creating tickets in templates/commands/jira-tasks.md
- [X] T032 [US3] Implement team resolution (Step 5c) - call jira-db skill to resolve team name to team_id in templates/commands/jira-tasks.md
- [X] T033 [US3] Implement issue type inference (Step 5d) - set "Task" for setup/foundational/polish phases, "Story" for user_story phase in templates/commands/jira-tasks.md
- [X] T034 [US3] Implement markdown-to-ADF conversion for Description field - support headings (##, ###), bold (**text**), lists (-, 1.), code blocks (```) in templates/commands/jira-tasks.md
- [X] T035 [US3] Implement markdown-to-ADF conversion for Test Plan field using same rules as Description in templates/commands/jira-tasks.md
- [X] T036 [US3] Implement markdown-to-ADF conversion for Technical Details field using same rules as Description in templates/commands/jira-tasks.md
- [X] T037 [US3] Implement JIRA ticket creation (Step 5e) - call jira-db skill with fields (project, summary, description ADF, issuetype, customfield_10332 ADF, customfield_10301 ADF, team_id, epic_link) in templates/commands/jira-tasks.md
- [X] T038 [US3] Capture JIRA ticket keys from jira-db responses and update jira-tickets.md with created keys in templates/commands/jira-tasks.md
- [X] T039 [US3] Add output mapping display - show "Tasks T007-T016 → PM-12345 [Story]" format for each created ticket in templates/commands/jira-tasks.md
- [X] T040 [US3] Implement error handling for jira-db skill unavailable - provide clear error with setup instructions in templates/commands/jira-tasks.md
- [X] T041 [US3] Implement error handling for team name not found - list available teams from jira-db cache in templates/commands/jira-tasks.md
- [X] T042 [US3] Implement error handling for invalid Epic key - fail fast with clear message suggesting verification in templates/commands/jira-tasks.md
- [X] T043 [US3] Implement warning for Epic from different project - note cross-project linkage may be intentional in templates/commands/jira-tasks.md
- [X] T044 [US3] Implement error handling for custom field IDs missing - provide guidance on configuring field mappings in templates/commands/jira-tasks.md

**Checkpoint**: All user stories 1-3 should now be independently functional - tickets can be generated and automatically created in JIRA

---

## Phase 6: User Story 4 - Interactive Review (Priority: P4)

**Goal**: Optional interactive mode for reviewing and modifying tickets before JIRA creation

**Independent Test**: Run `/speckit.jira-tasks --interactive --dry-run`, modify a ticket field, split a ticket, merge two tickets, and verify changes are saved

### Implementation for User Story 4

- [X] T045 [US4] Implement interactive mode detection - check for --interactive flag in parsed arguments in templates/commands/jira-tasks.md
- [X] T046 [US4] Implement ticket review loop (Step 6a) - present each ticket with Subject, Description snippet, ask Accept/Edit/Split/Merge/Skip/Quit in templates/commands/jira-tasks.md
- [X] T047 [US4] Implement Edit action - allow user to modify any of 4 fields (Subject, Description, Test Plan, Technical Details) in templates/commands/jira-tasks.md
- [X] T048 [US4] Implement Split action - prompt for split point (task ID), create two tickets, regenerate all 4 fields for both using task subsets in templates/commands/jira-tasks.md
- [X] T049 [US4] Implement Merge action - combine current ticket with next ticket, regenerate all 4 fields using combined task list in templates/commands/jira-tasks.md
- [X] T050 [US4] Implement Skip action - mark ticket as excluded from JIRA creation, note in jira-tickets.md in templates/commands/jira-tasks.md
- [X] T051 [US4] Update task-to-ticket mappings after split/merge operations - ensure all task IDs still accounted for in templates/commands/jira-tasks.md
- [X] T052 [US4] Write modified tickets back to jira-tickets.md after each change in templates/commands/jira-tasks.md

**Checkpoint**: All user stories should now be independently functional including optional interactive refinement

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Edge case handling, validation, documentation, and verification

- [X] T053 [P] Add edge case handling for empty tasks.md - detect and provide error message explaining /speckit.tasks must be run first in templates/commands/jira-tasks.md
- [X] T054 [P] Add edge case handling for tasks without file paths - flag as incomplete, warn in output in templates/commands/jira-tasks.md
- [X] T055 [P] Add edge case handling for very large task count (200+) - warn about feature scope, suggest splitting in templates/commands/jira-tasks.md
- [X] T056 [P] Add edge case handling for conflicting story labels - use first occurrence, note ambiguity in output in templates/commands/jira-tasks.md
- [X] T057 [P] Add edge case handling for mixed phase grouping - use majority phase for issue type, warn in output in templates/commands/jira-tasks.md
- [X] T058 [P] Add validation for tasks.md format - check for proper checklist structure before parsing in templates/commands/jira-tasks.md
- [X] T059 [P] Add absolute path validation - ensure all file paths from check-prerequisites.sh are absolute in templates/commands/jira-tasks.md
- [X] T060 [P] Document command usage in templates/commands/jira-tasks.md - add examples section with 7 usage patterns (dry-run, create, interactive, etc.)
- [X] T061 [P] Document jira-ticket-template.md structure - add comments explaining each field and subsection requirements in templates/jira-ticket-template.md
- [X] T062 [P] Document jira-tickets-template.md structure - add comments explaining output format and placeholder replacements in templates/jira-tickets-template.md
- [X] T063 [P] Add final summary output (Step 7) - display total tasks loaded, tickets generated, JIRA keys created (if any), output file path, next steps in templates/commands/jira-tasks.md
- [X] T064 [P] Test PowerShell version of prerequisite check - add ps: entry to YAML frontmatter for Windows support in templates/commands/jira-tasks.md
- [X] T065 Verify GitHub release workflow integration - test that release.yml triggers on template changes
- [X] T066 Verify create-release-packages.sh processes new command - test that ZIP contains jira-tasks command and templates in correct locations
- [ ] T067 Test end-to-end on this feature (001-jira-tasks-integration) - run /speckit.jira-tasks on specs/001-jira-tasks-integration/tasks.md
- [ ] T068 Verify generated jira-tickets.md has proper structure - check metadata, 4-field tickets, mapping table, statistics
- [ ] T069 Test with staging JIRA instance - run with --create flag, verify tickets created with correct issue types, Epic association, team assignment
- [X] T070 Update documentation - add usage examples to quickstart.md showing command execution patterns
- [ ] T071 Run verification-guard-quick agent to validate task completeness and format compliance

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3-6)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 → P2 → P3 → P4)
- **Polish (Phase 7)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational (Phase 2) - Independent of US1 but logically builds on ticket structure
- **User Story 3 (P3)**: Can start after Foundational (Phase 2) - Independent but references US1 and US2 ticket generation
- **User Story 4 (P4)**: Can start after Foundational (Phase 2) - Independent, optional refinement feature

### Within Each User Story

- Setup tasks before foundational
- Foundational before any user story
- Task parsing before grouping (T011 before T012-T016)
- Grouping before ticket generation (T012-T016 before T017)
- Ticket generation before content population (T017 before T021-T024)
- Content generation before JIRA integration (T021-T024 before T029-T044)
- Core functionality before interactive mode (T009-T044 before T045-T052)

### Parallel Opportunities

- All Setup tasks (T001, T002, T003) marked [P] can run in parallel
- All Foundational workflow modifications (T006, T007, T008) marked [P] can run in parallel
- Once Foundational phase completes, all 4 user stories can start in parallel (if team capacity allows)
- All Polish tasks marked [P] (T053-T062, T064) can run in parallel
- Different user stories can be worked on in parallel by different team members

---

## Implementation Strategy

### MVP First (User Story 1 + User Story 2)

1. Complete Phase 1: Setup (T001-T003)
2. Complete Phase 2: Foundational (T004-T008) - CRITICAL, blocks all stories
3. Complete Phase 3: User Story 1 (T009-T020) - Core grouping functionality
4. Complete Phase 4: User Story 2 (T021-T028) - AI-friendly ticket content
5. **STOP and VALIDATE**: Test on specs/001-jira-tasks-integration/tasks.md, verify jira-tickets.md output
6. Deploy/demo if ready - command can generate tickets offline

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 → Test independently → MVP ready (file generation only)
3. Add User Story 2 → Test independently → Enhanced content quality
4. Add User Story 3 → Test independently → JIRA automation enabled
5. Add User Story 4 → Test independently → Interactive refinement available
6. Each story adds value without breaking previous stories

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together (T001-T008)
2. Once Foundational is done:
   - Developer A: User Story 1 (T009-T020)
   - Developer B: User Story 2 (T021-T028) - starts templates/jira-ticket-template.md work
   - Developer C: User Story 3 (T029-T044) - research jira-db skill integration
   - Developer D: Polish tasks (T053-T062) - edge cases and validation
3. Stories complete and integrate independently
4. Final integration testing (T065-T069)

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- Templates in `templates/` directory (repo source) get copied to `.specify/templates/` during installation
- Command in `templates/commands/` (repo source) gets copied to `.claude/commands/` during installation
- Release workflow automatically handles distribution when templates change
- Based on clarifications from 2025-11-14 session (distribution, script extension, implementation approach)
