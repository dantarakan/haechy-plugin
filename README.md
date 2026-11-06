# Haechy plugin for Claude Code

[Haechy](https://haechy.com) puts the small apps you build with Claude online for your team: a database,
sign-in for your colleagues, nightly backups, and outside connections that you approve one by one.
Apps run in the US, the EU, the UK or Australia.

This plugin adds the Haechy connector (`https://mcp.haechy.com/mcp`) and a skill that tells Claude how to
make an app ready for Haechy and deploy it.

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

Docs: https://haechy.com/docs · Security: https://haechy.com/security · Support: support@haechy.com

Haechy is an independent product of DT Global Ventures Ltd. It is not made, sponsored or endorsed by Anthropic.
Claude is a trademark of Anthropic, PBC.
