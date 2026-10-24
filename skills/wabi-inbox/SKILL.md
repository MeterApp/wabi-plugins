---
name: wabi-inbox
description: Work and build a WhatsApp team inbox in Wabi — find conversations that need a reply or whose 24-hour window is closing, read the timeline, reply (free-form or template), assign, escalate, resolve, take over from an AI agent, keep saved replies and SLA settings, and manage contacts, consent, segments and campaigns. Use for "who's waiting for a reply", "answer this customer", "set up our support inbox", "send our promo to opted-in customers"; do not use for sending one-off API messages (wabi-messaging) or AI agent setup (wabi-agents).
---

# The inbox

Conversations track whose turn it is: `customer_waiting`, `acknowledged` and
`escalated` need a reply; automatic messages never count as an answer. Each
contact has a 24-hour service window after their last message.

## Work the queue

1. **What needs attention:** `get_inbox_counts` (needs_reply, unassigned,
   mine, escalated, window_expiring, sla_at_risk), then `list_conversations`
   with that `view`. `get_overview` has the same picture with today's numbers.
2. **Read before answering:** `get_conversation` (timeline, notes, the
   contact's messaging, consent and support states).
3. **Reply:** `reply_to_conversation` with `text` while the window is open,
   a `saved_reply` shortcut with `variables`, or a `template` once it has
   closed. Pass an `idempotency_key`. A person replying takes the conversation
   over from its AI agent.
4. **Route:** `claim_conversation` (to yourself), `assign_conversation`
   (a person, a team, or `strategy: least_loaded`), `escalate_conversation`
   with a reason and summary (AI agents stop answering), `set_human_takeover`,
   `tag_conversation`, `update_conversation` (priority drives the SLA).
5. **Close:** `resolve_conversation`; `set_turn_state answered` when the
   customer was answered outside WhatsApp; `reopen_conversation` if needed.
   `add_conversation_note` for internal notes.

## Set it up

- `save_saved_reply` / `list_saved_replies` for canned answers with
  `{{contact.first_name}}` and your own variables.
- `get_inbox_settings` / `update_inbox_settings` (admin): first-response SLA per
  priority, when `window.expiring` and `conversation.needs_reply` fire.
- Contacts: `upsert_contact` from your CRM, `import_contacts` (CSV, `dry_run`
  first), `record_consent` per purpose with evidence, `set_contact_opt_out`,
  `track_contact_event` for business events segments can use.
- Segments: `get_segment_fields`, `preview_segment`, `create_segment` (Growth),
  `explain_segment_membership` to see why someone is in or out.
- Campaigns: `create_campaign` → `estimate_campaign` (audience, exclusions,
  Meta cost) → `send_campaign` with `confirm: true` only after a person
  approved the estimate; `get_campaign` for progress.

## Rules

- Do not send campaigns or replies to real customers without the human's go-ahead.
- Marketing needs granted consent for its purpose; opt-outs are absolute.
