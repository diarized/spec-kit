# Data Model: JIRA Tasks Integration

**Feature**: [spec.md](spec.md) | **Plan**: [plan.md](plan.md)
**Date**: 2025-11-12

## Overview

This document defines the data structures used by the `/speckit.jira-tasks` command for parsing, transforming, and generating JIRA tickets from tasks.md.

---

## Entity: Task

**Description**: Individual work item parsed from tasks.md checklist format.

**Attributes**:

| Attribute | Type | Required | Description | Example |
|-----------|------|----------|-------------|---------|
| `id` | string | Yes | Sequential task identifier | "T001", "T042" |
| `checkbox` | boolean | No | Completion status (always unchecked initially) | false |
| `parallel` | boolean | No | Whether task can be parallelized (has [P] marker) | true |
| `story_label` | string | No | User story association marker | "US1", "US2", null |
| `description` | string | Yes | Task description text | "Create User model in src/models/user.py" |
| `file_path` | string | No | Extracted file path from description | "src/models/user.py" |
| `phase` | enum | Yes | Inferred phase from context | "setup", "foundational", "user_story", "polish" |
| `line_number` | integer | Yes | Line number in tasks.md for traceability | 42 |

**Validation Rules**:
- Task ID must match pattern: `T\d{3}` (e.g., T001, T099)
- If story_label exists, must match pattern: `US\d+` (e.g., US1, US10)
- Description must not be empty
- Phase must be one of: "setup", "foundational", "user_story", "polish"

**Source Format** (tasks.md):
```markdown
- [ ] T012 [P] [US1] Create User model in src/models/user.py
```

**Parsed Structure** (internal representation):
```json
{
  "id": "T012",
  "checkbox": false,
  "parallel": true,
  "story_label": "US1",
  "description": "Create User model in src/models/user.py",
  "file_path": "src/models/user.py",
  "phase": "user_story",
  "line_number": 42
}
```

**Lifecycle**:
1. Parse from tasks.md markdown checklist
2. Group into TaskGroups based on story label and phase
3. Referenced in JiraTicket description by task ID
4. Mapped to JIRA ticket key in task-to-ticket mapping

---

## Entity: TaskGroup

**Description**: Collection of related tasks that form a JIRA-sized work unit (approximately 1-3 days of work).

**Attributes**:

| Attribute | Type | Required | Description | Example |
|-----------|------|----------|-------------|---------|
| `group_id` | string | Yes | Sequential group identifier | "G001", "G012" |
| `tasks` | Task[] | Yes | Ordered list of tasks in this group | [Task, Task, Task] |
| `phase` | enum | Yes | Phase this group belongs to | "user_story" |
| `story_label` | string | No | User story if applicable | "US1" |
| `estimated_days` | float | No | Estimated work duration | 2.5 |
| `primary_files` | string[] | No | Most frequently referenced files | ["src/auth/service.py"] |

**Validation Rules**:
- Must contain 1-12 tasks (target: 5-8 tasks)
- All tasks in group must have same phase
- All tasks in group should have same story_label (if applicable)
- Task IDs must be sequential or close (prefer T001-T008 vs T001, T003, T050, T087)

**Grouping Logic**:
```text
1. Separate by phase (setup, foundational, user_story, polish)
2. Within user_story phase: Group by story_label
3. Within each story: Cluster by file_path similarity
4. Enforce size constraints: 5-8 tasks per group (1-3 days estimate)
5. Preserve task order: Earlier task IDs stay together
```

**Example**:
```json
{
  "group_id": "G003",
  "tasks": [
    {"id": "T012", "description": "Create User model...", "story_label": "US1"},
    {"id": "T013", "description": "Add User validation...", "story_label": "US1"},
    {"id": "T014", "description": "Create UserService...", "story_label": "US1"}
  ],
  "phase": "user_story",
  "story_label": "US1",
  "estimated_days": 2.0,
  "primary_files": ["src/models/user.py", "src/services/user_service.py"]
}
```

**Lifecycle**:
1. Created during grouping phase (Pass 1-4 of algorithm)
2. Transformed into JiraTicket with 4-field structure
3. Discarded after ticket generation (not persisted)

---

## Entity: JiraTicket

**Description**: Work item with 4-field AI-friendly structure ready for creation in JIRA.

**Attributes**:

| Attribute | Type | Required | Description | Example |
|-----------|------|----------|-------------|---------|
| `ticket_id` | string | Yes | Sequential ticket identifier (pre-creation) | "TICKET-001" |
| `subject` | string | Yes | Single clear sentence (50-100 chars) | "Implement user authentication system" |
| `description` | text | Yes | AI-friendly description with explicit sections | See template below |
| `test_plan` | text | Yes | Verification approach with explicit structure | See template below |
| `technical_details` | text | Yes | Implementation guide with files and sequence | See template below |
| `issue_type` | enum | Yes | Inferred JIRA issue type | "Task", "Story" |
| `phase` | enum | Yes | Source phase | "setup", "foundational", "user_story", "polish" |
| `story_label` | string | No | User story association | "US1" |
| `task_ids` | string[] | Yes | Original task IDs covered | ["T012", "T013", "T014"] |
| `jira_key` | string | No | JIRA ticket key after creation | "PM-12345" |
| `epic_key` | string | No | Epic parent if --epic flag used | "PM-1000" |

**Validation Rules**:
- Subject must be single sentence (no period at end)
- Description must include all required sections: Purpose, Tasks Covered, Affected Systems
- Test Plan must describe verification approach
- Technical Details must list specific files and functions
- Issue type must be "Task" (setup/foundational/polish) or "Story" (user_story)
- Task IDs must reference valid tasks from source TaskGroup

**Field Templates**:

**Subject Template**:
```text
[Verb] [object] [purpose/context]
Example: "Implement user authentication with JWT tokens"
```

**Description Template**:
```markdown
**Purpose**: [Why this work is needed - 1-2 sentences]

**Tasks Covered**: T012, T013, T014, T018

**Affected Systems**:
- Authentication module
- User management
- Session handling

**Key Workflows**:
- User registration flow
- Login/logout flow
- Token refresh flow

**Stakeholders**:
- End users (login experience)
- Security team (authentication standards)

**Edge Cases**:
- Expired tokens
- Concurrent login attempts
- Password reset during active session
```

**Test Plan Template**:
```markdown
**Verification Approach**: Manual testing + automated unit tests

**Component Tests**:
- **Auth Service**: Verify token generation and validation
- **User Model**: Verify password hashing and validation
- **Middleware**: Verify request authentication

**Integration Test**: End-to-end user registration → login → authenticated request flow

**Edge Case Validation**:
- Test token expiration handling
- Test concurrent session management
```

**Technical Details Template**:
```markdown
**Files to Modify**:
- `src/auth/service.py`: Add AuthService class with token methods
- `src/models/user.py`: Add password hashing and validation
- `src/middleware/auth.py`: Add authentication middleware

**Functions to Add/Modify**:
- `generate_token(user_id: str) -> str`: Generate JWT token
- `validate_token(token: str) -> User`: Validate and decode token
- `hash_password(password: str) -> str`: Hash password with bcrypt

**Implementation Sequence**:
1. Create User model with password hashing
2. Implement AuthService with token generation
3. Add authentication middleware
4. Wire up to API endpoints
5. Add error handling and logging

**Dependencies**:
- Internal: User model must exist (T011)
- External: PyJWT library, bcrypt library
```

**JIRA Field Mapping**:

| JiraTicket Field | JIRA Field | Field Type | Format |
|------------------|------------|------------|--------|
| `subject` | `summary` | Plain text | String |
| `description` | `description` | Rich text | ADF (Atlassian Document Format) |
| `test_plan` | `customfield_10332` | Rich text | ADF |
| `technical_details` | `customfield_10301` | Rich text | ADF |
| `issue_type` | `issuetype.name` | Enum | "Task" or "Story" |
| `epic_key` | Epic Link field | String | "PM-1000" (if --epic flag used) |

**Lifecycle**:
1. Created from TaskGroup during ticket generation
2. Written to jira-tickets.md for review
3. Converted to JIRA API format (ADF) when --create flag used
4. Created in JIRA via jira-db skill
5. JIRA key stored in jira_key attribute
6. Referenced in task-to-ticket mapping output

---

## Entity: FieldMapping

**Description**: Configuration that maps 4-field structure to JIRA issue fields (including custom field IDs).

**Attributes**:

| Attribute | Type | Required | Description | Example |
|-----------|------|----------|-------------|---------|
| `subject_field` | string | Yes | JIRA field for Subject | "summary" |
| `description_field` | string | Yes | JIRA field for Description | "description" |
| `test_plan_field` | string | Yes | JIRA custom field for Test Plan | "customfield_10332" |
| `technical_details_field` | string | Yes | JIRA custom field for Technical Details | "customfield_10301" |
| `issue_type_field` | string | Yes | JIRA field for issue type | "issuetype.name" |
| `epic_link_field` | string | No | JIRA field for Epic Link | "parent" or custom field |

**Usage**:
- Default mapping (hardcoded based on spec assumptions)
- Future enhancement: Configuration file for custom JIRA instances
- Used during JIRA ticket creation to populate correct fields

**Example**:
```json
{
  "subject_field": "summary",
  "description_field": "description",
  "test_plan_field": "customfield_10332",
  "technical_details_field": "customfield_10301",
  "issue_type_field": "issuetype.name",
  "epic_link_field": "parent"
}
```

---

## Entity: TaskToTicketMapping

**Description**: Traceability record linking original task IDs to created JIRA ticket keys.

**Attributes**:

| Attribute | Type | Required | Description | Example |
|-----------|------|----------|-------------|---------|
| `task_id_range` | string | Yes | Range of task IDs (human-readable) | "T012-T018" |
| `task_ids` | string[] | Yes | Explicit list of task IDs | ["T012", "T013", ...] |
| `jira_key` | string | No | JIRA ticket key (null if not created) | "PM-12345" |
| `issue_type` | enum | Yes | JIRA issue type | "Task", "Story" |
| `subject` | string | Yes | Ticket subject | "Implement authentication" |

**Output Format** (jira-tickets.md mapping table):
```markdown
| Task IDs | JIRA Key | Issue Type | Subject |
|----------|----------|------------|---------|
| T001-T008 | PM-12345 | Story | Implement user authentication |
| T009-T015 | PM-12346 | Task | Setup project infrastructure |
```

**Output Format** (command progress):
```text
Created ticket PM-12345 [Story]: Implement user authentication (Tasks T001-T008)
Created ticket PM-12346 [Task]: Setup project infrastructure (Tasks T009-T015)
```

**Lifecycle**:
1. Created during ticket generation (with null jira_key)
2. Updated with jira_key after JIRA creation
3. Written to jira-tickets.md mapping table
4. Displayed in command progress output

---

## Entity: Epic (External)

**Description**: JIRA issue type that serves as parent container for Stories and Tasks (defined in JIRA, not created by this feature).

**Attributes** (read-only, from JIRA):

| Attribute | Type | Description | Example |
|-----------|------|-------------|---------|
| `key` | string | JIRA Epic key | "PM-1000" |
| `summary` | string | Epic title | "Q1 Authentication Initiative" |
| `project_key` | string | JIRA project | "PM" |

**Usage**:
- When `--epic PM-1000` flag provided, validate Epic exists in JIRA
- Set Epic Link field on all created tickets
- Epic must exist before command execution (not created by command)

**Validation**:
- Epic key format: `[A-Z]+-\d+`
- Epic must exist in JIRA (checked via jira-db skill)
- Epic should belong to same project as `--project` flag (warning if cross-project)

---

## Relationships

```text
Task (1) ──< belongs to >── (1) TaskGroup
TaskGroup (1) ──< generates >── (1) JiraTicket
JiraTicket (1) ──< maps to >── (1) TaskToTicketMapping
JiraTicket (N) ──< child of >── (1) Epic [optional]

Workflow:
tasks.md → Parse → [Task] → Group → [TaskGroup] → Generate → [JiraTicket] → Create → JIRA
                                                                     ↓
                                                            [TaskToTicketMapping]
```

---

## State Transitions

### Task States
```text
[Unparsed in tasks.md] → [Parsed] → [Grouped] → [Referenced in Ticket]
```

### TaskGroup States
```text
[Created] → [Validated] → [Ticket Generated] → [Discarded]
```

### JiraTicket States
```text
[Created] → [Written to jira-tickets.md] → [ADF Converted] → [Created in JIRA] → [Mapped]
                                                                      ↓
                                                              jira_key populated
```

---

## Notes

- **No database**: All entities are in-memory during command execution
- **No persistence**: Only tasks.md (input) and jira-tickets.md (output) are files
- **Read-only tasks.md**: Command never modifies tasks.md
- **Idempotency**: Running command multiple times regenerates jira-tickets.md but doesn't duplicate JIRA tickets (no automatic creation without --create flag)
