# Haechy plugin for Claude Code

[Haechy](https://haechy.com) puts the small apps you build with Claude online for your team: a database,
sign-in for your colleagues, nightly backups, and outside connections that you approve one by one.
Apps run in the US, the EU, the UK or Australia.

## Install

```
/plugin marketplace add dantarakan/haechy-plugin
/plugin install haechy@haechy
```

Then run `/mcp`, choose **haechy** and sign in. New accounts are created in the same window.

Prefer not to use a plugin? Add the connector directly:

```
claude mcp add --transport http --scope user haechy https://mcp.haechy.com/mcp
```

## What this plugin does and sends

- **Adds one remote MCP server**, `https://mcp.haechy.com/mcp` (OAuth 2.1 with PKCE). Nothing else is installed
  or run on your computer: no hooks, no commands, no local server.
- **Includes one skill** (`skills/haechy`) with a technical build reference (`skills/haechy/references/build-guide.md`).
  When you ask Claude to put an app online, the skill has Claude call the Haechy connector's tools to create the app,
  deploy it, check the deploy and share it.
- **Uploads your project folder only when you ask to deploy.** In Claude Code, the connector returns a single-use
  upload URL on a Haechy host, and Claude runs one command from the app's folder that packs it with `tar`
  (leaving out `node_modules`, `.git`, `.env`, `__pycache__` and `.venv`) and sends it with `curl` to that URL.
  Outside Claude Code the files are passed to the connector directly.
- **Outside network access stays blocked** for your app until you approve each host yourself in the Haechy
  console. Claude can ask for access and cannot approve it.
- **No credentials are stored in the plugin.** API keys and passwords for your app are stored by Haechy as
  environment variables (`set_env_vars`) and their values are never shown back.

Privacy: https://haechy.com/legal/privacy · Security: https://haechy.com/security · Terms: https://haechy.com/legal/terms

Docs: https://haechy.com/docs · Support: support@haechy.com

## License

MIT. See [LICENSE](LICENSE).

Haechy is an independent product of DT Global Ventures Ltd. It is not made, sponsored or endorsed by Anthropic.
Claude is a trademark of Anthropic, PBC.
