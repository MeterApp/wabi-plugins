---
name: wabi-setup
description: Connect a machine, an agent or CI to Wabi — sign the CLI in through the browser, add the hosted MCP server to Claude Code, Codex or Cursor, or create a scoped API key for a backend or a GitHub Actions secret. Use for "connect Wabi", "set up the Wabi MCP server", "give our backend a Wabi key", "Wabi CLI login"; do not use for working inside an organization once connected (wabi) or for plans and billing (wabi-billing).
---

# Connect to Wabi

## The CLI

```bash
npx @meterapp/wabi login
```

opens the browser, the person approves the code, and the CLI saves an API key
for the organization they choose. On a machine without a browser:
`npx @meterapp/wabi login --no-browser` prints the code and the link. In CI,
set `WABI_API_KEY` from a repository secret instead.

## The MCP server

Hosted at `https://getwabi.dev/api/mcp` (streamable HTTP, OAuth sign-in). In
Claude Code:

```bash
claude mcp add --transport http wabi https://getwabi.dev/api/mcp
```

then `/mcp` to sign in. A client that only speaks stdio can use the CLI's
bridge: `npx @meterapp/wabi mcp`. Prices and plans need no sign-in at
`https://getwabi.dev/api/mcp/catalog`.

## API keys

A person creates keys in the dashboard (Settings → API keys) or with
`create_api_key`. A key acts with its creator's role (an owner's key acts as
admin) and can be narrowed with scopes: `messages:read`, `templates:write`,
`messages:*`, `*:read`, and `billing:purchase` (never implied). Keys belong to
the organization and keep working if their creator leaves; admins revoke them
with `revoke_api_key`. The full key (`wabi_…`) is shown once: store it as a
secret, never in client-side code.
