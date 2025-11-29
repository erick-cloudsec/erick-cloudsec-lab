# Repository Guidelines

This repository hosts cloud security lab scenarios and supporting code. Keep changes small, documented, and reproducible.

## Project Structure & Module Organization
- `labs/<scenario>/` — self‑contained exercises (readme, runbook, assets).
- `infra/terraform/` — IaC modules and environments (e.g., `envs/dev`, `modules/`).
- `src/` — application/runtime code (e.g., Lambda, containers) by language.
- `scripts/` — developer and CI helper scripts; keep idempotent.
- `tests/` — unit/integration tests mirroring `src/` and `infra/` paths.
- `docs/` — architecture, threat models, and lab guides.

Example: `labs/iam-least-privilege`, `infra/terraform/envs/dev`, `src/python/handlers/`.

## Build, Test, and Development Commands
Prefer Make targets; add them if missing:
- `make init` — install/verify toolchain (terraform, python deps, pre-commit).
- `make fmt` — run formatters/linters across repo.
- `make validate` — `terraform validate` + security linters (e.g., `tfsec`).
- `make test` — run unit/integration tests with coverage.
- `make plan` / `make apply` — plan/apply for IaC (use `-var-file` under `config/`).

If no Makefile: `terraform init && terraform plan -var-file=config/dev.tfvars`, `pytest -q`, `pre-commit run -a`.

## Coding Style & Naming Conventions
- Python: 4‑space indent, `black` + `isort` + `ruff`; files `snake_case.py`.
- Terraform: 2‑space indent, `terraform fmt -recursive`; modules in `snake_case`.
- Shell: POSIX‐sh where possible; lint with `shellcheck`.
- Directories: kebab‑case (e.g., `least-privilege-lab`).
- Add `pre-commit` and run locally before pushing.

## Testing Guidelines
- Python via `pytest`; tests in `tests/` named `test_*.py`; target ≥80% coverage (`pytest --cov`).
- IaC: `terraform validate`, `tflint`, and `tfsec` on `infra/terraform/`.
- Mirror structure: `tests/src/...`, `tests/infra/...` for clarity.

## Commit & Pull Request Guidelines
- Use Conventional Commits: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`.
- PRs must include: clear description, linked issues, test evidence (logs/screens), security impact notes, and any docs updates.
- Ensure `make fmt validate test` pass before requesting review.

## Security & Configuration Tips
- Never commit secrets; commit `*.example` and keep real values in local `.env` or `*.tfvars` excluded by `.gitignore`.
- Prefer remote backends for Terraform state; least‑privilege IAM; enable encryption at rest/in transit.
- Run secret scanners (`gitleaks`/`detect-secrets`) via pre‑commit.

## Agent Interaction Guidelines
- Always show the exact shell commands executed and briefly explain what each one does so the user can follow and learn.
