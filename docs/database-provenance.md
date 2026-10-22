# Database provenance and S3 publication

Use this guide whenever an agent creates, changes, reruns, or uploads a SQLite
`.db` file for PatientPunk or Lived Evidence. Keep patient data and run artifacts
outside Git. This file is the versioned policy, not a provenance record for a
particular database.

## Required artifact

For `example.db`, create `example_provenance.yaml`. Keep it beside the database
locally and upload the same YAML beside the database in its private S3 prefix.
If a database is updated by multiple runs, give each immutable database snapshot
its own provenance file and S3 key. Do not overwrite an earlier snapshot or
silently rewrite its provenance. For a report or notebook combining several
databases, also create `<analysis-name>_provenance.yaml` that lists every input
database and the exact join, filtering, deduplication, and denominator rules.

Start the record before running or uploading with `status: running`. Complete it
after validation and S3 verification with `status: completed`, or record
`failed`/`interrupted` and any partial outputs. A historical backfill must be
labeled `retrospective`; never present reconstructed commands or dates as observed.
Use UTC ISO 8601 timestamps. Use `null` with an explanation for unknown or
inapplicable fields. Never invent provenance.

## Minimum evidence to record

1. **Identity and timing:** database filename, purpose, run ID, status, start
   and completion dates, prior run or parent database, and whether the record is
   contemporaneous or retrospective.
2. **Code identity:** repository URL, branch, full `git rev-parse HEAD` SHA,
   verified GitHub commit URL, and pre-run clean/dirty state. If relevant code
   was uncommitted, preserve a reviewed, secret-free patch or source snapshot
   outside Git and record its SHA-256. A commit alone cannot reproduce a dirty
   run.
3. **Inputs:** each exact source snapshot, local path, private S3 URI, S3 object
   version ID if enabled, byte length, SHA-256, source date/window, record count,
   and acquisition command or an explicit reason it is unavailable. Include
   upstream databases and manifests. Before marking an S3 database publication
   complete, upload any necessary input snapshot absent from S3 to an appropriate
   private prefix when authorized, verify it, and record the immutable URI. If
   an input cannot be uploaded, state the restriction and reproduction limit.
4. **Execution:** ordered planned and actually executed commands, including
   working directory, shell/runtime, non-secret environment overrides, start/end
   time, exit code, logs, retries, resumes, and versioned scripts. Commands must
   be sufficient to reproduce from the identified input snapshots. Distinguish
   source acquisition, extraction, database construction, analysis, and upload.
5. **Settings and transformations:** dependency lockfile/hash, schema version,
   config and prompt file hashes, model provider and exact model ID, sampling and
   reasoning settings, random seeds, workers/batches/retries, selection and
   exclusion criteria, unit of analysis, joins, deduplication, missing-value
   treatment, and dose/route/effect attribution rules. Record effective defaults.
6. **Database output:** local and S3 paths, S3 version ID if available, SHA-256,
   byte length, SQLite integrity result, table names and row counts, relevant
   extraction run IDs, and the exact validation commands with expected and
   observed results. Hash a finalized, closed database snapshot. Verify the S3
   object against that snapshot; an S3 ETag alone is not a SHA-256 checksum.
7. **Interpretation:** known errors, manual corrections, unresolved records,
   eligibility limits, data gaps, unit-of-analysis caveats, and anything that
   prevents exact reproduction. State whether side effects are linked to a
   treatment, report, or particular dose episode.
8. **Database catalog:** after verifying the database and provenance YAML in
   private S3, update the database directory index under
   `s3://patientpunk/database_catalog/`. The existing dated inventory is
   `INDEX_2026-10-21.md`; preserve it as a historical snapshot. Maintain a
   current `INDEX.md` in that prefix, creating it from the latest verified
   inventory if absent. Add or update the entry for the exact database object
   with its original S3 URI, catalog-copy URI if present, accurate treatment or
   source label, upload date (distinct from run date), byte length, SHA-256, and
   provenance YAML URI. Recalculate index totals, record the index update UTC,
   upload the revised index, and read it back to verify the new entry. Do not
   claim catalog completion until this check succeeds.

Do not include credentials, signed URLs, `.env` contents, raw patient text, or
direct user identifiers. Do not upload a database or its input data to a public
bucket. Do not put database files, run-specific YAML, or patient data in Git.
The final handoff must identify the database snapshot and its local and S3
provenance paths. Preserve any existing workflow-specific manifests as well.

## YAML shape

The following is a field guide, not permission to fill unknowns by inference.
Add workflow-specific fields where needed. Replace placeholders with observed
values, or use `null` and explain why they are unavailable.

```yaml
schema_version: 1
record_kind: database_provenance
record_method: contemporaneous  # or retrospective
database:
  filename: example.db
  purpose: "<what this database contains>"
  run_id: "<run identifier>"
  status: running  # completed, failed, or interrupted at close
  started_utc: "<ISO 8601>"
  completed_utc: null
  parent_database: null
code:
  repository: "https://github.com/<owner>/<repo>"
  branch: "<observed branch or detached HEAD>"
  commit_sha: "<full 40-character SHA>"
  commit_url: "<verified GitHub commit URL>"
  commit_verified: false
  working_tree: "<clean or dirty>"
  uncommitted_source_snapshot: null  # path and SHA-256 if relevant
environment:
  os: "<name and version>"
  runtime: "<Python and package-manager versions>"
  dependency_lockfile: "<path and SHA-256>"
  schema: "<path, version, and SHA-256>"
  config_and_prompts: []  # paths, versions, and SHA-256 values
  model: null  # provider, exact model ID, effective settings, or reason N/A
  nonsecret_overrides: []
inputs:
  - name: "<source snapshot>"
    local_path: "<data-root-relative path>"
    s3_uri: "s3://<private-bucket>/<exact-key>"
    s3_version_id: null
    sha256: "<SHA-256>"
    bytes: 0
    record_count: null
    source_window: null
    acquisition_command: null
processing:
  unit_of_analysis: "<dose record, report, author-treatment, etc.>"
  selection_and_exclusions: "<exact rules>"
  joins_and_deduplication: "<exact rules>"
  missing_data_and_attribution: "<exact rules>"
  random_seeds: []
commands:
  planned: []
  executed:
    - stage: "<stage name>"
      command: "<exact command; secrets redacted by argument name>"
      shell: "<shell and version>"
      working_directory: "<repo- or data-root-relative path>"
      started_utc: "<ISO 8601>"
      ended_utc: "<ISO 8601>"
      exit_code: 0
      log_path: "<relative private path>"
output:
  local_path: "<data-root-relative path>/example.db"
  s3_uri: "s3://<private-bucket>/<exact-key>/example.db"
  s3_version_id: null
  uploaded_utc: null
  sha256: "<SHA-256 of finalized local database>"
  bytes: 0
  sqlite_integrity_check: "ok"
  schema_version: "<observed version>"
  table_row_counts: {}
  extraction_run_ids: []
validation:
  commands_and_results: []
  s3_sha256_verified: false
catalog:
  index_uri: "s3://patientpunk/database_catalog/INDEX.md"
  entry_verified_utc: null
  database_entry_uri: "<exact database S3 URI>"
limitations: []
related_manifests: []
```

For a multi-database report, set `record_kind: analysis_provenance`, replace
`database` and `output` with the report identity and outputs, and list every
input database under `inputs` with its own provenance URI and SHA-256. The
analysis file must say if an outcome was imported from a different pipeline or
linked only at author/report level rather than to the same dose episode.

