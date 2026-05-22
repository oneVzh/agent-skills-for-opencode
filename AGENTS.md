# AGENTS.md

This file provides guidance to AI coding agents when working with code on this machine. It lives at `~/.config/opencode/AGENTS.md` and applies globally across all projects. Project-local `AGENTS.md` (or `.opencode/AGENTS.md`) files take precedence for project-specific instructions.

## Global Configuration Overview

This directory is the global OpenCode configuration at `~/.config/opencode/`. It contains three asset directories shared across all projects on this machine:

- **`skills/`** — reusable workflows with steps and exit criteria (the _how_)
- **`commands/`** — user-facing slash commands (`/spec`, `/plan`, `/build`, `/review`, `/ship`, etc.)
- **`agents/`** — specialist personas usable as subagents via the `task` tool

## OpenCode Skill-Driven Execution

OpenCode uses a **skill-driven execution model** powered by the `skill` tool and this configuration's `skills/` directory.

### Core Rules

- If a task matches a skill, you MUST invoke it
- Skills are located in `skills/<skill-name>/SKILL.md`
- Never implement directly if a skill applies
- Always follow the skill instructions exactly (do not partially apply them)

### Intent → Skill Mapping

The agent should automatically map user intent to skills:

- Feature / new functionality → `spec-driven-development`, then `incremental-implementation`, `test-driven-development`
- Planning / breakdown → `planning-and-task-breakdown`
- Bug / failure / unexpected behavior → `debugging-and-error-recovery`
- Code review → `code-review-and-quality`
- Refactoring / simplification → `code-simplification`
- API or interface design → `api-and-interface-design`
- UI work → `frontend-ui-engineering`

### Lifecycle Mapping (Implicit Commands)

OpenCode supports slash commands defined in the `commands/` directory. Users can invoke `/spec`, `/plan`, `/build`, `/review`, `/test`, `/ship`, and `/code-simplify` directly.

When the user does not explicitly invoke a slash command, the agent should internally follow this lifecycle:

- DEFINE → `spec-driven-development`
- PLAN → `planning-and-task-breakdown`
- BUILD → `incremental-implementation` + `test-driven-development`
- VERIFY → `debugging-and-error-recovery`
- REVIEW → `code-review-and-quality`
- SHIP → `shipping-and-launch`

### Execution Model

For every request:

1. Determine if any skill applies (even 1% chance)
2. Invoke the appropriate skill using the `skill` tool
3. Follow the skill workflow strictly
4. Only proceed to implementation after required steps (spec, plan, etc.) are complete

### Anti-Rationalization

The following thoughts are incorrect and must be ignored:

- "This is too small for a skill"
- "I can just quickly implement this"
- "I'll gather context first"

Correct behavior:

- Always check for and use skills first

## Orchestration: Personas, Skills, and Commands

Three composable layers. They have different jobs and should not be confused:

- **Skills** (`skills/<name>/SKILL.md`) — workflows with steps and exit criteria. The _how_. Mandatory hops when an intent matches.
- **Personas** (`agents/<role>.md`) — roles with a perspective and an output format. The _who_.
- **Slash commands** (`commands/*.md`) — user-facing entry points. The _when_. The orchestration layer.

Composition rule: **the user (or a slash command) is the orchestrator. Personas do not invoke other personas.** A persona may invoke skills.

The only multi-persona orchestration pattern endorsed is **parallel fan-out with a merge step** — used by `/ship` to run `code-reviewer`, `security-auditor`, and `test-engineer` concurrently via the `task` tool, then synthesize their reports. Do not build a "router" persona that decides which other persona to call; that's the job of slash commands and intent mapping.

See `agents/README.md` for the decision matrix.

## Using Subagents

The personas in `agents/` are available as OpenCode subagents via the `task` tool. Launch them with `subagent_type` matching the persona's `name` field:

- `code-reviewer` — Senior Staff Engineer, five-axis review
- `security-auditor` — Security Engineer, vulnerability detection
- `test-engineer` — QA Engineer, test strategy and coverage analysis

The `subagent_type` values come from the `name` frontmatter field in each agent file. Subagents report results back to the main agent. They cannot spawn other subagents, and their `hooks`, `mcpServers`, and `permissionMode` frontmatter fields are ignored.
