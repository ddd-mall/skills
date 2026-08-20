# DDD Mall Skills

[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)
[![skills.sh](https://skills.sh/b/ddd-mall/skills)](https://skills.sh/ddd-mall/skills)

DDD Mall Skills is an open collection of reusable Agent Skills maintained by
[ddd-mall](https://github.com/ddd-mall). It captures practical guidance for
specification-driven delivery, software design, implementation, verification,
and engineering governance.

The skills in this repository focus on observable outcomes and reviewable
evidence. They help coding agents work within the conventions, architecture,
quality requirements, and authorization boundaries of the repository they are
changing, without imposing unnecessary ceremony on routine work.

## Available Skills

| Skill | Purpose |
| --- | --- |
| [`spec-dev`](skills/spec-dev/) | Outcome-focused specification, implementation, analysis, convergence, and verification for feature changes, bugfixes, and requirement revisions. |

## Installation

List the available skills with:

```bash
npx skills add ddd-mall/skills --list
```

Install a specific skill for a supported coding agent:

```bash
npx skills add ddd-mall/skills --skill <skill-name> --agent <agent> --yes
```

For example, install `spec-dev` for Codex:

```bash
npx skills add ddd-mall/skills --skill spec-dev --agent codex --yes
```

The core workflow is instruction-only and does not require a runtime
dependency. The optional deterministic schema and evaluation tools require
Python 3.10 or newer and the packages in
[`skills/spec-dev/requirements.txt`](skills/spec-dev/requirements.txt).
Installing the skill does not install those Python packages automatically.

## Repository Layout

```text
.
|-- skills/
|   `-- <skill-name>/
|       |-- SKILL.md
|       |-- agents/       # Optional agent-specific metadata
|       |-- references/   # Optional supporting guidance
|       |-- scripts/      # Optional deterministic tooling
|       |-- schemas/      # Optional machine-readable contracts
|       `-- evals/        # Optional behavioral evaluation assets
|-- tests/
|-- LICENSE
`-- README.md
```

Each published skill is self-contained and uses `SKILL.md` as its entry point.
Supporting resources are included only when they materially improve the
skill's decisions or make verification more reliable.

## Quality Principles

- Preserve user intent and the target repository's established constraints.
- Define completion through observable behavior and proportionate evidence.
- Keep descriptions precise so skills are discovered only for relevant work.
- Make permissions, external effects, skipped checks, and residual risks clear.
- Validate scripts and schemas deterministically, then evaluate consequential
  behavior with realistic tasks.
- Keep skills portable across supported agents unless a limitation is stated
  explicitly.

## Contributing

Issues and pull requests are welcome. A contribution should have a clear scope,
avoid project-specific secrets or private data, and include verification
appropriate to its risk. Review all instructions and scripts before installing
a skill from any source.

## License

Copyright 2026 DDD Mall Skills contributors.

Licensed under the [Apache License 2.0](LICENSE). Unless required by applicable
law or agreed to in writing, software and documentation distributed under the
License are provided on an "AS IS" basis, without warranties or conditions of
any kind.
