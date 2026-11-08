---
name: wabi-setup
description: Connect a machine, an agent or CI to Wabi — sign the CLI in through the browser, add the hosted MCP server to Claude Code, Codex or Cursor, or create a scoped API key for a backend or a GitHub Actions secret. Use for "connect Wabi", "set up the Wabi MCP server", "give our backend a Wabi key", "Wabi CLI login"; do not use for working inside an organization once connected (wabi) or for plans and billing (wabi-billing).
---

# Connect to Wabi

## The plugin

Install the plugin, then use any tool: your agent opens a Wabi sign-in in the
browser and keeps the connection. In Claude Code, `/mcp` shows the connection
and signs in again if needed. Codex and Cursor sign in the same way from their
connector settings. For scripts, CI and the SDK, create an API key in the
dashboard and keep it as a secret in that environment; the plugin never uses it.

## The CLI

```bash
npx @meterapp/wabi login
```

opens the browser, the person approves the code, and the CLI keeps a key for
the organization they choose. On a machine without a browser:
`npx @meterapp/wabi login --no-browser` prints the code and the link. For CI,
create a key in the dashboard and keep it as a GitHub Actions secret; the
[Wabi docs](https://getwabi.dev/docs/cli?ref=plugin#ci) show the workflow.

## The MCP server

Hosted at `https://getwabi.dev/api/mcp` (streamable HTTP, OAuth sign-in). In
Claude Code:

```bash
claude mcp add --transport http wabi https://getwabi.dev/api/mcp
```

then `/mcp` to sign in. On a machine without a browser, the person adds the
server themselves with their own key, as shown in
https://getwabi.dev/docs/mcp?ref=plugin#claude. A client that only speaks stdio
can use the CLI's bridge: `npx @meterapp/wabi mcp`. Prices and plans need no sign-in at
`https://getwabi.dev/api/mcp/catalog`.

## API keys

A person creates keys in the dashboard
(https://getwabi.dev/dashboard/settings?ref=plugin, API keys). A key acts with its creator's role (an owner's key acts as
admin) and can be narrowed with scopes: `messages:read`, `templates:write`,
`messages:*`, `*:read`, and `billing:purchase` (never implied). Keys belong to
the organization and keep working if their creator leaves; admins revoke them
with `revoke_api_key`. The full key (`wabi_…`) is shown once, to the person
who created it: they keep it as a secret, never in client-side code, and it
does not go in this conversation.
