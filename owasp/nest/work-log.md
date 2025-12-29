# OWASP Nest – Work Log

## 2025-12-28 – OWASP Nest: local environment + Projects API documentation PR

### What I shipped (in plain language)

I improved OWASP Nest API documentation by adding an initial doc page for the Projects list endpoint (`GET /api/v0/projects/`), using verified behavior from a working local environment (with synced data) to make examples realistic and readable.

- Umbrella issue: https://github.com/OWASP/Nest/issues/3062
- PR: https://github.com/OWASP/Nest/pull/3067
- Doc file: `docs/api/projects.md`

### Why it matters (AppSec + architecture lens)

This is “real-world API documentation work” in a security-aware OSS codebase:
- documents auth expectations (API key header) and local-dev differences
- documents pagination/validation behaviors and error responses
- provides concrete examples that reduce integration mistakes and support safer consumption of the API

### Key technical learnings (keep)

- Local setup issues can be “environment realness,” not code:
  - Algolia host resolution issue was fixed by updating DNS servers.
- Secrets/config need to be present in the *running container*, not just on the host:
  - verified `GITHUB_TOKEN` inside the backend container after `.env` placement/compose usage was corrected.
- CI/test harness can fail for repo-wide tooling unrelated to docs changes:
  - `make check-test` surfaced a cspell failure in `py-files.txt` (not modified by my docs PR).

### Evidence

- Linked issue + PR in OWASP/Nest showing reviewable docs changes and OSS workflow compliance.

## 2025-11-29

### What I shipped (in plain language)

I chose OWASP Nest as my first OWASP contribution focus and verified I could run the local environment and access the OpenAPI docs in the browser.

### Why it matters (AppSec + architecture lens)

Validating local OpenAPI access ensures I can review API behavior safely and document endpoints based on observed behavior rather than assumptions.

### Key technical learnings (keep)

- Local environment setup was successful, and OpenAPI docs rendered correctly.

### Evidence

- Local verification of the OpenAPI docs UI in the browser.
