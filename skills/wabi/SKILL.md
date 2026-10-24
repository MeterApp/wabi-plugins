---
name: wabi
description: Run a WhatsApp Business Account through Wabi from the conversation — orient in the organization, see today's operations snapshot, try the simulated sandbox number (delivered and read receipts, a customer who replies, templates approved in a minute), see what to set up next, read the event log and the audit trail. Use for "set up WhatsApp for us", "try a WhatsApp message", "how are we doing today", "what's next in Wabi", "did the webhook fire"; do not use for connecting the CLI, MCP or an API key (wabi-setup), connecting a number (wabi-connect), sending (wabi-messaging) or plans, usage and costs (wabi-billing).
---

# Wabi: the operating platform for WhatsApp Business

Wabi runs a business's WhatsApp on Meta's Cloud API: managed webhooks, a live
team inbox, contacts, localized templates, AI support agents with traces, and
usage and cost analytics. Every action in its dashboard is an **op** with the
same name everywhere: an MCP tool on the `wabi` server, `POST /api/v1/<op>`
over REST, and `npx @meterapp/wabi <op> --json '{…}'` in a terminal. This skill
uses the MCP tool names; the CLI and REST take the same JSON input.

## The model in one paragraph

An **organization** has members (viewer < member < admin < owner), teams, API
keys and one plan. It connects one or more **WhatsApp Business Accounts**
(WABAs, which stay the customer's own at Meta) with their **phone numbers**,
and every organization also has a **sandbox number**: a simulator that answers
like Meta, so everything can be tried before Meta approval. Messages belong to
**conversations** with **contacts**; templates are approved by Meta per
language; AI agents answer from knowledge sources and hand off to people.
Everything that happens lands in the **event log** (`list_events`), the same
events customer webhooks receive.

## Workflow

1. **Orient** (read-only): `get_organization` for the organization and the
   caller's role, `get_onboarding` for the setup checklist and the next step,
   `get_overview` for today: conversations needing a reply, windows closing,
   delivery, estimated Meta spend and AI containment.
2. **Try the sandbox**: `get_sandbox` returns the sandbox number, its test
   customers and what the simulator does. `simulate_inbound_message` has a
   test customer write in (`{ "text": "Hi, can I book for Friday?" }`).
   `send_message` to a test customer (the approved `hello_world` template
   first, since no window is open) goes sent → delivered → read and the
   customer replies on their own; recipients ending in `0000` fail on purpose.
3. **Follow what happened**: `list_events` (filter `type: "message."`) shows
   the platform events in order; `get_event` shows one with how each internal
   consumer handled it. `list_activity` is who changed what; owners and admins
   read sensitive changes with `list_audit_log`.
4. **Next steps** come from `get_onboarding`: connect a live number
   (wabi-connect), get a template approved, deploy an AI agent, invite a
   teammate.

## Rules

- Never send messages to real customers without the human asking; the sandbox
  is the place to try things.
- Meta's own fees are billed by Meta to the customer's WABA, at cost. Do not
  quote them as Wabi charges.
- Errors are RFC 9457 problems with a `request_id`; if something is wrong with
  Wabi itself, `report_issue` with that request id.
