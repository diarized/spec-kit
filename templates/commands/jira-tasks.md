---
description: Generate JIRA-sized work tickets from tasks.md with AI-friendly 4-field structure, optionally create in JIRA via jira-db skill
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --for-jira-tasks
  ps: scripts/powershell/check-prerequisites.ps1 -Json -ForJiraTasks
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

This command converts granular AI-friendly tasks from tasks.md into JIRA-sized work tickets (approximately 1-3 days of work). It groups related tasks, generates tickets with a 4-field AI-friendly structure, and optionally creates them in JIRA.

**Execution Steps**:

1. **Setup**: Run prerequisite script and parse paths
2. **Parse Flags**: Extract command-line flags from $ARGUMENTS
3. **Parse tasks.md**: Read and extract Task entities
4. **Group Tasks**: Apply 5-pass grouping algorithm
5. **Generate Tickets**: Create tickets with 4-field structure
6. **Write Output**: Generate jira-tickets.md file
7. **Interactive Review** (optional): Allow ticket editing
8. **Create in JIRA** (optional): Call jira-db skill
9. **Report Summary**: Display results and next steps

---

## Step 1: Setup and Prerequisite Check

**Purpose**: Validate prerequisites and get absolute file paths

Run the prerequisite script (already executed via YAML frontmatter) and parse the JSON output:

**Expected JSON Output**:

```json
{
  "FEATURE_DIR": "/absolute/path/to/specs/###-feature-name",
  "TASKS_FILE": "/absolute/path/to/specs/###-feature-name/tasks.md",
  "JIRA_TICKETS_FILE": "/absolute/path/to/specs/###-feature-name/jira-tickets.md",
  "FEATURE_NUM": "###",
  "FEATURE_NAME": "feature-name",
  "BRANCH": "###-feature-name"
}
```

**Actions**:

1. Parse JSON output from prerequisite script
2. Store paths in variables: `FEATURE_DIR`, `TASKS_FILE`, `JIRA_TICKETS_FILE`, `FEATURE_NUM`, `FEATURE_NAME`
3. Verify `TASKS_FILE` exists and is readable
4. If tasks.md missing or empty, ERROR: "tasks.md not found. Run `/speckit.tasks` first to generate task breakdown."

**Error Handling**:

- Missing tasks.md → Clear error with actionable message
- Unreadable tasks.md → Check file permissions
- Invalid JSON from script → Report script error

---

## Step 2: Parse Command-Line Flags

**Purpose**: Extract user-specified flags from $ARGUMENTS

**Supported Flags**:

- `--dry-run`: Preview ticket generation without creating in JIRA (default behavior if no --create)
- `--create`: Create tickets in JIRA via jira-db skill (requires --project)
- `--project PROJECT_KEY`: JIRA project key (required with --create)
- `--team "Team Name"`: Assign tickets to team (optional, resolves to team_id)
- `--epic EPIC_KEY`: Associate all tickets with Epic parent (optional, validates existence)
- `--interactive`: Review and edit tickets before finalizing (optional)

**Parsing Logic**:

- Parse $ARGUMENTS string to extract flags and values
- Handle flags with values: `--project PM`, `--team "Platform Team"`, `--epic PM-1000`
- Handle boolean flags: `--dry-run`, `--create`, `--interactive`
- Default to dry-run mode if `--create` not specified
- If `--create` specified without `--project`, ERROR: "--create requires --project flag"

**Flag Validation**:

- PROJECT_KEY format: Alphanumeric, typically 2-10 characters
- EPIC_KEY format: PROJECT_KEY-NUMBER (e.g., PM-1000)
- Team name: Any string (resolved via jira-db later)

**Store in Variables**:

```text
DRY_RUN = true/false
CREATE_IN_JIRA = true/false
PROJECT_KEY = string or null
TEAM_NAME = string or null
EPIC_KEY = string or null
INTERACTIVE = true/false
```

---

## Step 3: Parse tasks.md and Extract Task Entities

**Purpose**: Read tasks.md and parse into structured Task entities

**Task Format** (from tasks.md):

```markdown
- [ ] T012 [P] [US1] Create User model in src/models/user.py
```

**Task Entity Structure**:

```text
Task {
  id: "T012"
  checkbox: false (always unchecked for grouping)
  parallel: true (has [P] marker)
  story_label: "US1" (or null if no story label)
  description: "Create User model in src/models/user.py"
  file_path: "src/models/user.py" (extracted from description)
  phase: "user_story" (inferred from section header or story label)
  line_number: 42 (for traceability)
}
```

**Parsing Steps**:

1. Read `TASKS_FILE` line by line
2. Track current phase based on section headers:
   - "## Phase 1: Setup" → phase = "setup"
   - "## Phase 2: Foundational" → phase = "foundational"
   - "## Phase 3: User Story 1" → phase = "user_story"
   - "## Phase 7: Polish" → phase = "polish"
3. For each line matching `- [ ] T\d{3}`:
   - Extract task ID (T\d{3})
   - Check for [P] marker (parallel)
   - Extract story label if present: `\[US\d+\]`
   - Extract description (everything after markers)
   - Extract file path from description (last path-like token or text in backticks)
   - Assign current phase
   - Record line number
4. Store all tasks in `tasks` array

**Validation**:

- Warn if task description missing file path: "Task T### missing file path - may impact Technical Details generation"
- Error if no tasks found: "tasks.md contains no tasks or improper format. Expected: `- [ ] T### [P?] [Story?] Description with file path`"
- Warn if very large task count (>200): "Feature has 200+ tasks. Consider splitting into multiple features."

**Output**:

- Array of Task entities: `tasks[]`
- Total task count: `TOTAL_TASKS`
- Progress message: "Loaded [COUNT] tasks from tasks.md"

---

## Step 4: Task Grouping Algorithm (5-Pass Multi-Dimensional)

**Purpose**: Group tasks into JIRA-sized work units (5-8 tasks per ticket, approximately 1-3 days)

**Grouping Constraints**:

- Phase boundaries: Never mix setup, foundational, user_story, polish phases
- Story affinity: Tasks with same story label stay together
- File locality: Tasks modifying same files grouped together
- Size target: 5-8 tasks per ticket optimal
- Split oversized: If group >10 tasks, split into multiple tickets

### Pass 1: Separate by Phase

**Purpose**: Create phase-level buckets (phase boundaries are hard constraints)

**Logic**:

1. Create 4 buckets: `setup_tasks[]`, `foundational_tasks[]`, `user_story_tasks[]`, `polish_tasks[]`
2. Iterate through all tasks, assign to bucket based on `task.phase`
3. Each bucket becomes independent grouping scope for subsequent passes

**Output**: 4 phase buckets with tasks

### Pass 2: Group User Story Phase by Story Label

**Purpose**: Within user_story phase, group by story label ([US1], [US2], etc.)

**Logic**:

1. Focus on `user_story_tasks[]` bucket only
2. Create story groups: Group tasks with same `story_label`
3. Keep story groups as primary organization unit
4. Tasks without story_label: Create separate "Unlabeled" group

**Example**:

- US1 group: All tasks with story_label = "US1"
- US2 group: All tasks with story_label = "US2"
- Unlabeled group: Tasks with story_label = null

**Output**: Story-based groups within user_story phase

### Pass 3: Cluster by File Path Similarity

**Purpose**: Within each story group, cluster by file path to group related work

**Logic**:

1. For each story group from Pass 2:
2. Extract all unique file paths mentioned in task descriptions
3. Calculate file path similarity:
   - Same file = highest affinity
   - Same directory = high affinity
   - Same parent directory = medium affinity
   - Different paths = low affinity
4. Create sub-clusters of tasks with high file path affinity
5. Keep cluster size under 10 tasks

**Clustering Algorithm**:

- Start with first task in group
- Add tasks that share file paths or directories
- When cluster reaches 8 tasks or no more similar tasks, start new cluster
- Preserve original task ordering within clusters

**Output**: File-based clusters within each story group

### Pass 4: Validate and Split Oversized Groups

**Purpose**: Ensure no ticket contains too many tasks (hard limit: 12 tasks)

**Logic**:

1. For each cluster from Pass 3:
2. If cluster size ≤ 12 tasks: Keep as-is
3. If cluster size > 12 tasks:
   - Split cluster at logical boundary (different files, different sub-features)
   - Aim for roughly equal splits
   - If uniform work (same file, similar tasks), split evenly
4. Target range after split: 5-8 tasks per ticket

**Splitting Strategy**:

- Prefer splitting at file boundaries (different files → different tickets)
- Respect sequential dependencies (don't split dependent tasks)
- Keep parallel tasks [P] in same or different tickets as space allows

**Output**: Size-validated clusters (no cluster >12 tasks)

### Pass 5: Create TaskGroup Entities

**Purpose**: Formalize clusters as TaskGroup entities with metadata

**TaskGroup Structure**:

```text
TaskGroup {
  group_id: "G01" (sequential: G01, G02, G03, ...)
  tasks: [Task, Task, Task, ...] (ordered list)
  phase: "user_story" (or setup/foundational/polish)
  story_label: "US1" (or null)
  task_count: 7
  task_ids: ["T009", "T010", "T011", ...]
  primary_files: ["templates/commands/jira-tasks.md", "templates/jira-ticket-template.md"]
  issue_type: "Story" (or "Task" based on phase)
}
```

**Logic**:

1. For each cluster from Pass 4:
2. Assign sequential group_id (G01, G02, etc.)
3. Store tasks array
4. Extract phase from first task (all tasks same phase by Pass 1)
5. Extract story_label from first task (all tasks same label by Pass 2, or null)
6. Count tasks
7. Collect all task IDs
8. Extract unique file paths (up to 3 most common)
9. Infer issue_type:
   - If phase = "setup" or "foundational" or "polish" → issue_type = "Task"
   - If phase = "user_story" → issue_type = "Story"
10. Store TaskGroup in `ticket_groups[]` array

**Edge Case Handling**:

- Mixed phase grouping (should not occur after Pass 1, but warn if detected)
- Conflicting story labels (use first occurrence, warn)
- Tasks without file paths (note in Technical Details that paths missing)

**Output**:

- Array of TaskGroup entities: `ticket_groups[]`
- Total ticket count: `TOTAL_TICKETS`
- Progress message: "Grouped [TASK_COUNT] tasks into [TICKET_COUNT] tickets"

---

## Step 5: Generate Ticket Content (4-Field Structure)

**Purpose**: For each TaskGroup, generate Subject, Description, Test Plan, Technical Details

**For each TaskGroup in ticket_groups[]**:

### Generate Subject Field

**Format**: 50-100 characters, verb+object+purpose

**Logic**:

1. Identify primary action from task descriptions (common verbs: Implement, Create, Add, Build, Modify)
2. Identify primary object (what's being built: algorithm, template, integration, feature)
3. Identify purpose (why it's needed: for ticket generation, to enable JIRA creation, etc.)
4. Combine: "[Verb] [Object] [Purpose]"
5. Trim to under 100 characters

**Example Generation**:

- Tasks: T012-T016 all about grouping algorithm passes
- Verb: "Implement"
- Object: "task grouping algorithm"
- Purpose: "for JIRA ticket generation"
- Result: "Implement task grouping algorithm for JIRA ticket generation"

### Generate Description Field

**Structure**: 6 subsections with bold labels

**Purpose Subsection**:

- Extract why from task descriptions and user story context
- 1-2 sentences explaining value
- Example: "This work implements the core grouping logic that converts 50-100 granular tasks into 8-15 JIRA-sized tickets, enabling project managers to track work in JIRA while preserving detailed breakdowns for AI agents."

**Tasks Covered Subsection**:

- List all task IDs from TaskGroup.task_ids
- Format: Comma-separated or bullet list
- Include brief description for each task
- Example: "T012: Pass 1 - separate by phase, T013: Pass 2 - group by story label, ..."

**Affected Systems Subsection**:

- Extract unique file paths from tasks
- List primary systems/modules/components touched
- Example: "templates/commands/jira-tasks.md (grouping algorithm), Task entity parser, TaskGroup entity generator"

**Key Workflows Subsection**:

- Describe user interaction or system flow
- Extract from task descriptions and spec.md context
- Example: "Command reads tasks.md, parses Task entities, applies 5-pass grouping, generates TaskGroup entities, outputs to jira-tickets.md"

**Stakeholders Subsection**:

- Identify user roles from spec.md user stories
- Match ticket phase/story to relevant stakeholders
- Example: "Project managers (ticket creation), Developers (implementation guidance), Technical leads (review)"

**Edge Cases Subsection**:

- Extract edge cases from task descriptions or spec.md
- List scenarios to handle
- Example: "Empty tasks.md → error, Tasks without file paths → warning, Large task count (200+) → warning"

### Generate Test Plan Field

**Structure**: 4 subsections with bold labels

**Verification Approach Subsection**:

- High-level testing strategy
- Based on user story independent test criteria
- Example: "Manual verification by running `/speckit.jira-tasks` on this feature's tasks.md, examining generated jira-tickets.md for proper grouping and structure"

**Component Tests Subsection**:

- List components or functions to test
- Specify expected behavior
- Extract from task descriptions
- Example: "Task parsing: Verify all tasks parsed with id, parallel, story_label, file_path. Pass 1: Verify phase separation. Pass 2: Verify story grouping."

**Integration Test Subsection**:

- End-to-end scenario
- Specify inputs and outputs
- Example: "Run `/speckit.jira-tasks` on specs/001-jira-tasks-integration/tasks.md, verify output contains 8-12 tickets with 4 fields each"

**Edge Case Validation Subsection**:

- Test edge cases from Description field
- Specify expected outcomes
- Example: "Test empty tasks.md → verify error message. Test 200+ tasks → verify warning message."

### Generate Technical Details Field

**Structure**: 4 subsections with bold labels

**Files to Modify Subsection**:

- List unique file paths from TaskGroup.primary_files
- Add brief purpose for each
- Example: "templates/commands/jira-tasks.md - Add grouping algorithm. templates/jira-ticket-template.md - Define structure."

**Functions to Add/Modify Subsection**:

- Extract action items from task descriptions
- List steps or sections to implement
- Example: "Add Pass 1 section for phase separation. Add Pass 2 section for story grouping. Implement TaskGroup entity creation."

**Implementation Sequence Subsection**:

- Order tasks by dependencies
- Number steps 1, 2, 3
- Respect task ordering from tasks.md
- Example: "1. Parse tasks.md. 2. Apply Pass 1 (phase separation). 3. Apply Pass 2 (story grouping). 4. Create TaskGroup entities."

**Dependencies Subsection**:

- External: jira-db skill (if JIRA integration tasks)
- Internal: Prerequisites from other tasks
- Data: Required file formats
- Cross-ticket: Dependencies on other tickets in this batch
- Example: "Internal: tasks.md must exist. Data: Task format `- [ ] T### [P?] [Story?] Description`. Cross-ticket: Ticket G02 depends on G01 grouping algorithm."

**AI-Friendly Style**:

- Use bold labels for all subsection headers
- Write in bullet points or short paragraphs
- Provide explicit structure
- Avoid ambiguous language
- Use canonical terminology

**Output**:

- Each TaskGroup now has:
  - subject: string
  - description: markdown string (6 subsections)
  - test_plan: markdown string (4 subsections)
  - technical_details: markdown string (4 subsections)

**Progress Message**: "Generated ticket content for [TICKET_COUNT] tickets"

---

## Step 6: Write Output File (jira-tickets.md)

**Purpose**: Generate jira-tickets.md using jira-tickets-template.md structure

**Template Location**: `.specify/templates/jira-tickets-template.md` (or templates/jira-tickets-template.md in repo)

**Placeholder Replacement**:

**Header Section**:

- `[FEATURE_NAME]` → FEATURE_NAME from Step 1
- `[TIMESTAMP]` → Current timestamp (YYYY-MM-DD HH:MM:SS)
- `[TASKS_FILE_PATH]` → TASKS_FILE from Step 1
- `[FEATURE_NUMBER]` → FEATURE_NUM from Step 1
- `[TOTAL_TASKS]` → TOTAL_TASKS from Step 3
- `[TOTAL_TICKETS]` → TOTAL_TICKETS from Step 4

**For Each Ticket**:

- `[GROUP_ID]` → TaskGroup.group_id
- `[TICKET_SUBJECT]` → TaskGroup.subject
- `[PHASE_NAME]` → TaskGroup.phase
- `[STORY_LABEL or N/A]` → TaskGroup.story_label or "N/A"
- `[TASK_IDS]` → Comma-separated TaskGroup.task_ids
- `[Task or Story]` → TaskGroup.issue_type
- Subject section → TaskGroup.subject
- Description section → TaskGroup.description (6 subsections)
- Test Plan section → TaskGroup.test_plan (4 subsections)
- Technical Details section → TaskGroup.technical_details (4 subsections)

**Task-to-Ticket Mapping Table**:

- For each TaskGroup, add row:
  - Task IDs: "T009-T020" (format as range or list)
  - Group ID: "G01"
  - Phase: "User Story 1"
  - Story: "US1" or "N/A"
  - Subject: First 50 chars of subject
  - JIRA Key: "Not Created" (will be updated if --create flag used)

**Statistics Section**:

**Tickets by Phase**:

- Count tickets per phase (setup, foundational, user_story with story labels, polish)
- Count tasks per phase
- Calculate avg tasks/ticket per phase

**Tickets by Issue Type**:

- Count "Task" issue types
- Count "Story" issue types
- Calculate percentages

**Grouping Effectiveness**:

- Report target group size: 5-8
- Report actual range: min-max
- Report median group size
- Report tasks successfully grouped vs isolated

**Write to File**:

- Write complete jira-tickets.md to `JIRA_TICKETS_FILE` path
- Ensure proper markdown formatting
- Preserve newlines and structure

**Progress Message**: "Written jira-tickets.md to [JIRA_TICKETS_FILE]"

---

## Step 7: Interactive Review (Optional, if --interactive flag)

**Purpose**: Allow user to review and modify tickets before JIRA creation

**Skip if**: `INTERACTIVE == false`

**For each ticket in ticket_groups[]**:

1. **Display Ticket Summary**:

   ```text
   Ticket G01: Implement task grouping algorithm for JIRA ticket generation
   Phase: User Story 1 (US1)
   Tasks: T009-T020 (12 tasks)
   Issue Type: Story

   Subject (80 chars): Implement task grouping algorithm for JIRA ticket generation
   Description: [Purpose]: This work implements the core grouping logic...
   [First 200 chars of description]
   ```

2. **Prompt User**:

   ```text
   Options:
   [A]ccept - Use ticket as-is, continue to next
   [E]dit - Modify ticket fields
   [S]plit - Split ticket into multiple tickets
   [M]erge - Merge with next ticket
   [K]ip - Exclude from JIRA creation
   [Q]uit - Exit interactive mode

   Choose option [A/E/S/M/K/Q]:
   ```

3. **Handle User Choice**:

   **Accept (A)**: Move to next ticket

   **Edit (E)**:
   - Prompt: "Which field to edit? [1] Subject, [2] Description, [3] Test Plan, [4] Technical Details, [C]ancel"
   - If field selected, prompt: "Enter new content (multiline, end with blank line):"
   - Update ticket field with new content
   - Re-display ticket summary
   - Return to options prompt

   **Split (S)**:
   - Display task list: "Tasks in this ticket: T009, T010, T011, T012, T013"
   - Prompt: "Split after task ID (e.g., T011 creates two tickets: T009-T011 and T012-T013):"
   - Parse split point
   - Create two new TaskGroup entities from original
   - Regenerate all 4 fields for both tickets using Step 5 logic
   - Update ticket_groups[] array
   - Update group_ids sequentially
   - Display both new tickets
   - Continue review with first new ticket

   **Merge (M)**:
   - Check if next ticket exists, error if last ticket
   - Display both tickets: "Merging G01 (12 tasks) with G02 (8 tasks) → G01 (20 tasks)"
   - Combine tasks from both tickets
   - Regenerate all 4 fields for merged ticket using Step 5 logic
   - Update ticket_groups[] array
   - Remove second ticket
   - Display merged ticket
   - Continue review with merged ticket

   **Skip (K)**:
   - Mark ticket as `excluded: true`
   - Ticket will not be created in JIRA (if --create)
   - Ticket still appears in jira-tickets.md with note: "**Status**: Skipped (excluded from JIRA creation)"
   - Move to next ticket

   **Quit (Q)**:
   - Exit interactive mode
   - Proceed to Step 8 with current ticket_groups[] state

4. **After Review Complete**:
   - Re-write jira-tickets.md with modified tickets (call Step 6 again)
   - Progress message: "Interactive review complete. [MODIFIED_COUNT] tickets modified."

---

## Step 8: Create Tickets in JIRA (Optional, if --create flag)

**Purpose**: Automatically create JIRA tickets using jira-db skill

**Skip if**: `CREATE_IN_JIRA == false` (default dry-run mode)

**Prerequisites Check**:

1. Verify jira-db skill available (attempt to call skill, handle failure gracefully)
2. Verify PROJECT_KEY provided (required, error if missing)
3. If EPIC_KEY provided, validate Epic exists (call jira-db to check)
4. If TEAM_NAME provided, resolve to team_id (call jira-db team resolution)

### Validate Epic (if EPIC_KEY provided)

**Call jira-db skill**:

```yaml
Skill: "jira-db"
Action: "validate_epic"
Parameters: {
  epic_key: EPIC_KEY,
  project_key: PROJECT_KEY
}
```

**Handle Response**:

- Success: Epic exists → Continue
- Error "Epic not found": ERROR with message "Epic [EPIC_KEY] not found in JIRA. Verify Epic key and try again."
- Error "Different project": WARN "Epic [EPIC_KEY] belongs to project [OTHER_PROJECT], not [PROJECT_KEY]. Cross-project linking may be intentional for portfolio management. Continue? (yes/no)"

### Resolve Team (if TEAM_NAME provided)

**Call jira-db skill**:

```yaml
Skill: "jira-db"
Action: "resolve_team"
Parameters: {
  team_name: TEAM_NAME
}
```

**Handle Response**:

- Success: team_id returned → Store in TEAM_ID variable
- Error "Team not found": ERROR with message "Team '[TEAM_NAME]' not found. Available teams: [LIST_FROM_CACHE]. Update team name and try again."

### For Each Ticket in ticket_groups[]

**Skip if**: ticket.excluded == true (marked as Skip in interactive mode)

**Convert Markdown to ADF** (Atlassian Document Format):

JIRA rich text fields require ADF format. Convert Description, Test Plan, Technical Details from markdown to ADF:

**Conversion Rules**:

- Headings: `## Text` → `{"type": "heading", "attrs": {"level": 2}, "content": [{"type": "text", "text": "Text"}]}`
- Bold: `**Text**` → `{"type": "text", "text": "Text", "marks": [{"type": "strong"}]}`
- Bullet list: `- Item` → `{"type": "bulletList", "content": [{"type": "listItem", "content": [{"type": "paragraph", "content": [{"type": "text", "text": "Item"}]}]}]}`
- Numbered list: `1. Item` → `{"type": "orderedList", "content": [...]}`
- Code block: ` ```code``` ` → `{"type": "codeBlock", "content": [{"type": "text", "text": "code"}]}`
- Inline code: `` `code` `` → `{"type": "text", "text": "code", "marks": [{"type": "code"}]}`
- Paragraph: Regular text → `{"type": "paragraph", "content": [{"type": "text", "text": "..."}]}`

**ADF Document Structure**:

```json
{
  "type": "doc",
  "version": 1,
  "content": [
    {"type": "heading", "attrs": {"level": 2}, "content": [...]},
    {"type": "paragraph", "content": [...]},
    {"type": "bulletList", "content": [...]}
  ]
}
```

**Convert Each Field**:

- description_adf = convert_markdown_to_adf(ticket.description)
- test_plan_adf = convert_markdown_to_adf(ticket.test_plan)
- technical_details_adf = convert_markdown_to_adf(ticket.technical_details)

**Create JIRA Issue via jira-db skill**:

**Call jira-db skill**:

```yaml
Skill: "jira-db"
Action: "create_issue"
Parameters: {
  project_key: PROJECT_KEY,
  summary: ticket.subject,
  description: description_adf,
  issuetype: ticket.issue_type,
  customfield_10332: test_plan_adf,
  customfield_10301: technical_details_adf,
  team_id: TEAM_ID (if TEAM_NAME provided),
  epic_link: EPIC_KEY (if EPIC_KEY provided)
}
```

**Handle Response**:

- Success: JIRA key returned (e.g., "PM-12345") → Store in ticket.jira_key
- Error "Custom field missing": ERROR with message "Custom field [FIELD_ID] not found in JIRA. Configure field mappings or contact JIRA admin."
- Error "Permission denied": ERROR with message "Insufficient permissions to create issues in project [PROJECT_KEY]. Verify jira-db credentials."
- Error "Project not found": ERROR with message "Project [PROJECT_KEY] not found in JIRA. Verify project key."

**Progress Message**:

```text
Created PM-12345 [Story]: Implement task grouping algorithm (Tasks T009-T020)
Created PM-12346 [Story]: Generate AI-friendly 4-field ticket structure (Tasks T021-T028)
...
```

**Update jira-tickets.md**:

- Re-read jira-tickets.md
- Update Task-to-Ticket Mapping table with JIRA keys
- Update each ticket header with: `**JIRA Key**: PM-12345`
- Re-write jira-tickets.md

**Error Recovery**:

- If ticket creation fails for one ticket, log error, continue with remaining tickets
- Report all failures at end: "3 tickets created successfully, 1 failed: [ERROR_DETAILS]"

---

## Step 9: Final Summary and Report

**Purpose**: Display completion summary and next steps

**Summary Output**:

```text
==========================================================
JIRA Tickets Generation Complete
==========================================================

Tasks Loaded: [TOTAL_TASKS] from [TASKS_FILE]
Tickets Generated: [TOTAL_TICKETS]

Grouping:
- Setup phase: [COUNT] tickets ([TASK_COUNT] tasks)
- Foundational phase: [COUNT] tickets ([TASK_COUNT] tasks)
- User Story 1 (P1): [COUNT] tickets ([TASK_COUNT] tasks)
- User Story 2 (P2): [COUNT] tickets ([TASK_COUNT] tasks)
- User Story 3 (P3): [COUNT] tickets ([TASK_COUNT] tasks)
- User Story 4 (P4): [COUNT] tickets ([TASK_COUNT] tasks)
- Polish phase: [COUNT] tickets ([TASK_COUNT] tasks)

Issue Types:
- Task: [COUNT] tickets ([PERCENT]%)
- Story: [COUNT] tickets ([PERCENT]%)

Output File: [JIRA_TICKETS_FILE]

[If CREATE_IN_JIRA:]
JIRA Integration:
- Tickets Created: [SUCCESS_COUNT]
- Tickets Failed: [FAILURE_COUNT]
- Project: [PROJECT_KEY]
- Team: [TEAM_NAME] (ID: [TEAM_ID])
- Epic: [EPIC_KEY] (if provided)

Created Tickets:
- PM-12345 [Story]: [Subject] (Tasks T009-T020)
- PM-12346 [Story]: [Subject] (Tasks T021-T028)
...

[If DRY_RUN:]
Dry-Run Mode: No tickets created in JIRA

==========================================================
Next Steps
==========================================================

1. Review generated tickets in: [JIRA_TICKETS_FILE]
2. Verify ticket content, grouping, and structure
3. To create tickets in JIRA:
   /speckit.jira-tasks --create --project [PROJECT] --team "[TEAM]"
4. For interactive review before creation:
   /speckit.jira-tasks --interactive --create --project [PROJECT]
5. To associate with an Epic:
   /speckit.jira-tasks --create --project [PROJECT] --epic [EPIC-KEY]

==========================================================
```

**Completion**: Command execution complete

---

## Error Handling

**Missing tasks.md**:

- ERROR: "tasks.md not found at [TASKS_FILE]. Run `/speckit.tasks` first to generate task breakdown."
- Exit with error code

**Empty tasks.md**:

- ERROR: "tasks.md is empty. Run `/speckit.tasks` to generate tasks."
- Exit with error code

**Invalid tasks.md format**:

- WARN: "Task T### has invalid format. Expected: `- [ ] T### [P?] [Story?] Description with file path`"
- Continue processing other tasks

**Tasks without file paths**:

- WARN: "Task T### missing file path. Technical Details may be incomplete."
- Continue processing

**Large task count (200+)**:

- WARN: "Feature has 200+ tasks. Consider splitting into multiple features for better manageability."
- Continue processing

**jira-db skill unavailable** (when --create used):

- ERROR: "jira-db skill not found or not configured. Install jira-db skill and configure JIRA credentials. See: [SETUP_LINK]"
- Exit with error code

**Team name not found**:

- ERROR: "Team '[TEAM_NAME]' not found. Available teams: [LIST]. Use correct team name or omit --team flag."
- Exit with error code

**Invalid Epic key**:

- ERROR: "Epic [EPIC_KEY] not found in JIRA. Verify Epic key or omit --epic flag."
- Exit with error code

**Epic from different project**:

- WARN: "Epic [EPIC_KEY] belongs to project [OTHER_PROJECT], not [PROJECT_KEY]. Cross-project linkage may be intentional. Continue? (yes/no)"
- If no: Exit
- If yes: Continue with cross-project Epic link

**Custom field IDs missing**:

- ERROR: "Custom field [FIELD_ID] not found in JIRA instance. Configure field mappings or contact JIRA administrator. Expected fields: Test Plan (customfield_10332), Technical Details (customfield_10301)"
- Exit with error code

**Conflicting story labels**:

- WARN: "Task T### has multiple story labels or belongs to multiple phases. Using first occurrence: [LABEL]"
- Continue processing

**Mixed phase grouping** (should not occur after Pass 1):

- WARN: "Ticket G## contains tasks from multiple phases: [PHASES]. Using majority phase: [PHASE] for issue type determination."
- Continue processing

---

## Usage Examples

### Example 1: Dry-Run (Default)

```bash
/speckit.jira-tasks
```

**Result**: Generates jira-tickets.md, no JIRA creation

### Example 2: Create in JIRA

```bash
/speckit.jira-tasks --create --project PM --team "Platform Team"
```

**Result**: Generates jira-tickets.md, creates tickets in JIRA project PM, assigns to Platform Team

### Example 3: Create with Epic

```bash
/speckit.jira-tasks --create --project PM --team "Backend Team" --epic PM-1000
```

**Result**: Creates tickets with Epic PM-1000 as parent

### Example 4: Interactive Review

```bash
/speckit.jira-tasks --interactive
```

**Result**: Prompts for review/edit of each ticket, saves to jira-tickets.md

### Example 5: Interactive + JIRA Creation

```bash
/speckit.jira-tasks --interactive --create --project PM --team "Platform Team"
```

**Result**: Review tickets, edit as needed, then create in JIRA

### Example 6: Preview with Dry-Run

```bash
/speckit.jira-tasks --dry-run
```

**Result**: Explicit dry-run (same as default), preview tickets without creation

### Example 7: Full Workflow

```bash
# 1. Generate and review locally
/speckit.jira-tasks

# 2. Review generated jira-tickets.md
# Make manual edits if needed

# 3. Create in JIRA when ready
/speckit.jira-tasks --create --project PM --team "Platform Team" --epic PM-1000
```

---

**Command Version**: 1.0.0
**Created**: 2025-11-14
**Last Updated**: 2025-11-14
