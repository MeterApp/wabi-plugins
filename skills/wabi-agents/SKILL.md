---
name: wabi-agents
description: Build, test and deploy an AI support agent on a WhatsApp number with Wabi — instructions, knowledge sources (help center, sitemap, documents), tools, escalation to people, test turns, golden-answer evals, versions promoted from draft to test to production, rollbacks, and the session explorer with traces, tokens and cost. Use for "set up an AI agent to answer our customers", "why did the agent say that", "add our FAQ to the agent", "deploy the agent on our number"; do not use for the inbox itself (wabi-inbox) or auditing many conversations (wabi-audit).
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
3. **Secrets for tools and BYO model keys:** `set_agent_secret` (value never
   returned), `list_agent_secrets`.
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
  turns on the customer's own key (Growth and up) do not.
