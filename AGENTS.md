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

Record the GitHub repository URL, exact checked-out branch name, full commit SHA
(not an abbreviated hash), and GitHub commit URL **before the run**. A branch
name can move; the commit SHA identifies the code revision. Use `git rev-parse HEAD`
for the full SHA and `git branch --show-current` for the branch. If HEAD is
detached, record `detached HEAD` instead of inventing a branch. State whether the
commit is available on GitHub (`verified`, `not pushed`, or `not verified`);
do not claim that a constructed commit URL was verified or push code merely to
complete this field. Use a credential-free repository URL.

Also record the checkout's pre-run clean/dirty state. A SHA alone does not
describe uncommitted changes. If relevant code was uncommitted, list the affected
files, including relevant untracked source files, and preserve a reviewed,
secret-free patch or source snapshot in the external run directory. Link it and
record its SHA-256. Capture a new code identity for any attempt that changes code.

**Log every command issued for the run, in execution order**, from setup through
final verification. Include dependency installation, data preparation, Git/code
checks, pipeline stages, analysis, validation, failed commands, retries, and
commands issued by delegated agents. Checks associated with a run belong in its
command log even though standalone routine checks do not need their own record.
Include preparatory commands already executed when initializing the file, then
append command entries as execution proceeds.
Keep planned commands separate from commands actually executed. For each command,
fill in the numbered command-entry fields below, preserving arguments, quoting,
shell redirections, and non-secret environment overrides. Record shell/version,
working directory, start/end times, exit code, and stdout/stderr log links. Link
versioned scripts invoked by those commands. For notebook cells or tool/API
actions without a shell command, record the invocation, cell/script reference,
parameters, and outcome in an entry rather than leaving a gap in the sequence.

Fill in each runtime and settings field explicitly, including effective defaults.
Do not replace the individual fields with a vague description such as
"standard settings". Add rows for workflow-specific settings.

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

- Run ID: <unique run identifier>
- Purpose: <question or task>
- Agent/tool: <name and version if known>
- Status: <running | completed | failed | interrupted>
- Started (UTC): <ISO 8601 timestamp>
- Ended (UTC): <timestamp, or pending>
- Prior run/resumed attempt: <relative provenance link, or not applicable + reason>
- Data root: <PATIENTPUNK_DATA override used, or default sibling PatientPunk_data>
- Run directory: <path relative to data root>

## Code identity (captured before execution)

- GitHub repository URL: <credential-free https://github.com/owner/repository>
- Branch: <exact checked-out branch name, or detached HEAD>
- Full commit SHA: <complete output of git rev-parse HEAD>
- GitHub commit URL: <repository URL>/commit/<full SHA>
- Commit availability on GitHub: <verified | not pushed | not verified; reason>
- Pre-run working tree: <clean | dirty>
- Relevant modified/untracked source files: <paths, or not applicable + reason>
- Uncommitted patch/source snapshot: <relative link and SHA-256, or not applicable + reason>

## Runtime and effective settings

- Operating system: <name/version>
- Runtime: <Python or other runtime name/version>
- Dependency environment: <lockfile path and SHA-256; installed versions/deviations>
- Config: <path/version and SHA-256>
- Schema: <path/version and SHA-256>
- Prompts/templates: <paths/versions and SHA-256 hashes>
- Model provider: <provider identifier>
- Model identifiers: <exact requested and resolved identifiers, where available>
- Sampling settings: <temperature, top_p, token limits, and other effective values>
- Random seeds: <values>
- Data selection: <filters, date windows, sampling method, and limits>
- Execution settings: <workers, batch sizes, retry policy, and budgets>
- Cache behaviour: <enabled/disabled; cache reference and reuse policy>
- Resume behaviour: <checkpoint reference and skipped/reused stages>
- Other non-secret overrides: <names and effective values>
- Planned commands: <ordered plan; actual execution goes in the log below>

## Command log (one numbered entry per executed command/action)

### 1. <stage/purpose>

- Attempt ID: <unique attempt identifier within this run>
- Code identity: <full SHA and patch/snapshot reference for this attempt>
- Shell/tool: <name/version>
- Working directory: <repo-relative or data-root-relative path>
- Started (UTC): <ISO 8601 timestamp>
- Ended (UTC): <timestamp, or pending>
- Exact command/invocation: <complete command, or tool/notebook invocation>
- Non-secret environment overrides: <names and values, or not applicable + reason>
- Script/notebook reference: <path and version/hash; cell ID if applicable>
- Exit code/outcome: <exit code, or tool/cell result; pending while running>
- Stdout log: <relative link, or not applicable + reason>
- Stderr log: <relative link; identify if combined with stdout>
- Deviations/redactions: <what changed from the plan; secret argument names only>

<Repeat numbered entries for every command/action, including failures and retries.>

## Inputs

| Input | Relative link/private URI | Source/snapshot version | SHA-256 or immutable fingerprint | Record count |
| --- | --- | --- | --- | --- |
| <name> | <reference> | <version> | <fingerprint> | <count> |

## Outputs and run links

| Output | Relative link/private URI | SHA-256 where practical | Record count | Complete/partial |
| --- | --- | --- | --- | --- |
| <name> | <reference> | <checksum> | <count> | <state> |

- Manifest/metadata files: <relative links>
- Pipeline/stage run IDs: <stage-to-ID mapping>
- Database references: <relative link/private URI, relevant tables, and run IDs>

## Attempts, validation, and limitations

- Attempt history: <timestamped attempts and their final status>
- Validation commands: <command-log entry numbers>
- Validation results: <expected versus observed results; pass/fail>
- Failures/interruption: <cause, affected stage, and partial output references>
- Deviations from plan: <what changed and why>
- Unknowns/reproduction limits: <missing evidence, reasons, and limitations>
```
