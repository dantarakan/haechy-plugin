# Building apps that run on Haechy

Technical reference for the Haechy skill. The connector's `get_build_guide` tool returns the current version of
the same rules; this copy ships with the plugin so the skill works without it.

## Stacks
- **Node.js**: a `package.json` with a `start` script (or `main`, or a top-level `server.js` / `index.js`). A `build`
  script runs first if present. Include `package-lock.json` for repeatable installs.
- **Python 3.12**: `requirements.txt` or `pyproject.toml`. FastAPI and Flask apps are found and started
  automatically; Django through `wsgi.py`; or add a `Procfile` with `web: <command>`.
- **Static files**: `index.html` at the top level, or a Node build that outputs `dist/`, `build/` or `out/`.
- Override detection with `haechy.json`: `{"stack", "build", "start", "port"}`.

## Runtime
- The app listens on `0.0.0.0` and the `PORT` environment variable (8080 by default).
- `DATABASE_URL` points to the app's own Postgres database. Tables are created at startup
  (`CREATE TABLE IF NOT EXISTS`, or migrations on start). The filesystem is read-only except `/tmp`, which is wiped
  on restart. Idle apps on some plans sleep and restart on the next visit, so in-memory state is lost; the database
  is kept and backed up every night.
- Other variables: `HAECHY_APP_URL` (the app's public address), `HAECHY_APP_ID`, `HAECHY_REGION`.
- Each app has a memory and CPU ceiling; small server-rendered pages or a small API with a static front end fit well.

## Sign-in
Haechy signs people in before a request reaches the app, so the app has no login of its own. Every request carries
`X-Haechy-User-Email`, `X-Haechy-User-Name` and `X-Haechy-User-Id` (a stable id per person per app); the browser can
read the same from `GET /_haechy/me`. On public apps the headers are missing for visitors who are not signed in.

## Secrets
API keys and passwords are stored with `set_env_vars` (`secret: true`) and read from the environment. A deploy whose
code or `.env` files contain hard-coded keys is blocked by the security check.

## Outside services
The app reaches the internet only through Haechy's proxy (`HTTP_PROXY` / `HTTPS_PROXY`; Node's built-in `fetch` uses
it because `NODE_USE_ENV_PROXY=1` is set), and only hosts the user has approved. `request_outbound_access` files a
request with the host and a plain reason and returns an approval link; the request stays pending until the user
approves it in the Haechy console. `list_outbound_access` shows what is approved, pending and recently blocked.
Requests to other hosts fail with "Blocked by Haechy". Outbound SMTP is not available.

## Email
`POST $HAECHY_API/v1/email` with header `Authorization: Bearer $HAECHY_APP_TOKEN` and JSON
`{"to": ["a@example.com"], "subject": "...", "text": "..." | "html": "...", "replyTo"?: "..."}`.

## Deploying
- In Claude Code, `deploy_app` without files returns a one-time upload command. Run it from the app's folder: it
  packs the folder with `tar` (leaving out `node_modules`, `.git`, `.env`, `__pycache__`, `.venv`) and uploads it with
  `curl` to a single-use Haechy upload URL.
- Without a shell, pass every file in `files` (text, or base64 with `encoding: "base64"`). Each deploy replaces the
  previous code entirely, so send the complete app.
- `get_deploy` shows build progress, the build log and the security check.
- Bundle limits: 10,000 files, 200 MB unpacked. Leave out dependencies, virtualenvs, build output and `.git`.

## When something goes wrong
- Build failed: the build log is in `get_deploy`.
- "Couldn't tell how to run this app": add `package.json`, `requirements.txt` or `index.html`, or a `haechy.json`.
- App live but erroring: `get_app_logs`. Usual causes: not listening on `PORT`, a missing table, a missing variable.
- "Blocked by Haechy" in logs: the host needs an approved outbound request.
- Data mistakes: `list_backups` and `restore_backup` (a restore replaces the current data), or `create_backup` before
  risky changes. `query_database` reads; `execute_database_sql` changes data.
