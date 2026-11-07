---
name: wabi-agents
description: Build, test and deploy an AI support agent on a WhatsApp number or an iMessage/SMS channel with Wabi — instructions, knowledge sources (help center, sitemap, documents), tools, escalation to people, test turns, golden-answer evals, versions promoted from draft to test to production, rollbacks, the session explorer with traces, tokens and cost, and external handlers ("bring your own agent": your own HTTPS endpoint answers each turn). Use for "set up an AI agent to answer our customers", "why did the agent say that", "add our FAQ to the agent", "deploy the agent on our number", "connect our own agent to Wabi"; do not use for the inbox itself (wabi-inbox) or auditing many conversations (wabi-audit).
---

# AI agents

An agent answers customers on a number from its knowledge, cites sources,
calls tools within their permissions and hands off to a person when it is
unsure or a rule says so. Configurations are versioned: a draft is edited,
promoted to **test** (the sandbox number and test chats), then to
**production** (live numbers). Running sessions keep their version.

## Build and ship

1. **Config:** `get_agent_config_schema` for the configuration's schema,
   models and defaults; `create_agent` with a name and config (version 1 is a
   draft). `update_agent` edits the draft.
2. **Knowledge:** `add_knowledge_source` (`kind: "url"` with `follow_links`,
   `"sitemap"`, or `"markdown"`/`"text"` content), `list_knowledge_sources`,
   `sync_knowledge_source`. `search_knowledge` shows exactly what the agent
   would retrieve for a question.
3. **Secrets for tools and BYO model keys:** the person adds these themselves
   at https://getwabi.dev/dashboard/agents?ref=plugin#secrets. Tell them which
   name and kind the configuration expects (model key, HTTP signing secret,
   Slack webhook, MCP server token); never ask for the value or carry it
   yourself. `list_agent_secrets` confirms it is there (names only), and the
   configuration refers to it by name.
4. **Try it:** `test_agent` with `text` (or a conversation) returns the
   decision (respond, approve, escalate), the reply, cited sources, confidence,
   tool calls (side effects simulated) and cost. Golden cases:
   `create_eval_case`, `run_eval_suite` (Growth), `get_eval_run`.
   `replay_conversation` re-runs a real past conversation against the draft.
5. **Ship:** `promote_agent_version` to `test`, `deploy_agent` on the sandbox
   number (`environment: "test"`), `simulate_inbound_message` to talk to it,
   then `promote_agent_version` to `production` and `deploy_agent` on a live
   number (Starter and up). `rollback_agent` restores the previous production
   version; `undeploy_agent` hands the number back to people.
6. **Channels:** `deploy_agent` with `channel_id` (an iMessage or SMS
   connection from `list_channels`; a sandbox connection counts as the
   sandbox) instead of `phone_number_id`. Replies go out on the channel the
   customer wrote on, with no fallback to another channel. Email
   conversations get no agent replies.

## Bring your own agent (external handler)

When the customer already runs an agent, set the version's
`config.handler` to `{type: "external", url: "https://…", signing_secret,
timeout_seconds}`, where `signing_secret` is the name of an HTTP signing secret
the person added in the dashboard (step 3).
Wabi POSTs every customer turn to the URL (signed `X-Wabi-Signature`) and
sends back the `reply` it returns; `handoff.needed` escalates to the inbox,
`end_session` closes the session, `deferred` lets the endpoint answer later
through the API. Wabi keeps sessions, human takeover, approvals, escalation,
the explorer and usage; external turns are free (not AI replies on Wabi's
key). `get_agent_config_schema` has the fields; the request and response
contract is in the docs ("External agents").

## Watch it

- `list_agent_sessions` and `get_agent_session` (every turn: retrieved and
  cited sources, checks, tool calls, tokens, cost, latency, escalation reason).
- `get_agent_metrics`: resolution, containment, escalation, cost per resolved
  issue, latency percentiles, per-tool success rates.
- `list_agent_approvals` / `decide_agent_approval` for replies and tool calls
  held for a person; `suggest_reply` drafts an answer for a person in the inbox.

## Rules

- Deploying to a live number or promoting to production is the human's call.
- AI replies on Wabi's model key count against the plan (`get_usage`);
  turns on the customer's own key (Growth and up) and external handlers do
  not.
