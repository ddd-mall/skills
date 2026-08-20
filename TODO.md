# Cross-Harness Skill Runtime TODO

## Purpose

Track the compatibility problems that arise when the same Skill runs in
different agent harnesses, evaluate candidate solutions, and define an
incremental path toward a reusable Adapter runtime for `spec-dev` and future
Skills.

This document is a design and implementation backlog. It does not grant tools,
filesystem access, network access, credentials, approval authority, or any
other capability to a Skill or Adapter.

## Current State

- [x] `spec-dev/SKILL.md` avoids depending on a specific harness, tool name,
  invocation syntax, or sub-agent API.
- [x] Deterministic `spec-dev` checks are available through
  `skills/spec-dev/scripts/specctl.py`.
- [x] The base Skill remains usable as instructions when optional tooling is
  unavailable.
- [ ] There is no executable, versioned contract for runtime capability
  discovery, operation invocation, or normalized results.
- [ ] There is no shared Adapter runtime that another Skill can reuse.
- [ ] There is no CLI/MCP equivalence test suite.
- [ ] There is no real cross-harness conformance matrix.
- [ ] `skills/spec-dev/references/harness-compatibility.md` does not currently
  exist. A future document at that path would describe intended behavior, but
  documentation alone could not enforce harness behavior.

## Problems To Solve

### 1. Tool availability differs by harness

One harness may expose a shell and native file-editing tools, another may expose
only MCP tools, and another may prohibit subprocess execution. Tool names,
argument formats, result formats, timeouts, streaming behavior, and error
semantics also differ.

Consequences:

- Skill instructions that name one harness's tools are not portable.
- Reimplementing tool selection in every Skill causes drift and duplication.
- A declared capability may be installed but unusable in the active sandbox.
- Falling back silently can produce weaker evidence than the Skill claims.

### 2. Permission and approval models differ

Sandbox boundaries, allowlists, approval prompts, credential injection, network
policy, and destructive-action controls belong to the active harness. A Skill
can request behavior and declare expected effects, but cannot enforce or expand
the harness's authority.

Consequences:

- `allowed-tools` and similar metadata are compatibility hints, not portable
  security boundaries.
- An Adapter must not reinterpret a harness denial as an ordinary domain error.
- Generated harness configuration must not silently loosen permissions,
  approvals, credentials, tool allowlists, or network access.

### 3. Installation compatibility is not runtime compatibility

A Skill may be discoverable and load successfully while its optional scripts,
Python dependencies, shell, MCP server, repository paths, or write access are
unavailable.

Consequences:

- Installation success cannot be used as evidence that an operation can run.
- Compatibility must be checked in the active invocation environment.
- The Skill needs an instruction-only fallback that reports reduced assurance.

### 4. Documentation cannot constrain actual harness behavior

`SKILL.md` and compatibility references can describe desired behavior, but the
harness controls tool dispatch and authorization before Adapter code may run.

Consequences:

- Normative prose remains necessary for agent decisions but is insufficient for
  deterministic interoperability.
- Important behavior needs schemas, executable validation, contract tests, and
  observed cross-harness evaluations.
- Results must distinguish verified runtime facts from declared capabilities.

### 5. A universal low-level tool wrapper would be unsafe and brittle

Wrapping generic shell, file editing, Git, network, and sub-agent APIs would
create a lowest-common-denominator API while duplicating harness security and
lifecycle behavior.

Consequences:

- The Adapter could accidentally become a permission bypass or confused deputy.
- Harness-native features would be hidden or degraded.
- The compatibility surface would grow with every vendor-specific tool.

The reusable layer should therefore expose high-level Skill domain operations,
not a second generic agent tool system.

### 6. Shared runtime packaging can break Skill self-containment

This repository currently publishes self-contained Skill directories. A shared
root package may not be copied by an installer that installs only one Skill.

Consequences:

- A monorepo import such as `../../runtime` may work in development and fail
  after installation.
- Mandatory MCP or SDK dependencies would make the base Skill less portable.
- Runtime and Skill protocol versions need explicit compatibility rules.

### 7. Capability claims can become stale

Static manifests cannot observe a missing executable, rejected subprocess,
read-only workspace, unavailable dependency, expired credential, or runtime
policy change.

Consequences:

- Capability negotiation needs both declarations and runtime probes.
- Probes must be safe, bounded, and must not perform privileged mutations.
- A probe result is local to the current environment and invocation context.

## Design Principles

- Keep `SKILL.md` portable and useful without an Adapter.
- Standardize contracts and lifecycle, not every harness-native tool.
- Keep domain logic independent of CLI, MCP, and harness SDK transport.
- Let the harness remain the final authority for sandboxing and approval.
- Declare expected side effects before execution when the transport supports it.
- Treat actual runtime probes as stronger evidence than static declarations.
- Never convert unavailable verification into successful evidence.
- Use versioned schemas and machine-readable results.
- Make read-only operations the first supported surface.
- Preserve each published Skill's self-contained installation behavior.
- Stabilize the abstraction only after at least two materially different Skills
  have exercised it.

## Candidate Solutions

### Candidate A: Documentation-only compatibility guidance

Add a compatibility reference describing tool alternatives, permissions, and
fallback behavior for every supported harness.

Advantages:

- Small implementation cost.
- Works in environments that cannot execute an Adapter.
- Useful as the final fallback and for human review.

Limitations:

- Cannot enforce actual harness behavior.
- Cannot normalize tool results or errors.
- Becomes repetitive as Skills and harnesses increase.
- Provides no executable compatibility evidence.

Decision: retain as a fallback and explanation layer, but do not use it as the
primary compatibility mechanism.

### Candidate B: Per-harness instructions inside every Skill

Give each Skill separate Codex, Claude Code, Gemini CLI, OpenCode, Cursor, and
other harness instructions.

Advantages:

- Can use each harness's native features directly.
- Requires no shared runtime.

Limitations:

- Produces an `O(skills * harnesses)` maintenance surface.
- Security and error-handling rules will drift.
- Makes `SKILL.md` larger and harms progressive disclosure.
- Still cannot guarantee the harness follows the instructions.

Decision: reject as the primary architecture. Permit small harness-specific
installation notes or bridges outside the domain instructions.

### Candidate C: One universal low-level tool API

Wrap shell, file access, Git, network calls, approvals, and agents behind a
single API.

Advantages:

- Appears to give every Skill one tool vocabulary.
- Could simplify narrow demonstrations.

Limitations:

- Duplicates security and lifecycle responsibilities already owned by harnesses.
- Encourages permission bypasses and ambiguous authorization.
- Cannot faithfully represent every harness's native semantics.
- Creates a large, unstable vendor-compatibility surface.

Decision: reject.

### Candidate D: Shared domain Adapter runtime with transport bridges

Build a small runtime that registers high-level operations from Skill-specific
providers. Expose those operations through a default CLI bridge and an optional
MCP bridge. Add native bridges only when they provide a demonstrated benefit.

Advantages:

- Reuses capability negotiation, validation, diagnostics, result semantics,
  security checks, and lifecycle behavior across Skills.
- Keeps domain logic independent of transport.
- CLI works well for coding agents and CI; MCP covers no-shell and remote cases.
- Allows deterministic contract and equivalence testing.
- Does not need to wrap generic harness tools.

Limitations:

- Requires a versioning and packaging strategy.
- Cannot run after a harness rejects the call.
- MCP adds dependencies and operational complexity.
- A single initial Skill may bias the abstraction toward its own domain.

Decision: recommended.

### Candidate E: MCP-only common layer

Publish every Skill operation through one MCP server.

Advantages:

- Uses a broadly adopted tool interoperability protocol.
- Supports structured discovery and remote execution.
- Can centralize stateful or authenticated domain integrations.

Limitations:

- Not every harness enables MCP.
- Heavier than a local CLI for deterministic repository checks.
- Server configuration, transport security, and lifecycle become prerequisites.
- Makes an optional interoperability mechanism a mandatory dependency.

Decision: provide MCP as an optional bridge, not the base runtime contract.

## Recommended Architecture

```text
Portable SKILL.md
        |
        v
Skill Provider
  - registers high-level domain operations
  - owns domain validation and result data
        |
        v
Shared Adapter Runtime
  - protocol negotiation
  - capability discovery and safe probes
  - input/output schema validation
  - operation registry and dispatch
  - diagnostics and normalized errors
  - workspace boundary checks
        |
        +--------------------+
        |                    |
        v                    v
CLI Bridge              MCP Bridge
default                 optional, read-only v1
        |                    |
        +----------+---------+
                   v
Harness-native sandbox, approvals, credentials, and tool policy
```

### Responsibilities

Shared Adapter Runtime:

- Define and validate versioned request, result, capability, and effect schemas.
- Register namespaced operations without importing harness-specific APIs.
- Resolve and validate workspace-relative paths, including symlink escape.
- Report dependency and environment problems consistently.
- Support deterministic JSON input and output.
- Avoid ambient network access or credential discovery unless an operation
  explicitly requires them.

Skill Provider:

- Define high-level operations such as `spec.validate` or `spec.converge`.
- Reuse one domain implementation across all bridges.
- Declare required inputs, expected effects, and optional dependencies.
- Return domain findings and evidence without claiming harness authorization.
- Supply an instruction-only fallback where meaningful.

Transport Bridge:

- Convert CLI, MCP, or native SDK requests into the common invocation contract.
- Preserve structured results without changing domain meaning.
- Translate transport failures separately from domain findings.
- Avoid changing harness configuration or permission policy at runtime.

Harness:

- Own tool registration, sandbox enforcement, approval prompts, credentials,
  network policy, process execution, cancellation, and resource limits.
- May deny an invocation before the Adapter executes.

## Proposed Protocol

### Stable commands

```text
adapter describe
adapter doctor
adapter invoke <operation>
```

`describe` reports static protocol and operation metadata. `doctor` performs
safe environment checks and reports currently observed capabilities. `invoke`
executes one namespaced domain operation.

### Capability description

Each capability should include at least:

- protocol version;
- operation name and operation version;
- human-readable summary;
- input and result schema identifiers;
- read-only or mutating classification;
- expected filesystem, subprocess, network, credential, and external-service
  effects;
- required and optional runtime dependencies;
- whether an instruction-only fallback exists.

Static capability declarations must not claim that permission is currently
granted. Runtime observations should include their scope and timestamp.

### Invocation request

```json
{
  "protocolVersion": "1.0",
  "operation": "spec.validate",
  "operationVersion": "1.0",
  "workspaceRoot": "/absolute/project/path",
  "arguments": {},
  "requestId": "caller-generated-id"
}
```

The runtime must reject unknown fields where ambiguity would be unsafe, reject
unsupported major versions, and validate `workspaceRoot` before domain code
runs.

### Normalized result

```json
{
  "protocolVersion": "1.0",
  "operation": "spec.validate",
  "outcome": "pass",
  "summary": "Validation completed",
  "data": {},
  "findings": [],
  "evidence": [],
  "warnings": []
}
```

Allowed outcomes:

- `pass`: the operation ran and its domain condition passed;
- `findings`: the operation ran and produced actionable domain findings;
- `input_error`: the caller supplied invalid or unsupported input;
- `environment_error`: an executable, dependency, path, or runtime facility was
  unavailable after the Adapter started;
- `internal_error`: the Adapter or provider failed unexpectedly.

`approval_required` and `denied` are deliberately excluded. Those states are
owned by the harness and may occur before the Adapter starts. A bridge may
report a transport-level denial, but must not fabricate an Adapter result.

### Versioning

- Use semantic versions for the Adapter protocol and every operation.
- Reject unsupported major versions.
- Permit additive optional fields within a major version.
- Keep result meaning stable within a major version.
- Include the provider and runtime versions in diagnostics.
- Define a deprecation period before removing an operation or field.

## Packaging Options

### Option 1: Runtime installed as a separate package

Useful when several Skills are installed together or a managed environment can
guarantee the dependency. This minimizes duplication but weakens standalone
Skill installation.

### Option 2: Vendor a lightweight runtime into every published Skill

Preserves self-containment and works with installers that copy only the selected
Skill directory. Generated copies must carry a runtime version and must be
checked for drift.

### Option 3: Build-time bundling from one source

Maintain one runtime source tree in this repository and produce self-contained
Skill artifacts during release. This combines source-level reuse with portable
installation, at the cost of a release build step.

Recommendation: use build-time bundling for published artifacts. During the
first implementation, keep `spec-dev` operable without the runtime so the
packaging mechanism can be validated before it becomes a dependency.

## Proposed Repository Layout

```text
runtime/
|-- schemas/
|   |-- adapter-capabilities.schema.json
|   |-- adapter-request.schema.json
|   `-- adapter-result.schema.json
|-- python/skill_adapter/
|   |-- registry.py
|   |-- runtime.py
|   |-- diagnostics.py
|   `-- security.py
`-- bridges/
    |-- cli.py
    `-- mcp.py

skills/spec-dev/
|-- provider/
|   `-- operations.py
|-- scripts/
|   `-- specctl.py
`-- references/
    `-- harness-compatibility.md
```

This layout is provisional. It must not be finalized until the installation
and release process proves that a single Skill receives all required runtime
files.

## `spec-dev` Initial Operation Set

Start with read-only, deterministic operations:

```text
spec.status
spec.validate
spec.analyze
spec.converge
spec.archive-preview
```

Do not include generic file editing, shell execution, Git mutation, arbitrary
test execution, approval handling, credential management, or sub-agent creation
in the Adapter API.

## Implementation Backlog

### Phase 0: Record boundaries and baseline behavior

- [ ] Add `skills/spec-dev/references/harness-compatibility.md` and route to it
  only when installation or runtime compatibility is relevant.
- [ ] Document which `specctl.py` commands and outputs are current public
  behavior.
- [ ] Capture baseline tests before refactoring `specctl.py`.
- [ ] Record supported Python versions and optional dependency behavior.
- [ ] Decide whether release artifacts are built, vendored, or separately
  packaged; test the decision with the actual Skill installer.

Exit criteria: existing `spec-dev` behavior is documented and protected by
tests, and the packaging experiment proves where shared runtime files must live.

### Phase 1: Define versioned contracts

- [ ] Add capability, request, result, finding, evidence, and effect schemas.
- [ ] Define protocol and operation version negotiation.
- [ ] Define exit-code mapping for CLI outcomes.
- [ ] Define transport error versus Adapter result semantics.
- [ ] Define cancellation, timeout, and output-size behavior.
- [ ] Add schema fixtures for valid and invalid examples.
- [ ] Add forward-compatibility and unknown-major-version tests.

Exit criteria: contracts can be reviewed and tested without any CLI or MCP
implementation.

### Phase 2: Extract the shared runtime and `spec-dev` provider

- [ ] Extract domain logic from `specctl.py` without changing observable
  behavior.
- [ ] Introduce a provider registry with namespaced operation identifiers.
- [ ] Implement workspace containment and symlink-escape protection once in the
  runtime.
- [ ] Keep CLI parsing and presentation outside domain operations.
- [ ] Preserve the existing `specctl.py` entrypoint as a compatibility shim.
- [ ] Add `describe --json` and `doctor --json`.
- [ ] Normalize JSON results while retaining documented legacy output until a
  migration is complete.

Exit criteria: legacy CLI tests pass, normalized CLI contract tests pass, and
domain operations can be invoked in process without CLI parsing.

### Phase 3: Add the CLI bridge

- [ ] Implement deterministic stdin/stdout JSON invocation.
- [ ] Keep diagnostics on stderr when stdout is a machine-readable result.
- [ ] Define stable exit codes without encoding harness approval states.
- [ ] Test invocation from repository root and an explicit workspace root.
- [ ] Test missing Python dependency, missing executable, malformed JSON,
  unsupported version, timeout, cancellation, and oversized output.
- [ ] Verify operation behavior in a read-only workspace.

Exit criteria: the CLI bridge works in coding agents and CI without depending
on a particular harness API.

### Phase 4: Add an optional read-only MCP bridge

- [ ] Use the official MCP SDK for the selected implementation language.
- [ ] Start with stdio transport and no remote listener.
- [ ] Generate MCP tool definitions from the same operation registry.
- [ ] Reuse exactly the same provider and schema validation as the CLI bridge.
- [ ] Keep MCP dependencies optional and outside the base Skill requirements.
- [ ] Add CLI/MCP equivalence tests for requests, findings, evidence, and errors.
- [ ] Test client disconnect, cancellation, malformed input, and unavailable
  workspace access.

Exit criteria: supported read-only operations produce semantically equivalent
results through CLI and MCP, and removing MCP dependencies does not break the
base Skill or CLI.

### Phase 5: Add harness integration helpers

- [ ] Document setup for Codex, Claude Code, Gemini CLI, OpenCode, and other
  explicitly supported harnesses.
- [ ] Generate configuration only when requested and support `--dry-run`.
- [ ] Never automatically change sandbox, approval, credential, allowlist, or
  network settings.
- [ ] Add `doctor` diagnostics that distinguish missing installation, missing
  dependency, denied execution, and inaccessible workspace where observable.
- [ ] Record limitations that cannot be observed from inside the Adapter.

Exit criteria: each supported harness has a reproducible setup and teardown
path without automatic permission expansion.

### Phase 6: Prove reuse with a second Skill

- [ ] Select a Skill with operations materially different from `spec-dev`.
- [ ] Implement its provider without modifying the core protocol for
  domain-specific convenience.
- [ ] Measure duplicated code, required escape hatches, and missing lifecycle
  hooks.
- [ ] Revise the runtime only for requirements demonstrated by both Skills or a
  clear security/correctness need.
- [ ] Freeze the Adapter `v1` contract after this evaluation.

Exit criteria: two Skills share the runtime and bridges while keeping their
domain logic independent.

### Phase 7: Cross-harness conformance evaluation

- [ ] Define a machine-readable harness/capability matrix.
- [ ] Test shell available/unavailable.
- [ ] Test MCP available/unavailable.
- [ ] Test read-only and writable workspaces.
- [ ] Test harness-denied subprocess and network calls.
- [ ] Test path traversal and symlink escape attempts.
- [ ] Test missing dependencies and incompatible versions.
- [ ] Test cancellation, timeout, and partial transport failure.
- [ ] Verify that the instruction-only fallback reports unavailable evidence.
- [ ] Preserve exact traces, harness versions, Adapter versions, and candidate
  revisions for behavioral evaluations.

Exit criteria: compatibility claims are backed by observed runs rather than
installation metadata or documentation alone.

## Security Invariants

- [ ] The Adapter never grants or simulates harness permissions.
- [ ] The Adapter never retries a denied action through another bridge to evade
  policy.
- [ ] Workspace paths are explicit, canonicalized, and containment-checked.
- [ ] Symlinks cannot escape the authorized workspace boundary.
- [ ] Read-only operations do not create project files, lockfiles, caches, or
  logs inside the target workspace.
- [ ] Secrets are neither accepted nor emitted unless an operation contract
  explicitly requires them.
- [ ] Diagnostics redact credentials, tokens, authorization headers, and
  sensitive environment values.
- [ ] Configuration generators default to preview and require explicit action
  before writing.
- [ ] MCP v1 does not expose mutating operations.
- [ ] Domain findings cannot be confused with transport or authorization errors.

## Compatibility Matrix Template

| Harness | Skill discovery | Shell | MCP | Native tools | Approval owner | Adapter route | Verified version/date |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Codex | TBD | TBD | TBD | TBD | Harness | CLI or MCP | Not verified |
| Claude Code | TBD | TBD | TBD | TBD | Harness | CLI or MCP | Not verified |
| Gemini CLI | TBD | TBD | TBD | TBD | Harness | CLI or MCP | Not verified |
| OpenCode | TBD | TBD | TBD | TBD | Harness | CLI or MCP | Not verified |
| Cursor | TBD | TBD | TBD | TBD | Harness | MCP or fallback | Not verified |

Do not replace `TBD` from product documentation alone. Record the tested
version, operating system, installation method, relevant policy, date, and
observable result.

## Evaluation Scenarios

- A fully capable CLI environment returns normalized results and evidence.
- A no-shell environment uses MCP when explicitly configured.
- An environment with neither CLI nor MCP follows the portable Skill and marks
  deterministic checks as unavailable.
- A harness denial remains a harness denial and is not converted to
  `environment_error`, `pass`, or `findings` by the Adapter.
- A read-only workspace permits read-only operations without hidden writes.
- A malicious relative path and a symlink escape are rejected before provider
  logic accesses the target.
- CLI and MCP produce equivalent domain results for the same operation and
  repository state.
- Missing optional MCP dependencies do not affect the CLI or instruction-only
  fallback.
- A second Skill registers operations without importing `spec-dev` code.

## Mature References To Revisit During Implementation

- Microsoft Playwright CLI and Playwright MCP: compare the split between a
  token-efficient CLI and richer stateful MCP integration.
  - <https://github.com/microsoft/playwright-cli>
  - <https://github.com/microsoft/playwright-mcp>
- GitHub MCP Server: study domain-oriented tools and read-only/lockdown modes.
  - <https://github.com/github/github-mcp-server>
- Vercel `skills` CLI: use as an installation/discovery reference, while keeping
  runtime compatibility as a separate concern.
  - <https://github.com/vercel-labs/skills>
- OpenAI Figma Skill and Figma MCP: study the separation between workflow
  instructions and stable domain tools.
  - <https://github.com/openai/skills/tree/main/skills/.curated/figma>
- OpenAI `migrate-to-codex`: study compatibility classification, dry-run,
  diagnostics, and validation patterns.
  - <https://github.com/openai/skills/tree/main/skills/.curated/migrate-to-codex>
- Docker MCP Gateway: revisit only if multi-server aggregation becomes a
  demonstrated requirement.
  - <https://github.com/docker/mcp-gateway>

These references are architectural inputs, not proof that their current APIs or
behavior match this repository. Revalidate primary documentation and released
versions before implementation.

## Open Decisions

- [ ] Which language and packaging format should own the shared runtime?
- [ ] Should build-time bundling be part of this repository or an external
  release pipeline?
- [ ] Which existing `specctl.py` output is a compatibility contract?
- [ ] What is the second Skill used to validate generality?
- [ ] Which effects must be declared statically versus observed dynamically?
- [ ] How should callers negotiate optional result fields and output limits?
- [ ] What timeout and cancellation guarantees can be made across CLI and MCP?
- [ ] Which harness/version combinations are officially supported versus best
  effort?
- [ ] When, if ever, should mutating domain operations be exposed through MCP?

## Definition Of Done For Adapter V1

- [ ] The protocol, operation, capability, effect, and result schemas are
  versioned and tested.
- [ ] At least two materially different Skills use the shared runtime.
- [ ] `spec-dev` retains an instruction-only fallback.
- [ ] CLI is the default bridge and MCP remains optional.
- [ ] CLI/MCP equivalence tests pass for all shared operations.
- [ ] Path traversal, symlink escape, read-only workspace, missing dependency,
  cancellation, timeout, and malformed-input tests pass.
- [ ] Real harness evaluations cover the declared support matrix.
- [ ] No integration helper expands permissions or installs credentials without
  an explicit user action.
- [ ] Documentation states what the Adapter cannot observe or enforce.
- [ ] Published Skill artifacts remain self-contained and reproducible.
