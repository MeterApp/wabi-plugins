---
name: wabi-audit
description: Audit a business's WhatsApp conversations, AI agent quality and costs in Wabi — conversation intelligence (recurring unresolved issues and why, documentation gaps, failing tools, recommended fixes), support metrics with definitions (first response, resolution, AI containment, escalation, reopen, CSAT), and Meta cost estimates against actuals with optimization hints. Use for "what are customers complaining about", "how is our AI agent doing", "weekly support review", "why is our WhatsApp bill high"; do not use for answering individual conversations (wabi-inbox).
---

# Audit conversations, agents and costs

## Workflow

1. **Snapshot:** `get_overview` (needs reply, windows closing, today's
   delivery, spend today, AI containment).
2. **Support metrics:** `get_analytics` for a range (`days: 7`), optionally
   by `account_id`, `agent_id` or `language`: first response and resolution
   (p50/p90), needs reply, reopen, AI-contained resolution (provisional for 72
   hours), escalation, answer quality, CSAT, categories, workload. Every number
   comes with its definition in `definitions`; quote them rather than guessing.
3. **Conversation intelligence:** `audit_conversations` with `since_hours`
   (and `agent_id`) runs in the background; `get_conversation_audit` returns the
   report: outcomes, intents, escalation reasons, failing tools, unresolved
   patterns and why they stay unresolved, documentation gaps with the article to
   write, and recommendations. `wait: true` answers directly for small sets.
   `list_conversation_audits` lists past ones.
4. **Drill into examples:** `list_conversations`, `get_conversation`,
   `list_agent_sessions`, `get_agent_session`, `get_agent_metrics`.
5. **Costs:** `get_costs` shows Meta's fees as estimates (one per delivered
   message, from Meta's rate card that day) and, separately, reconciled actuals
   from Meta with discrepancies flagged; by category, template, country,
   campaign and agent, with the month-end forecast. `get_analytics` → `hints`
   lists savings with their numbers: billed templates inside an open window,
   utility outside the window, expensive destinations, duplicates, failure
   patterns, low-read languages. `get_meta_rates` explains a market's rates.

## Report

Lead with what changed and what to do, each backed by the numbers and the
metric's definition. Label estimates as estimates and actuals as Meta's.
Recommendations that change agents, knowledge or spend budgets
(`set_spend_budget`) are for the human to approve.
