# Implementation Plan: JIRA Tasks Integration

**Branch**: `001-jira-tasks-integration` | **Date**: 2025-11-12 | **Updated**: 2025-11-14 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/001-jira-tasks-integration/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

This feature adds a new `/speckit.jira-tasks` slash command that bridges AI-friendly micro-tasks from tasks.md with human project management in JIRA. The command groups granular tasks (T001, T002, etc.) into JIRA-sized work units (1-3 days of work), generates tickets with a 4-field AI-friendly structure (Subject, Description, Test Plan, Technical Details), and optionally auto-creates tickets in JIRA using the jira-db skill with team assignment and Epic association.

**Distribution**: Command distributed via GitHub release ZIP (integrated into spec-kit's standard release process), installed automatically during `specify init` into `.claude/commands/` directory alongside existing spec-kit commands.

**Primary Value**: Eliminates manual work of translating detailed AI task breakdowns into project management tickets while preserving traceability between systems.

## Technical Context

**Language/Version**: Markdown (slash command) + Python 3.11+ (jira-db skill integration)
**Primary Dependencies**:
- Claude Code slash command system (.claude/commands/)
- jira-db skill (MCP-based JIRA integration)
- Existing Spec Kit infrastructure (.specify/templates/, .specify/scripts/)

**Storage**: File-based (tasks.md input, jira-tickets.md output)
**Testing**: Manual verification against acceptance scenarios from spec.md
**Target Platform**: Claude Code CLI environment (macOS/Linux/Windows)
**Project Type**: Single project (CLI tooling extension)
**Performance Goals**: <30 seconds for typical feature (50-100 tasks), <200ms for file I/O operations
**Constraints**:
- Must work within Claude Code slash command execution model
- Limited to synchronous execution (no background jobs)
- Must handle jira-db skill unavailability gracefully
- File paths must be absolute for cross-platform compatibility

**Scale/Scope**:
- Typical input: 50-100 tasks from tasks.md
- Expected output: 8-15 JIRA tickets in jira-tickets.md
- Batch creation: up to 20 tickets per command invocation

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### I. Specification-First Development ✅

**Status**: PASS
- Feature spec completed and clarified before planning
- All user stories prioritized (P1-P4) with independent test criteria
- Requirements use MUST/SHOULD appropriately
- No [NEEDS CLARIFICATION] markers remain in spec

### II. Structured Workflow Discipline ✅

**Status**: PASS
- Constitution established (v1.0.0)
- Specification underwent structured clarification (3 questions resolved)
- Technical planning initiated after spec validation
- Tasks will be organized by user story in Phase 2

### III. Independent User Story Delivery ✅

**Status**: PASS
- P1: Generate JIRA-sized work units (independently testable via file output)
- P2: Create AI-friendly 4-field structure (independently testable via content validation)
- P3: Auto-create via jira-db skill (independently testable with JIRA credentials)
- P4: Interactive review (independently testable in isolation)
- Each story delivers standalone value without inter-story dependencies

### IV. Explicit Over Implicit ✅

**Status**: PASS
- Technical context specifies: Markdown/Python, slash command system, file-based storage
- Performance goals explicit: <30s execution, <200ms I/O
- Constraints documented: synchronous execution, graceful skill unavailability handling
- File structure will be concrete (no placeholders) in Project Structure section

### V. Test-Driven When Requested ⚠️

**Status**: TESTS NOT REQUESTED
- Specification does not request formal automated tests
- Acceptance criteria provide manual verification approach
- User Story acceptance scenarios serve as test specifications
- **Decision**: Manual testing via acceptance scenarios is sufficient for CLI tooling extension

### VI. Technology Independence ✅

**Status**: PASS
- Specification remained technology-agnostic
- Technical decisions made in planning phase (this document)
- Technology choices justified: Markdown for slash commands (Claude Code requirement), jira-db skill for JIRA integration (specification requirement)

### Simplicity Bias ✅

**Status**: PASS - No violations to justify
- Single project structure (no multi-project split needed)
- Reuses existing Spec Kit infrastructure (.specify/templates/, scripts)
- No new external dependencies introduced (jira-db skill already available)
- File-based I/O (simplest approach for command-line tool)
- No architectural patterns beyond functional decomposition

### Observability & Debuggability ✅

**Status**: PASS
- Success Criterion SC-009 requires: "clear progress output showing: tasks loaded, groups formed, tickets generated, JIRA creation status"
- Dry-run mode (FR-010) enables debugging without side effects
- Error messages required for edge cases (missing files, invalid config, skill failures)
- Traceability via task ID mappings (FR-004, FR-015)

### Phase Gate Status

**Gate: Before Planning** ✅
- Constitution exists (v1.0.0)
- Specification complete and clarified
- Constitution Check prepared

**Next Gate: Before Task Generation**
- Plan must pass constitution validation (re-check after Phase 1)
- Technical context must be concrete (currently concrete - no NEEDS CLARIFICATION)
- Project structure must be finalized (will complete in Project Structure section)
- No complexity justifications needed (Simplicity Bias passes)

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
# Template sources (in repository)
templates/
├── commands/
│   ├── tasks.md                    # Existing command (reference pattern)
│   └── jira-tasks.md               # NEW: Command definition source
├── tasks-template.md               # Existing (reference pattern)
├── jira-ticket-template.md         # NEW: Individual ticket structure template
└── jira-tickets-template.md        # NEW: Output file structure template

# Script modifications
scripts/
└── bash/
    └── check-prerequisites.sh      # MODIFIED: Add --for-jira-tasks flag support

# Release workflow modifications
.github/
└── workflows/
    ├── release.yml                 # MODIFIED: Add jira-tasks.md to triggers
    └── scripts/
        └── create-release-packages.sh  # MODIFIED: Process new command

# Installed files (after `specify init`, in user's project)
.claude/
└── commands/
    └── speckit.jira-tasks.md       # Copied from templates/commands/jira-tasks.md

.specify/
├── templates/
│   ├── jira-ticket-template.md    # Copied from templates/jira-ticket-template.md
│   └── jira-tickets-template.md   # Copied from templates/jira-tickets-template.md
└── scripts/
    └── bash/
        └── check-prerequisites.sh  # Copied with --for-jira-tasks support
```

**Structure Decision**:

This feature extends Spec Kit's command infrastructure following the established distribution pattern:

1. **Template Sources (in repo)**: Command and template files authored in `templates/` directory
2. **Release Processing**: GitHub workflow transforms templates into agent-specific formats
3. **Installation**: `specify init` downloads release ZIP and extracts to user's project
4. **No Python code**: Feature implemented purely as Claude slash command (markdown)

**Key Files**:

1. **`templates/commands/jira-tasks.md`** (source in repo): Main command definition
   - YAML frontmatter with `scripts: { sh: "scripts/bash/check-prerequisites.sh --json --for-jira-tasks" }`
   - Command description
   - Execution workflow (10 steps)
   - Task grouping algorithm (5-pass multi-dimensional)
   - JIRA integration via jira-db skill
   - Error handling for all edge cases

2. **`templates/jira-ticket-template.md`** (source in repo): 4-field structure
   - Subject (50-100 char, verb+object+purpose)
   - Description (6 subsections with bold labels)
   - Test Plan (4 subsections)
   - Technical Details (4 subsections)

3. **`templates/jira-tickets-template.md`** (source in repo): Output file format
   - Feature header with metadata (generated timestamp, counts)
   - Per-ticket sections (all 4 fields)
   - Task-to-Ticket mapping table
   - Statistics (by phase, by issue type)
   - Usage instructions

4. **`scripts/bash/check-prerequisites.sh`** (modified): Prerequisite validation
   - NEW: Accepts `--for-jira-tasks` flag
   - NEW: Returns `TASKS_FILE` and `JIRA_TICKETS_FILE` in JSON when flag present
   - Existing: Validates git repo, branch pattern, feature directory
   - Existing: Returns `FEATURE_DIR`, `FEATURE_NUM`, `FEATURE_NAME`, `BRANCH`

5. **`.github/workflows/release.yml`** (modified): Release triggers
   - Add `templates/commands/jira-tasks.md` to watch paths
   - Add `templates/jira-*.md` to watch paths

6. **`.github/workflows/scripts/create-release-packages.sh`** (modified): Package builder
   - Process new command template (convert placeholders, copy to agent dirs)
   - Copy jira-*.md templates to package

**No Python/src/ changes needed**: The existing `src/specify_cli/` structure remains unchanged as this feature operates entirely within Claude Code's command system and spec-kit's template infrastructure.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

**No violations identified** - Constitution Check passes all gates without requiring justification.

---

## Phase 1 Completion & Final Constitution Re-Check

### Artifacts Generated

**Phase 0: Research** ✅
- [research.md](research.md) - 7 technical decisions documented with rationale and alternatives

**Phase 1: Design & Contracts** ✅
- [data-model.md](data-model.md) - 7 entities defined with attributes, relationships, and state transitions
- [quickstart.md](quickstart.md) - 7 usage scenarios with expected outputs and troubleshooting
- No contracts/ directory - Feature does not expose APIs (CLI command only)

**Agent Context** ✅
- CLAUDE.md updated with technology stack from this feature

### Final Constitution Re-Check

**Gate: Before Task Generation** ✅

All requirements satisfied:

1. ✅ **Plan passes constitution validation**: All 6 core principles validated above
2. ✅ **Technical context is concrete**: No NEEDS CLARIFICATION markers remain
3. ✅ **Project structure finalized**: Concrete file paths specified (no Option labels)
4. ✅ **No complexity justifications needed**: Simplicity Bias passes without violations

### Design Decisions Summary

| Decision Area | Choice | Rationale |
|---------------|--------|-----------|
| Command Architecture | Markdown slash command | Follows existing Spec Kit patterns |
| Task Grouping | Multi-pass with story labels | Balances automation with user story structure |
| Documentation Style | AI-friendly principles | Maximizes AI agent effectiveness |
| JIRA Integration | jira-db skill | Reuses existing infrastructure |
| Error Handling | Fail fast with guidance | Supports Observability principle |
| Output Format | Markdown (jira-tickets.md) | Human-readable and version-controllable |
| Dry-Run Mode | Full preview capability | Risk mitigation for batch operations |

### Phase 2 Readiness

**Ready for `/speckit.tasks` command** ✅

All prerequisites met:
- Specification clarified and validated
- Technical plan complete with concrete decisions
- Data model defines all entities and relationships
- Quickstart provides clear usage patterns
- Constitution compliance verified
- No blocking unknowns or ambiguities remain

---

## Implementation Notes for Task Generation

When running `/speckit.tasks`, the following should guide task breakdown:

### User Story Mapping

- **P1 (Generate JIRA-sized Work Units)**: Core grouping algorithm, file I/O, task parsing
- **P2 (AI-Friendly 4-Field Structure)**: Ticket content generation, template application
- **P3 (Auto-Create via jira-db)**: JIRA API integration, ADF conversion, error handling
- **P4 (Interactive Review)**: User interaction, content editing (lowest priority, can defer)

### Key Implementation Files

From Project Structure section:
1. `templates/commands/jira-tasks.md` - Main command definition (source in repo)
2. `templates/jira-ticket-template.md` - 4-field structure template (source in repo)
3. `templates/jira-tickets-template.md` - Output file template (source in repo)
4. `scripts/bash/check-prerequisites.sh` - Modified to add --for-jira-tasks flag
5. `.github/workflows/release.yml` - Modified to trigger on new templates
6. `.github/workflows/scripts/create-release-packages.sh` - Modified to process new command

### Critical Clarifications from 2025-11-14

**Distribution Strategy** (Decision from clarification session):
- Command MUST be integrated into GitHub release workflow
- Source files in `templates/` directory (not `.specify/` or `.claude/`)
- Release process copies to `.claude/commands/` and `.specify/templates/` during `specify init`

**Script Architecture** (Decision from clarification session):
- Extend existing `check-prerequisites.sh` with `--for-jira-tasks` flag
- Do NOT create new `jira-tasks-setup.sh` script
- Script returns `TASKS_FILE` and `JIRA_TICKETS_FILE` in JSON

**Implementation Approach** (Decision from clarification session):
- Claude Code slash command (markdown specification)
- NOT Python script, NOT shell script, NOT hybrid
- Claude interprets instructions and calls jira-db skill directly

**YAML Frontmatter** (Decision from clarification session):
```yaml
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --for-jira-tasks
  ps: scripts/powershell/check-prerequisites.ps1 -Json -ForJiraTasks
```

### Testing Strategy

Manual verification against acceptance scenarios (per constitution - tests not requested):
- P1: Verify jira-tickets.md contains properly grouped tickets
- P2: Verify all 4 fields populated with AI-friendly content
- P3: Verify tickets created in JIRA with correct metadata
- P4: Verify interactive editing updates jira-tickets.md

### Implementation Phases

**Phase 1: Template Creation** (P1, P2)
- Create `templates/commands/jira-tasks.md` with command logic
- Create `templates/jira-ticket-template.md` with 4-field structure
- Create `templates/jira-tickets-template.md` with output format
- Implement task parsing, grouping algorithm (5-pass)
- Implement ticket content generation with AI-friendly style

**Phase 2: Script Extension** (P1)
- Modify `scripts/bash/check-prerequisites.sh` to accept `--for-jira-tasks` flag
- Add JSON output fields: `TASKS_FILE`, `JIRA_TICKETS_FILE`
- Validate tasks.md exists before command execution
- (Optional) Add PowerShell version for Windows support

**Phase 3: Release Integration** (distribution)
- Modify `.github/workflows/release.yml` to watch new template paths
- Modify `.github/workflows/scripts/create-release-packages.sh` to process new command
- Test release build locally
- Verify ZIP contains new command and templates in correct locations

**Phase 4: JIRA Integration** (P3)
- Implement jira-db skill invocation in command
- Implement markdown-to-ADF conversion
- Add `--create`, `--project`, `--team`, `--epic`, `--dry-run` flag handling
- Implement Epic validation, team resolution
- Add error handling for all jira-db failure modes

**Phase 5: Interactive Mode** (P4, optional)
- Implement `--interactive` flag handling
- Add ticket review prompts (Accept/Edit/Split/Merge/Skip/Quit)
- Implement field editing
- Implement split/merge with field regeneration
- Save changes back to jira-tickets.md

---

## References

- [spec.md](spec.md) - Feature specification with user stories and requirements
- [research.md](research.md) - Technical decisions and best practices
- [data-model.md](data-model.md) - Data structures and entity definitions
- [quickstart.md](quickstart.md) - Usage scenarios and troubleshooting
- [clarification-coverage.md](checklists/clarification-coverage.md) - Clarification session summary
- [.specify/memory/constitution.md](../../.specify/memory/constitution.md) - Project governance
