# spec-dev

`spec-dev` helps coding agents deliver feature changes, bugfixes, and
requirement revisions through outcome-focused specification, implementation,
analysis, and verification. It adapts the amount of specification and evidence
to the change instead of imposing one fixed process.

The normative agent instructions are in [SKILL.md](SKILL.md). This document is
for people installing and invoking the skill.

## When To Use It

Use `spec-dev` when a coding task benefits from one or more of these outcomes:

- clarify intent, scope, acceptance criteria, or quality goals;
- implement a feature from an existing or newly created specification;
- diagnose and repair a bug while protecting unchanged behavior;
- analyze conflicts or gaps among requirements, design, tasks, code, and tests;
- converge a completed change against its intended behavior and evidence;
- coordinate a substantial change across specifications, implementation, and
  verification.

Routine, low-risk edits remain lightweight. The skill does not require a full
specification package when the request, code, and tests already make the
desired behavior clear.

## Installation

The commands below require Node.js and use the [`skills`](https://skills.sh/)
CLI through `npx`.

List the skills available in this repository:

```bash
npx skills add ddd-mall/skills --list
```

Install `spec-dev` into the current project:

```bash
npx skills add ddd-mall/skills --skill spec-dev --agent <agent> --yes
```

Common agent identifiers and project installation paths are:

| Harness | Agent identifier | Project path |
| --- | --- | --- |
| Codex | `codex` | `.agents/skills/spec-dev/` |
| Claude Code | `claude-code` | `.claude/skills/spec-dev/` |
| Cursor | `cursor` | `.agents/skills/spec-dev/` |
| OpenCode | `opencode` | `.agents/skills/spec-dev/` |
| Gemini CLI | `gemini-cli` | `.agents/skills/spec-dev/` |

For example:

```bash
npx skills add ddd-mall/skills --skill spec-dev --agent codex --yes
npx skills add ddd-mall/skills --skill spec-dev --agent claude-code --yes
```

Install for more than one harness in the same project:

```bash
npx skills add ddd-mall/skills \
  --skill spec-dev \
  --agent codex \
  --agent claude-code \
  --yes
```

Add `--global` to install the skill for the current user instead of only the
current project. Add `--copy` when the environment cannot use the CLI's default
symlink installation.

```bash
npx skills add ddd-mall/skills \
  --skill spec-dev \
  --agent codex \
  --global \
  --yes
```

Review the installed instructions and supporting scripts before using a Skill
from any source. Installation does not expand the active harness's permissions
or authorize external or destructive actions.

## Usage

After installation, describe the change normally. A harness that supports
automatic Skill discovery can select `spec-dev` from its description when the
task matches.

To request it explicitly, name the Skill in the prompt:

```text
Use the spec-dev skill to implement account email verification and provide
acceptance evidence.
```

Harnesses that support `$` Skill mentions, such as Codex, can use:

```text
Use $spec-dev to analyze the active checkout specifications for conflicts.
```

Invocation syntax is controlled by the harness. Naming `spec-dev` in ordinary
language is the portable fallback when a harness does not support `$spec-dev`.

The `skills` CLI can also prepare a one-off prompt without installing the
Skill:

```bash
npx skills use ddd-mall/skills@spec-dev
```

To start a supported agent interactively with that generated prompt:

```bash
npx skills use ddd-mall/skills --skill spec-dev --agent codex
```

## Example Requests

Feature delivery:

```text
Use spec-dev to add saved delivery addresses. Clarify material product
ambiguities, implement the accepted behavior, and verify the result.
```

Bugfix:

```text
Use spec-dev to diagnose and fix duplicate order submission. Preserve existing
checkout behavior and add regression evidence.
```

Read-only analysis:

```text
Use spec-dev to analyze requirement.md, design.md, and tasks.md for semantic
conflicts. Report findings without modifying the repository.
```

Convergence review:

```text
Use spec-dev to assess whether this change is complete against its accepted
outcomes, tests, and repository constraints. Report missing evidence and
residual risks.
```

The Skill decides whether separate `requirement.md`, `design.md`, `tasks.md`,
or summary artifacts add value. Repository rules and explicit user requests
remain authoritative.

## Optional Deterministic Tools

The core Skill is instruction-only. Its optional schema validation and
machine-readable change tooling require Python 3.10 or newer plus the packages
in [requirements.txt](requirements.txt).

From the target repository, create an isolated environment and install the
optional dependency:

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -r <skill-root>/requirements.txt
```

Replace `<skill-root>` with the installed `spec-dev` directory shown by the
active harness. Inspect the available deterministic commands with:

```bash
.venv/bin/python <skill-root>/scripts/specctl.py --help
```

`specctl.py` provides read-only `status`, `validate`, `analyze`, `converge`,
and `archive --dry-run` checks for repositories using the optional
machine-readable change contract. See
[references/machine-state.md](references/machine-state.md) before adopting
that contract. The scripts support the workflow but do not replace semantic
review or executable tests.

## Update, Inspect, And Remove

List project installations for one harness:

```bash
npx skills list --agent codex
```

Update the project installation:

```bash
npx skills update spec-dev --project --yes
```

For a global installation, replace `--project` with `--global`.

Remove the Skill from a specific harness:

```bash
npx skills remove spec-dev --agent codex --yes
```

Use the same scope flag that was selected during installation when updating or
removing a global installation.

## Package Contents

- [SKILL.md](SKILL.md): normative outcome, authority, workflow, and completion
  contracts loaded by the agent;
- [`references/`](references/): focused guidance for bugfixes, analysis,
  convergence, change state, roles, and coordination;
- [`scripts/`](scripts/): optional deterministic validation tools;
- [`schemas/`](schemas/): JSON Schemas used by the optional tools;
- [`evals/`](evals/): behavioral evaluation cases, records, and durable traces;
- [`agents/openai.yaml`](agents/openai.yaml): optional UI metadata for
  compatible harnesses.
