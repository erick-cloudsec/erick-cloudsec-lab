# OWASP Nest – Local Setup Notes

Read the contents of https://github.com/OWASP/Nest/blob/main/CONTRIBUTING.md before proceeding.
Reason: it defines the expected dev setup, workflow, and contribution rules.

Step-by-step local setup (from CONTRIBUTING):
1. Install prereqs: Docker and pre-commit. On Windows, install WSL and use the WSL terminal; enable Docker WSL integration.
2. Fork and clone your repo:
   - `git clone https://github.com/<your-account>/<nest-fork>`
3. Create env files:
   - `touch backend/.env && cp backend/.env.example backend/.env`
   - `touch frontend/.env && cp frontend/.env.example frontend/.env`
   - Save both files as UTF-8 without BOM.
4. Configure `backend/.env`:
   - Set `DJANGO_CONFIGURATION=Local`.
   - Add Algolia keys: `DJANGO_ALGOLIA_APPLICATION_ID`, `DJANGO_ALGOLIA_WRITE_API_KEY`.
   - Optional: add `GITHUB_TOKEN` if you plan to sync GitHub data.
5. Run the app from the project root:
   - `make run` (leave it running).
6. In another terminal, load and index data:
   - `make load-data`
   - `make index-data`
7. Verify endpoints:
   - `http://localhost:8000/api/v0/`
   - `http://localhost:8000/graphql/`

Optional: GitHub data sync
1. `make create-superuser`
2. Add `GITHUB_TOKEN` to `backend/.env`
3. `make sync-data`

Optional: NestBot dev (never use OWASP Slack workspace)
1. Set up ngrok and static domain, then `ngrok start NestBot`.
2. Add `DJANGO_SLACK_BOT_TOKEN` and `DJANGO_SLACK_SIGNING_SECRET` in `backend/.env`.
3. Configure Slack app via `backend/apps/slack/MANIFEST.yaml` and reinstall the app.

Quality checks before PRs:
- `make check`
- `make test`
- Follow branching/PR process and Code of Conduct from CONTRIBUTING.
