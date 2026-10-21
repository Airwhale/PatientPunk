# When creating Pull Requests
1. Use detailed commit messages
2. Every PR Needs a Description that has the following sections
- ## Why - Explaining the problem being solved, and why it is important
- ## Approach taken - Explaining the solution fully and the design
- ## User-facing changes  - Explain how this affects users
- ## Detailed test plan - A Test plan that an agent can follow, including end-to-end verification

The test plan must include exact commands to run and detailed steps to verify the output of the run

# Run provenance

**For every data, pipeline, or analysis run an agent starts, create a
`provenance.md` file connected to that specific run.** This includes corpus
building, imports, extraction, canonicalisation, classification, experiments,
evaluations, and notebook/report generation, including failed or interrupted
attempts. Routine inspection, linting, type checking, and unit tests do not need
their own provenance files.

## Location and connection to the run

- Keep `provenance.md` in the run's output directory. Corpora, databases, run
  outputs, and these provenance records belong outside the checkout, under
  `PATIENTPUNK_DATA` when set, otherwise the checkout's sibling
  `PatientPunk_data/` directory. Resolve this from the repository root, without
  hardcoded machine-specific paths.
- Use a separate directory for each new run, for example
  `<data-root>/<study>/runs/<run-id>/provenance.md`. A run that updates a shared
  database still needs its own directory for its provenance and logs.
- Use the pipeline's run ID when available. Otherwise assign a unique ID before
  starting (for example a UTC timestamp plus a random suffix). Record any IDs
  assigned later by pipeline stages or database run records in the same file.
- Link to the actual inputs, outputs, logs, and existing manifest or metadata
  files using paths relative to `provenance.md`, or stable private storage URIs.
  For shared databases, identify the relevant run IDs/tables as well as the file.
  Preserve existing manifest schemas; this Markdown record supplements them.
- In the final handoff, include each run's ID and the path to its
  `provenance.md`, so a reviewer can find the record from the reported result.

## When to write it

Create the file **before execution**, with status `running`, the intended command,
inputs, and settings. Update it after each stage and at completion with the actual
outputs, checks, UTC end time, exit code (when available), and final status:
`completed`, `failed`, or `interrupted`. On an abrupt interruption, reconcile a
stale `running` record at the next opportunity; do not report it as completed.

Give reruns a new run ID and directory, and link to the earlier record. For a
resume, append a timestamped attempt with its command, code/config changes, and
outcome to the existing record. Preserve earlier attempts and their outputs;
do not silently overwrite the history.

## Required content

Use the template below. Replace every placeholder with observed values, or
`not applicable` / `unknown` with a reason. Never invent missing provenance.

Record the exact code commit and whether the checkout was dirty **before the
run**. If relevant code was uncommitted, list the affected files and preserve a
reviewed, secret-free patch in the external run directory, linked from the record.
Include runtime/dependency versions or a lockfile reference, effective settings
(including defaults), config/schema/prompt versions or hashes, and, where used,
model/provider identifiers, sampling settings, seeds, filters, limits, and cache
or resume behaviour.

Identify input snapshots with checksums or immutable source versions and record
output checksums/counts where practical. For mutable database inputs, capture the
pre-run snapshot/version before modifying them. Explain unavailable fingerprints
and any limits to reproduction. Record validation results and known limitations,
not just the command's exit status.

**Never read or copy `.env` files into provenance, dump the environment, or include
credentials, signed URLs, or raw patient text.** Record only non-secret settings;
redact sensitive command arguments and use references to private inputs instead
of copying their contents. Keep paths portable by using repo-relative or
data-root-relative references in commands and relative links in the record.

```markdown
# Run provenance: <run-id>

- Purpose: <question or task>
- Agent/tool: <name and version if known>
- Status: <running | completed | failed | interrupted>
- Started (UTC): <ISO 8601 timestamp>
- Ended (UTC): <timestamp, or pending>
- Code: <commit SHA; pre-run clean/dirty state; relevant patch link>
- Runtime/dependencies: <versions and lockfile reference/hash>
- Working directory: <repo-relative path>

## Reproduction
<Exact commands in execution order, with portable paths and secrets redacted.
List non-secret effective settings, config/schema/prompt references and hashes,
models/providers, seeds, filters, limits, and cache/resume behaviour.>

## Inputs
<Relative links or private URIs; source/snapshot IDs, checksums, and counts.>

## Outputs and run links
<Relative links to outputs, logs, existing manifests/metadata, and any prior run.
Include pipeline/database run IDs and tables; checksums and counts as practical.>

## Attempts, validation, and limitations
<Timestamped stages/attempts, exit codes, checks and results, failures, partial
outputs, deviations from the plan, and any unknowns or reproduction limits.>
```
