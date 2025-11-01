# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**etnovector** is a specification and implementation management framework designed to bridge natural language requirements with executable code. It implements a multi-stage workflow that converts feature descriptions into formal specifications, technical plans, and structured tasks.

**Project type**: Specification-to-Implementation Automation Framework (PowerShell-based)
**Tech stack**: PowerShell 7+, Markdown, Git, Claude AI API
**Primary language**: PowerShell (cross-platform scripts)

## Core Workflow

The project uses a **five-stage specification pipeline**:

1. **Specification** (`/speckit.specify`) → Creates `spec.md` with user stories, requirements, success criteria
2. **Clarification** (`/speckit.clarify`) → Resolves ambiguities (optional)
3. **Planning** (`/speckit.plan`) → Creates `plan.md` with architecture and technical decisions
4. **Task Generation** (`/speckit.tasks`) → Creates `tasks.md` with dependency-ordered work units
5. **Implementation** (`/speckit.implement`) → Executes tasks following the generated breakdown

Key concept: Artifacts drive the implementation. Each stage generates human-readable, version-controlled Markdown documents that feed the next stage.

## Project Structure

```
H:\git\etnovector/ (ROOT)
├── .claude/                       # Claude Code configuration
│   ├── commands/                  # 8 Claude Code slash commands (speckit.*)
│   │   ├── speckit.specify.md
│   │   ├── speckit.clarify.md
│   │   ├── speckit.plan.md
│   │   ├── speckit.tasks.md
│   │   ├── speckit.implement.md
│   │   ├── speckit.analyze.md
│   │   ├── speckit.checklist.md
│   │   └── speckit.constitution.md
│   └── settings.local.json
│
├── .specify/                      # Specify framework
│   ├── memory/constitution.md
│   ├── scripts/powershell/
│   └── templates/
│
├── research/                      # Technology research documents
│   ├── research-embedding-models.md
│   ├── research-databases.md
│   └── research-llm-apis.md
│
├── spec.md                        # ✅ Feature specification (6 user stories, 18 requirements)
├── plan.md                        # ✅ Implementation plan (11 weeks, 5 phases)
├── data-model.md                  # ✅ PostgreSQL schema + CARE principles
├── api-specification.md           # ✅ RESTful API endpoints (14+ endpoints)
├── quality-checklist.md           # Quality validation checklist
│
├── CLAUDE.md                      # This file - Claude Code guidance
├── README.md                      # Project information (PT-BR)
├── LICENSE                        # MIT License
└── .git/                          # Git repository
```

**Everything references the main branch directly - no separate feature folders.**

## Development Commands

### PowerShell Automation Scripts

These scripts are in `etnovector/.specify/scripts/powershell/`:

| Script | Purpose | Usage |
|--------|---------|-------|
| `create-new-feature.ps1` | Create feature branch and spec template | `.specify/scripts/powershell/create-new-feature.ps1 [-Json] [-ShortName name] "description"` |
| `setup-plan.ps1` | Initialize plan template | `.specify/scripts/powershell/setup-plan.ps1 [-Json]` |
| `check-prerequisites.ps1` | Validate feature readiness before implementation | `.specify/scripts/powershell/check-prerequisites.ps1 -Json [-RequireTasks]` |
| `update-agent-context.ps1` | Update AI agent knowledge base with new technologies | `.specify/scripts/powershell/update-agent-context.ps1 -AgentType claude` |
| `common.ps1` | Shared utility functions | Sourced by other scripts |

### Claude Code Slash Commands

Execute via `/speckit.COMMAND`:

| Command | Stage | Purpose |
|---------|-------|---------|
| `/speckit.specify "description"` | 1 | Generate feature specification from natural language |
| `/speckit.clarify` | 1.5 | Resolve ambiguities marked [NEEDS CLARIFICATION] in spec |
| `/speckit.constitution` | 0 | View or update project governance document |
| `/speckit.plan` | 2 | Generate technical implementation plan from spec |
| `/speckit.tasks` | 3 | Generate task breakdown from plan |
| `/speckit.analyze` | 3.5 | Cross-artifact consistency validation (read-only) |
| `/speckit.checklist` | 4 | Generate quality checklists for feature |
| `/speckit.implement` | 5 | Execute all tasks from tasks.md |

### Common Git Operations

```bash
# Create feature branch (automated via create-new-feature.ps1)
git checkout -b NNN-feature-name

# View current feature branch
git branch --show-current

# Update branches from remote
git fetch origin

# Push feature branch
git push -u origin NNN-feature-name

# Create pull request
gh pr create --title "Feature title" --body "Description"
```

## Architecture & Design Patterns

### Pattern 1: Specification Pipeline

Features flow through artifact stages:
```
Natural Language Description
  ↓ (spec.md)
Functional Specification (What)
  ↓ (plan.md)
Technical Plan (How)
  ↓ (tasks.md)
Executable Tasks (When/Who)
  ↓ (implementation)
Implemented Feature
```

Each stage:
- Preserves context from previous stage
- Adds new information (decisions, architecture, schedule)
- Is human-readable and AI-processable
- Is version-controlled in Git

### Pattern 2: Artifact-First Design

Documentation drives implementation, not vice versa:

- **spec.md** (WHAT): User stories, acceptance criteria, success metrics, security/compliance requirements
- **plan.md** (HOW): Architecture decisions, technology stack justification, project structure, complexity analysis
- **tasks.md** (WHEN/WHO): Task breakdown by user story phase, dependencies, assignees, estimates
- **data-model.md**: Entity definitions, relationships
- **contracts/**: API specifications, data schemas (enables parallel development)

### Pattern 3: User Story-Driven Task Organization

Tasks organized hierarchically:
- Phase 1: Setup (shared infrastructure)
- Phase 2: Foundational (prerequisites)
- Phase 3+: User Stories (each independently implementable)
- Phase N: Polish (cross-cutting concerns)

**Key principle**: Each user story must be independently:
- Implementable (no cross-story blocking)
- Testable (acceptance criteria verifiable alone)
- Deployable (demo-able without other stories)
- Valuable (delivers user value independently)

### Pattern 4: PowerShell-Based Automation

Why PowerShell:
- Cross-platform (Windows, Linux, macOS via Core)
- Rich object model (JSON parsing, type safety)
- Better error handling than shell scripts
- Excellent GitHub CLI integration

### Pattern 5: Project Constitution

Define non-negotiable principles in `constitution.md`:
- Core principles (e.g., "Library-First", "Test-First", "CLI-First")
- Additional constraints (security, compliance, performance)
- Development workflow requirements
- Governance rules

All technical decisions in plan.md should justify alignment with constitution.

### Pattern 6: Branch-Numbered Feature Organization

Feature directories follow naming: `[###]-[feature-name]`
```
specs/
├── 001-user-auth/
├── 002-payment-system/
└── 003-analytics/
```

Benefits:
- Sequential numbering prevents collisions
- Branch name matches spec directory (`001-user-auth` branch → `specs/001-user-auth/` directory)
- Enables parallel feature development with clear progress
- Stable feature numbering even if names change

## Configuration

### Claude Code Local Settings

File: `.claude/settings.local.json`

Contains permission whitelist for Bash commands used in scripts (tree, dir, Get-ChildItem, etc.).

### Project Constitution (To Be Completed)

File: `etnovector/.specify/memory/constitution.md`

Fill in:
- **Core Principles**: Your project's non-negotiable architectural rules
- **Additional Constraints**: Security, performance, compliance requirements
- **Development Workflow**: Code review process, testing standards, deployment approval
- **Governance**: How constitution is enforced and amended

This constitution gates all technical decisions in plan.md stage.

## Key Files & Their Purposes

| File | Purpose |
|------|---------|
| `.claude/commands/*.md` | Slash command definitions for AI workflow automation |
| `.specify/templates/*.md` | Markdown templates for specs, plans, tasks |
| `.specify/memory/constitution.md` | Project governance (non-negotiable principles) |
| `.specify/scripts/powershell/*.ps1` | Automation for feature creation, planning, validation |
| `specs/[###]-feature/spec.md` | Functional specification (requirements + user stories) |
| `specs/[###]-feature/plan.md` | Technical implementation plan |
| `specs/[###]-feature/tasks.md` | Task breakdown organized by user story + phase |
| `specs/[###]-feature/data-model.md` | Entity definitions and relationships |
| `specs/[###]-feature/contracts/*.md` | API contracts, data schemas |

## Important Considerations When Working on This Project

1. **Preserve artifact structure**: Each artifact (spec, plan, tasks) has a specific purpose. Don't merge them—keep them separate for AI and human processing.

2. **Template-driven development**: New artifacts should use the templates in `.specify/templates/`. This ensures consistency and AI-readability.

3. **PowerShell is the automation language**: Don't add Python scripts, shell scripts, or other languages for automation. Use PowerShell with cross-platform Core.

4. **Constitution gates decisions**: Technical choices in plan.md must reference and justify alignment with constitution.md principles.

5. **User stories are work units**: Task breakdown in tasks.md should decompose user stories, not arbitrary features. Each story should be independently testable.

6. **Version control the artifacts, not the implementation**: The point is that specs/plans/tasks drive implementation. Future Claude instances will read these artifacts to understand the feature.

7. **Feature branches match feature directories**: Branch name `NNN-feature-name` should match `specs/NNN-feature-name/`.

8. **AI gates are optional, human gates are mandatory**: Pre-implementation checklists should be 100% complete before calling `/speckit.implement`.

## Feature Development Workflow Example

```
1. Create feature: /speckit.specify "Allow users to export reports as PDF"
   → Creates specs/NNN-export-reports/ with spec.md

2. Plan implementation: /speckit.plan
   → Creates plan.md with architecture, tech choices, file structure

3. Generate tasks: /speckit.tasks
   → Creates tasks.md breaking down by user story:
     - Setup: PDF library selection
     - User Story 1: Basic export
     - User Story 2: Template customization
     - Polish: Error handling

4. Implement:
   - Follow tasks.md task-by-task
   - Mark tasks complete with [x]
   - Ensure each user story passes acceptance criteria before moving to next

5. Validate: /speckit.analyze
   → Checks spec/plan/tasks consistency

6. Complete: /speckit.implement
   → Runs final checks and marks feature ready
```

## Notes for Claude Code Future Instances

- The spec at `specs/[###]-feature-name/spec.md` is your requirements document
- The plan at `specs/[###]-feature-name/plan.md` explains architectural choices and current project structure
- The tasks at `specs/[###]-feature-name/tasks.md` is your breakdown into work units
- Always check the constitution before making architecture decisions
- When adding new technology or patterns, update `specs/[feature]/plan.md` with rationale
- If you need to add new slash commands, put them in `etnovector/.claude/commands/`
