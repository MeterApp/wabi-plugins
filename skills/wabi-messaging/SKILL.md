---
name: wabi-messaging
description: Send WhatsApp messages through Wabi — text, templates, media, interactive buttons and lists, Flows and bulk sends — with Meta's 24-hour window, consent, plan and duplicate safeguards, then follow delivery and read receipts, retry failures and estimate Meta's cost first. Use for "send a WhatsApp to this customer", "send our template to these numbers", "did my message get delivered", "why did this send fail"; do not use for creating templates (wabi-templates), working inbox conversations (wabi-inbox) or connecting a number (wabi-connect).
---

# Send messages

Every send goes through the same engine and safeguards, from the dashboard,
MCP, REST, the CLI or the SDK: number eligibility, opt-out and consent, the
24-hour customer service window, template approval, rate limits (the send
waits), the plan's allowance and budget cap, duplicate detection. A refusal
names the `safeguard` and who controls it (`controlled_by`).

## Workflow

1. **Pick the sender.** `list_whatsapp_accounts` (or `get_sandbox` for the
   sandbox) gives each number's `id`: the `phone_number_id` of every send.
2. **Window or template?** Inside the customer's 24-hour window (they wrote
   in the last 24 hours) any message type is allowed and service replies are
   free. Outside it only an approved template can be sent.
3. **Send one:** `send_message`, e.g.
   `{ "phone_number_id": "…", "to": "15550100001", "type": "text", "text": { "body": "Your table is ready" } }`
   or `{ …, "type": "template", "template": { "name": "hello_world", "language": "en_US" } }`.
   Always pass an `idempotency_key` you can reuse when retrying, so a retry
   never sends twice.
4. **Follow it.** The answer says `submitted` once Meta accepted it; sent →
   delivered → read arrive by webhook. `get_message` shows the timeline,
   Meta's error and its pricing category. `list_messages` filters by
   conversation, contact, number or status; `dead_letter: true` lists sends
   that ran out of retries.
5. **Fix failures.** Read the error (`get_message`), fix the cause (template,
   token, window), then `retry_message`. `cancel_message` stops one that has
   not left yet.
6. **Many recipients:** `send_bulk_messages` queues one message to up to many
   customers with per-recipient variables (Growth plan and up on live numbers;
   free on the sandbox). For a segment, prefer a campaign (wabi-inbox:
   `create_campaign` → `estimate_campaign` → `send_campaign` with
   `confirm: true` after a person approved the estimate).
7. **Flows:** `list_flows` then `send_flow` (inside the window); answers land in
   `list_flow_submissions`.

## Cost

Meta charges per delivered message by market, category and volume tier
(service replies, and utility templates inside an open window, are free).
`get_meta_rates` with a recipient's number gives the rate before you send;
`get_costs` shows what was spent. Meta bills its fees to the customer's WABA
at cost; Wabi never marks them up.

## Rules

- Never message real customers the human did not ask you to message; use the
  sandbox's test recipients to try things (recipients ending in `0000` fail
  on purpose).
- Marketing needs the contact's consent; respect opt-outs (`safeguard: consent`).
