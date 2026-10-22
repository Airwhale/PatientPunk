# Database provenance and S3 publication

For every PatientPunk/Lived Evidence `.db` created, changed, rerun, or uploaded, place `<stem>_provenance.yaml` beside it locally and in private S3. Never overwrite a database snapshot or its YAML. Multi-database reports also need `<analysis-name>_provenance.yaml` with all inputs and join, filter, deduplication, and denominator rules.

Start with `status: running`; finish as `completed` only after validation and S3 verification. Otherwise record `failed`/`interrupted` and partial outputs. Label backfills `retrospective`; distinguish observed from reconstructed facts. Use UTC ISO 8601 and explained `null` values. Never invent provenance.

## Required evidence

1. **Identity:** filename, purpose, run ID, status, dates, parent database, and record method.
2. **Code:** repository, branch, full commit SHA and verified URL, pre-run clean/dirty state. Preserve and hash any relevant uncommitted, secret-free code outside Git.
3. **Inputs:** exact snapshots and manifests; local/private-S3 paths, S3 version IDs, bytes, SHA-256, source windows, counts, and acquisition commands. Upload missing required inputs when authorized, verify them, or explain the reproduction limit.
4. **Commands:** one ordered list of the exact commands actually run, sufficient to rebuild this `.db` from the listed inputs and code. Include directory, shell/runtime, non-secret overrides, times, exit codes, logs, retries, resumes, and script versions; cover acquisition through validation and upload. If exact replay is impossible, say why.
5. **Methods:** dependency/schema/config/prompt hashes; exact model and effective settings; seeds, concurrency, selection, exclusions, analysis unit, joins, deduplication, missing values, and dose/route/effect attribution.
6. **Output:** local/S3 paths, version ID, bytes, SHA-256, SQLite integrity, tables/counts, extraction run IDs, and validation commands with expected/observed results. Hash the closed database and verify its S3 copy; ETag is not SHA-256.
7. **Limits:** errors, corrections, unresolved records, data gaps, eligibility, reproduction limits, and whether effects link to treatment, report, or dose episode.
8. **Catalog:** after verifying both S3 objects, update `s3://patientpunk/INDEX.md`, creating it from the latest verified inventory if absent. Include original and optional catalog-copy URIs, scope label, upload date separate from run date, bytes, SHA-256, and YAML URI. Recalculate totals, timestamp in UTC, upload, and read back the entry. Do not claim completion before verification.

Keep credentials, signed URLs, `.env` contents, patient text, identifiers, databases, and run YAML out of Git. Store data only in private S3. Preserve existing manifests and hand off database and provenance locations.

## YAML shape

Use observed values; explain `null` fields and add workflow-specific fields as needed.

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
commands:  # ordered commands actually run; rerun from the pinned inputs
  - stage: "<stage name>"
    command: "<exact runnable command; redact secrets by argument name>"
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
  index_uri: "s3://patientpunk/INDEX.md"
  entry_verified_utc: null
  database_entry_uri: "<exact database S3 URI>"
limitations: []
related_manifests: []
```

For multi-database work, use `record_kind: analysis_provenance`, identify report outputs, and list each input database's provenance URI and SHA-256. Note outcomes imported from another pipeline or linked only at author/report level.
