# When creating Pull Requests
1. Use detailed commit messages
2. Every PR Needs a Description that has the following sections
- ## Why - Explaining the problem being solved, and why it is important
- ## Approach taken - Explaining the solution fully and the design
- ## User-facing changes  - Explain how this affects users
- ## Detailed test plan - A Test plan that an agent can follow, including end-to-end verification

The test plan must include exact commands to run and detailed steps to verify the output of the run

# Database provenance

Whenever an agent creates or modifies a `.db` file, or uploads a `.db` file to S3, follow [docs/database-provenance.md](docs/database-provenance.md). Create `<database-stem>_provenance.yaml` beside each database snapshot locally and beside its object in private S3. Record the exact inputs, code commit, commands, validation, hashes, and S3 object identity required by the guide. For an analysis combining databases, create the separate analysis provenance described there. Keep databases, run-specific provenance, and patient data out of Git.
