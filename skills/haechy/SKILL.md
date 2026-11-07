---
name: haechy
description: Use when the user wants to put an app they built with Claude online, share it with their team or colleagues, give it a database, sign-in, email or backups, or host it somewhere safe for work data (US, UK, EU or Australia). Deploys through the Haechy connector.
---

# Hosting an app on Haechy

Haechy runs the user's app in a sandboxed container with its own Postgres database, sign-in handled by
Haechy, nightly backups, and no outside network access except connections the user has approved.
The technical rules for making code run there are in [references/build-guide.md](references/build-guide.md).
The connector's `get_build_guide` tool returns the same rules and is optional reference.

## Workflow
1. `get_started` shows whether the account is ready. If it needs a step, it returns a link for the user.
2. `create_app` with a short slug (the address becomes `<slug>.haechy.app`). New apps are private: only the owner
   and the people they add can open them.
3. Make the app Haechy-ready (see the build guide):
   - listen on `process.env.PORT` / `os.environ["PORT"]` (8080);
   - use `DATABASE_URL` (Postgres) for anything that must persist; the filesystem is read-only except `/tmp`;
   - no login system of its own: Haechy signs people in and passes `X-Haechy-User-Email`,
     `X-Haechy-User-Name` and `X-Haechy-User-Id` headers on every request (`/_haechy/me` from the browser);
   - secrets go in `set_env_vars`, not in code or `.env` files (deploys with hard-coded keys are blocked);
   - email goes through `POST $HAECHY_API/v1/email` with `Authorization: Bearer $HAECHY_APP_TOKEN`.
4. `deploy_app`. In Claude Code it returns a one-time upload command that packs the project folder with `tar`
   (without `node_modules` and `.git`) and uploads it with `curl` to Haechy; run it from the app's folder.
   Elsewhere, pass the files in `files`.
5. `get_deploy` until the deploy is live. Blocking findings in the security check stop the deploy until fixed.
6. Outside services (APIs the app calls): `request_outbound_access` with the host and a plain reason returns an
   approval link for the user. Only the user can approve, in the Haechy console; the request stays pending until
   then, and `list_outbound_access` shows its state. Calls to hosts that are not approved fail with "Blocked by Haechy".
7. Sharing: `add_app_members` (emails) or `set_app_access` (team, domain, public). Colleagues sign in with a link
   sent to their email; they don't need a Haechy account.
8. `delete_app`, `restore_backup` and data-changing `execute_database_sql` delete or overwrite data; the user
   confirms them first.
