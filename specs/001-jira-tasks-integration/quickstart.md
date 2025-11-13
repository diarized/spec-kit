# Quickstart: JIRA Tasks Integration

**Feature**: [spec.md](spec.md) | **Plan**: [plan.md](plan.md) | **Data Model**: [data-model.md](data-model.md)
**Date**: 2025-11-12

## Overview

This quickstart guide demonstrates how to use the `/speckit.jira-tasks` command to convert AI-friendly micro-tasks into JIRA-sized work tickets.

---

## Prerequisites

Before using this command, ensure:

1. **tasks.md exists**: Run `/speckit.tasks` to generate tasks.md for your feature
2. **Working directory**: Execute from project root (where .claude/ directory exists)
3. **jira-db skill** (optional): Required only if you want automatic JIRA ticket creation (`--create` flag)

---

## Scenario 1: Generate JIRA Tickets (No JIRA Creation)

**Goal**: Convert tasks.md into jira-tickets.md for review without creating anything in JIRA.

**Use Case**: You want to review the ticket grouping and structure before committing to JIRA.

### Steps

```bash
# From project root
cd /path/to/your/project

# Run the command (no flags needed)
/speckit.jira-tasks
```

### Expected Output

```text
✓ Loading tasks.md... (87 tasks loaded from specs/001-feature/tasks.md)
✓ Grouping tasks by story and phase... (12 groups formed)
  - Setup phase: 1 group (6 tasks)
  - Foundational phase: 1 group (8 tasks)
  - User Story 1 (US1): 3 groups (24 tasks)
  - User Story 2 (US2): 4 groups (31 tasks)
  - User Story 3 (US3): 2 groups (14 tasks)
  - Polish phase: 1 group (4 tasks)
✓ Generating ticket content... (12 tickets created)
✓ Writing jira-tickets.md... (/path/to/specs/001-feature/jira-tickets.md)

Summary:
  Total tasks: 87
  Total tickets: 12
  Average tasks per ticket: 7.3
  Output: /path/to/specs/001-feature/jira-tickets.md

Next steps:
  - Review jira-tickets.md for ticket structure and grouping
  - Run with --dry-run to preview JIRA creation
  - Run with --create --project PM --team "Team Name" to create tickets in JIRA
```

### Verify

```bash
# Open the generated file
cat specs/001-feature/jira-tickets.md

# Or open in editor
code specs/001-feature/jira-tickets.md
```

### What You Should See

A markdown file with:
- Feature header with metadata
- 12 tickets, each with 4 fields (Subject, Description, Test Plan, Technical Details)
- Task-to-ticket mapping table
- Statistics summary

---

## Scenario 2: Preview JIRA Creation (Dry-Run)

**Goal**: See what tickets would be created in JIRA without actually creating them.

**Use Case**: You want to verify ticket issue types, Epic associations, and team assignments before executing.

### Steps

```bash
# From project root
/speckit.jira-tasks --dry-run --project PM --team "Engineering Team" --epic PM-1000
```

### Expected Output

```text
✓ Loading tasks.md... (87 tasks loaded)
✓ Grouping tasks by story and phase... (12 groups formed)
✓ Generating ticket content... (12 tickets created)
✓ Validating Epic PM-1000... (Epic found: "Q1 Authentication Initiative")

DRY-RUN MODE: The following tickets would be created:

Ticket 1 [Task]: Setup project infrastructure
  Tasks: T001-T006 (6 tasks)
  Files: pyproject.toml, src/__init__.py, tests/conftest.py
  Epic: None (Setup phase tickets not associated with Epic)
  Team: Engineering Team
  Project: PM

Ticket 2 [Task]: Create foundational authentication components
  Tasks: T007-T014 (8 tasks)
  Files: src/auth/base.py, src/auth/middleware.py
  Epic: PM-1000
  Team: Engineering Team
  Project: PM

Ticket 3 [Story]: Implement user registration flow
  Tasks: T015-T022 (8 tasks)
  Story: US1
  Files: src/api/auth.py, src/services/user.py
  Epic: PM-1000
  Team: Engineering Team
  Project: PM

... [remaining tickets]

Summary:
  Total tickets to create: 12
  Issue types: 3 Task, 9 Story
  Epic association: 11 tickets linked to PM-1000, 1 ticket without Epic
  Team assignment: Engineering Team (all tickets)
  Project: PM

Output written to: /path/to/specs/001-feature/jira-tickets.md

To create these tickets in JIRA, run without --dry-run:
  /speckit.jira-tasks --create --project PM --team "Engineering Team" --epic PM-1000
```

### Verify

Check that:
- Issue types are correct (Setup/Foundational/Polish → Task, User Stories → Story)
- Epic association is as expected (Setup tickets have no Epic, others linked to PM-1000)
- Team assignment is correct
- File paths look reasonable

---

## Scenario 3: Create JIRA Tickets Automatically

**Goal**: Actually create tickets in JIRA with proper team assignment and Epic association.

**Use Case**: You've reviewed jira-tickets.md and dry-run output, ready to populate JIRA.

### Prerequisites

- jira-db skill configured with valid credentials
- Valid JIRA project key (e.g., "PM")
- Valid team name that jira-db can resolve
- Valid Epic key (optional, e.g., "PM-1000")

### Steps

```bash
# From project root
/speckit.jira-tasks --create --project PM --team "Engineering Team" --epic PM-1000
```

### Expected Output

```text
✓ Loading tasks.md... (87 tasks loaded)
✓ Grouping tasks by story and phase... (12 groups formed)
✓ Generating ticket content... (12 tickets created)
✓ Validating Epic PM-1000... (Epic found: "Q1 Authentication Initiative")
✓ Resolving team "Engineering Team"... (Team ID: 12345)
✓ Creating tickets in JIRA...
  ✓ Created PM-12345 [Task]: Setup project infrastructure (Tasks T001-T006)
  ✓ Created PM-12346 [Task]: Create foundational authentication components (Tasks T007-T014)
  ✓ Created PM-12347 [Story]: Implement user registration flow (Tasks T015-T022)
  ✓ Created PM-12348 [Story]: Implement login/logout flow (Tasks T023-T031)
  ✓ Created PM-12349 [Story]: Add password reset functionality (Tasks T032-T038)
  ✓ Created PM-12350 [Story]: Implement user profile management (Tasks T039-T050)
  ✓ Created PM-12351 [Story]: Add OAuth2 integration (Tasks T051-T062)
  ✓ Created PM-12352 [Story]: Implement session management (Tasks T063-T069)
  ✓ Created PM-12353 [Story]: Add multi-factor authentication (Tasks T070-T077)
  ✓ Created PM-12354 [Story]: Create admin user management (Tasks T078-T082)
  ✓ Created PM-12355 [Task]: Polish authentication UI and error handling (Tasks T083-T087)

✓ Writing jira-tickets.md with ticket keys... (/path/to/specs/001-feature/jira-tickets.md)

Summary:
  Total tasks: 87
  Total tickets created: 12
  Issue types: 3 Task, 9 Story
  Epic association: 11 tickets linked to PM-1000
  Team assignment: Engineering Team (all tickets)
  Project: PM

Task-to-Ticket Mapping:
  T001-T006 → PM-12345 [Task]
  T007-T014 → PM-12346 [Task]
  T015-T022 → PM-12347 [Story]
  T023-T031 → PM-12348 [Story]
  T032-T038 → PM-12349 [Story]
  T039-T050 → PM-12350 [Story]
  T051-T062 → PM-12351 [Story]
  T063-T069 → PM-12352 [Story]
  T070-T077 → PM-12353 [Story]
  T078-T082 → PM-12354 [Story]
  T083-T087 → PM-12355 [Task]

View tickets in JIRA: https://yourcompany.atlassian.net/browse/PM-1000
```

### Verify

1. Open JIRA and navigate to your Epic (PM-1000)
2. Verify all child tickets appear under the Epic
3. Check that issue types are correct (Task vs Story)
4. Verify team assignment
5. Open a few tickets and verify all 4 fields are populated (Subject, Description, Test Plan, Technical Details)

---

## Scenario 4: Create Tickets Without Epic Association

**Goal**: Create tickets that are not linked to any Epic (useful for features not part of Epic planning).

**Use Case**: Your team doesn't use Epics, or you want to manually assign Epics later in JIRA.

### Steps

```bash
# From project root
/speckit.jira-tasks --create --project PM --team "Engineering Team"
# Note: No --epic flag
```

### Expected Output

```text
... [same as Scenario 3, but Epic-related lines omitted]

✓ Creating tickets in JIRA...
  ✓ Created PM-12345 [Task]: Setup project infrastructure (Tasks T001-T006)
  ... [no Epic association mentioned]

Summary:
  Total tickets created: 12
  Epic association: None (use --epic flag to associate with Epic)
  ...
```

### Verify

- Tickets appear in JIRA project PM
- Tickets have no Epic parent link
- You can manually assign Epic in JIRA UI if needed later

---

## Scenario 5: Handle jira-db Skill Not Configured

**Goal**: Generate jira-tickets.md even when jira-db skill is unavailable.

**Use Case**: Working offline or in environment without JIRA access.

### Steps

```bash
# From project root
/speckit.jira-tasks --create --project PM --team "Engineering Team"
```

### Expected Output (Error)

```text
✓ Loading tasks.md... (87 tasks loaded)
✓ Grouping tasks by story and phase... (12 groups formed)
✓ Generating ticket content... (12 tickets created)
✓ Writing jira-tickets.md... (/path/to/specs/001-feature/jira-tickets.md)

✗ Failed to create tickets in JIRA

Error: jira-db skill not configured or unavailable

Resolution:
  1. Configure jira-db skill with valid JIRA credentials
  2. Verify skill is accessible via Claude Code skill system
  3. Or omit --create flag to generate jira-tickets.md only

Output file jira-tickets.md still created for manual ticket creation.
```

### Fallback Workflow

1. Review jira-tickets.md manually
2. Copy ticket content from jira-tickets.md into JIRA manually
3. Or configure jira-db skill and retry with --create flag

---

## Scenario 6: Handle Invalid Epic Key

**Goal**: Fail fast when Epic key doesn't exist in JIRA.

**Use Case**: Prevent creating tickets with invalid Epic associations.

### Steps

```bash
# From project root
/speckit.jira-tasks --create --project PM --team "Engineering Team" --epic PM-9999
```

### Expected Output (Error)

```text
✓ Loading tasks.md... (87 tasks loaded)
✓ Grouping tasks by story and phase... (12 groups formed)
✓ Generating ticket content... (12 tickets created)
✗ Validating Epic PM-9999... (Epic not found)

Error: Epic PM-9999 not found in JIRA

Resolution:
  1. Verify Epic key is correct (check JIRA)
  2. Ensure Epic exists in project PM
  3. Or omit --epic flag to create tickets without Epic association

No tickets created in JIRA.
```

### Recovery

```bash
# Fix Epic key and retry
/speckit.jira-tasks --create --project PM --team "Engineering Team" --epic PM-1000

# Or proceed without Epic
/speckit.jira-tasks --create --project PM --team "Engineering Team"
```

---

## Scenario 7: Handle Team Name Not Found

**Goal**: Provide helpful error when team name doesn't resolve.

**Use Case**: Typo in team name or team doesn't exist in jira-db cache.

### Steps

```bash
# From project root
/speckit.jira-tasks --create --project PM --team "NonexistentTeam"
```

### Expected Output (Error)

```text
✓ Loading tasks.md... (87 tasks loaded)
✓ Grouping tasks by story and phase... (12 groups formed)
✓ Generating ticket content... (12 tickets created)
✗ Resolving team "NonexistentTeam"... (Team not found)

Error: Team "NonexistentTeam" not found in jira-db cache

Available teams:
  - Engineering Team
  - Product Team
  - Design Team
  - QA Team

Resolution:
  1. Use one of the available team names (exact match required)
  2. Or contact admin to add team to jira-db cache

No tickets created in JIRA.
```

### Recovery

```bash
# Use valid team name
/speckit.jira-tasks --create --project PM --team "Engineering Team"
```

---

## Common Patterns

### Pattern 1: Review Before Creating

```bash
# Step 1: Generate tickets for review
/speckit.jira-tasks

# Step 2: Review output
cat specs/001-feature/jira-tickets.md

# Step 3: Dry-run to preview JIRA creation
/speckit.jira-tasks --dry-run --project PM --team "Team Name" --epic PM-1000

# Step 4: Actually create in JIRA
/speckit.jira-tasks --create --project PM --team "Team Name" --epic PM-1000
```

### Pattern 2: Iterative Refinement

If you need to adjust task grouping:

1. Run `/speckit.jira-tasks` to see initial grouping
2. Review jira-tickets.md and identify issues
3. Adjust tasks.md (add/remove story labels, change task order)
4. Re-run `/speckit.jira-tasks` to regenerate
5. Repeat until satisfied
6. Finally run with `--create` flag

### Pattern 3: Working Without JIRA Integration

If you don't have jira-db skill configured:

```bash
# Generate ticket content
/speckit.jira-tasks

# Manually create tickets in JIRA using jira-tickets.md as template
# (Copy Subject, Description, Test Plan, Technical Details for each ticket)
```

---

## Command Reference

### Flags

| Flag | Required | Default | Description |
|------|----------|---------|-------------|
| (none) | No | N/A | Generate jira-tickets.md only, no JIRA creation |
| `--dry-run` | No | false | Preview what would be created without JIRA side effects |
| `--create` | No | false | Actually create tickets in JIRA via jira-db skill |
| `--project KEY` | Yes if --create | N/A | JIRA project key (e.g., "PM") |
| `--team "Name"` | Yes if --create | N/A | Team name for ticket assignment (quotes if spaces) |
| `--epic KEY` | No | N/A | Epic key for parent link (e.g., "PM-1000") |

### Examples

```bash
# Basic: Generate tickets only
/speckit.jira-tasks

# Dry-run with Epic
/speckit.jira-tasks --dry-run --project PM --team "Engineering" --epic PM-1000

# Create without Epic
/speckit.jira-tasks --create --project PM --team "Engineering"

# Create with Epic
/speckit.jira-tasks --create --project PM --team "Engineering Team" --epic PM-1000
```

---

## Troubleshooting

### Issue: "tasks.md not found"

**Cause**: Command run before `/speckit.tasks`

**Solution**: Run `/speckit.tasks` first to generate tasks.md

### Issue: "jira-db skill not available"

**Cause**: Skill not configured or Claude Code can't access it

**Solution**:
1. Configure jira-db skill with JIRA credentials
2. Verify skill is registered in Claude Code
3. Or use manual workflow (omit --create flag)

### Issue: "Epic PM-1000 belongs to different project"

**Cause**: Epic in different project than --project flag

**Solution**:
- This is a warning, not an error
- Verify cross-project linkage is intentional (may be valid for portfolio management)
- Or use Epic from same project

### Issue: Tickets have wrong issue types

**Cause**: Tasks not properly labeled with story markers or phases

**Solution**:
1. Review tasks.md structure
2. Ensure User Story tasks have [US1], [US2] labels
3. Ensure Setup/Foundational/Polish tasks are in correct phases
4. Re-run `/speckit.tasks` if needed

### Issue: Too many/few tasks per ticket

**Cause**: Grouping algorithm not optimal for your feature structure

**Solution**:
- Currently: No tuning options (fixed 5-8 task target)
- Future enhancement: Allow configuration of group size targets
- Workaround: Manually adjust story labels in tasks.md to influence grouping

---

## Next Steps

After using this command:

1. **Review jira-tickets.md**: Ensure grouping and content are appropriate
2. **Verify JIRA tickets**: Check that tickets appear correctly in JIRA (if --create used)
3. **Proceed with implementation**: Use `/speckit.implement` to execute tasks.md
4. **Track progress**: Update JIRA tickets as work progresses
5. **Maintain traceability**: Use task IDs from tickets to link JIRA with tasks.md

---

## Related Documentation

- [spec.md](spec.md) - Feature specification
- [plan.md](plan.md) - Technical implementation plan
- [data-model.md](data-model.md) - Data structures and entities
- [tasks.md](tasks.md) - Detailed task breakdown (generate with `/speckit.tasks`)
