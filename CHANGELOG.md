# Changelog

## 0.5.2 (2026-11-05)

- Claude Code: the optional `api_key` setting is gone; the Wabi MCP server
  signs in through the browser (`/mcp`), as in Codex and Cursor. On a machine
  without a browser, sign the CLI in or add the server yourself with your own
  key (see `wabi-setup`).
- The Claude manifest drops `privacyPolicyUrl`, `termsOfServiceUrl` and
  `supportUrl`, which Claude Code's plugin schema does not define (the links
  stay in the Codex interface and on the directory listing).

## 0.5.1 (2026-11-04)

- The plugin ships no scripts: the repository validator moved out of the
  plugin into the source repository's tooling, so an install contains only
  skills, manifests and docs. No change to the skills or the MCP servers.

## 0.5.0 (2026-10-27)

- `wabi-agents`: external handlers ("bring your own agent": a version's
  `handler` names the HTTPS endpoint that answers each turn, signed requests,
  handoff, deferred and end-of-session semantics) and deployments on iMessage
  and SMS channels (`deploy_agent` / `undeploy_agent` with `channel_id`).

## 0.4.0 (2026-10-24)

- New skills: `wabi-agents` (build, test, version and deploy an AI support
  agent, session explorer and metrics) and `wabi-audit` (conversation
  intelligence, support metrics and Meta cost review).

## 0.3.0 (2026-10-24)

- New skills: `wabi-inbox` (work and set up the team inbox, contacts, consent,
  segments and campaigns), `wabi-templates` (create, translate, validate,
  submit and analyze templates, templates as code) and `wabi-webhooks`
  (signed customer webhooks, verification, delivery log and replays).

## 0.2.0 (2026-10-24)

- New skills: `wabi-connect` (connect your own WhatsApp Business Account and
  number, health checks) and `wabi-messaging` (send messages, templates, Flows
  and bulk sends with Meta's safeguards, follow delivery, estimate cost).
- `wabi-billing` now covers usage forecasts, Meta's per-message fees (estimates,
  reconciled actuals, rates by country) and spend budgets.

## 0.1.0 (2026-10-23)

- First release: the `wabi`, `wabi-setup` and `wabi-billing` skills and the
  hosted Wabi MCP servers (account and catalog).
