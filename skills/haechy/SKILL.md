---
name: haechy
description: Use when the user wants to put an app they built with Claude online, share it with their team or colleagues, give it a database, sign-in, email or backups, or host it somewhere safe for work data (US, UK, EU or Australia). Deploys through the Haechy connector.
---

# Hosting an app on Haechy

Haechy runs the user's app in a sandboxed container with its own Postgres database, sign-in handled by
Haechy, nightly backups, and **no outside network access except connections the user has approved**.

## Workflow
1. Call `get_build_guide` once (how to make code run on Haechy), then `get_started`. If the account isn't set up, give the user the link it returns and wait.
2. `create_app` (pick a short slug; default access is private: only the owner and people they add).
3. Make the app Haechy-ready:
   - listen on `process.env.PORT` / `os.environ["PORT"]` (8080);
   - use `DATABASE_URL` (Postgres) for anything that must persist; the filesystem is read-only except `/tmp`;
   - **don't build a login system**: Haechy signs people in and passes `X-Haechy-User-Email`,
     `X-Haechy-User-Name` and `X-Haechy-User-Id` headers on every request (`/_haechy/me` from the browser);
   - never put secrets in code or `.env` files in the bundle: use `set_env_vars` (deploys with hard-coded keys are blocked);
   - to send email, `POST $HAECHY_API/v1/email` with `Authorization: Bearer $HAECHY_APP_TOKEN`.
4. `deploy_app`. In Claude Code prefer the upload URL it returns
   (`tar czf - --exclude=node_modules --exclude=.git . | curl …`); otherwise pass `files`.
5. Poll `get_deploy` until live. Read the security check: fix blocking findings, then redeploy.
6. Outside services (APIs the server calls): call `request_outbound_access` with the host and a plain reason.
   **Only the user can approve**, by opening the link returned. Tell them what it is for. Never claim it is approved
   until `list_outbound_access` says so. If requests fail with "Blocked by Haechy", check `list_outbound_access`.
7. Share: `add_app_members` (emails) or `set_app_access` (team, domain, public). Teammates sign in with a link
   sent to their email; they don't need a Haechy account.
