---
name: wabi-webhooks
description: Receive Wabi's signed events in your own backend — add a webhook endpoint for message, conversation, template, Flow and AI agent events, verify Standard Webhooks signatures (the @meterapp/wabi-sdk helper or a few lines of code), test, read the delivery log, replay failures and rotate secrets. Use for "send WhatsApp events to our server", "our webhook isn't receiving", "verify Wabi webhook signatures"; Meta's own webhooks are managed by Wabi and need nothing from you.
---

# Customer webhooks

Meta's webhooks for the WhatsApp account are Wabi's job (verified, stored,
deduplicated, normalized). Customer webhooks are Wabi's events to your HTTPS
endpoint, signed with the endpoint's secret.

## Workflow

1. **Events:** `list_webhook_event_types` lists every event with a sample
   body, the signature headers and the retry schedule (message.received,
   message.delivered, message.failed, conversation.needs_reply,
   window.expiring, template.approved, flow.submitted,
   agent.session.completed…).
2. **Add an endpoint:** `create_webhook_endpoint` with `url` and optional
   `event_types`. It returns the signing secret (`whsec_…`) **once**: put it
   in the receiver's secrets, never in code or chat logs.
3. **Verify every delivery** over the raw body, before parsing:

   ```ts
   import { verifyWebhook } from "@meterapp/wabi-sdk/webhooks";

   // signingSecret: the endpoint's whsec_… secret, loaded from your secret store
   export async function POST(request: Request) {
     const event = await verifyWebhook(await request.text(), request.headers, signingSecret);
     // dedupe on event.id (webhook-id); answer 2xx fast, work afterwards
     return new Response(null, { status: 204 });
   }
   ```

   Without the SDK: HMAC-SHA256 of `${webhook-id}.${webhook-timestamp}.${body}`
   with the base64 secret after `whsec_`, compared to each `v1,<base64>` in
   `webhook-signature`; reject timestamps older than 5 minutes. Any Standard
   Webhooks library works as is.
4. **Test and debug:** `test_webhook_endpoint` sends a signed `webhook.test`
   now; `list_webhook_deliveries` and `get_webhook_delivery` show every attempt
   (status, latency, error, the exact signed body); `replay_webhook_delivery`
   or `replay_webhook_events` resend; `update_webhook_endpoint` changes the
   URL, filter or disables it.
5. **Rotate:** `rotate_webhook_secret` returns a new secret; during
   `grace_hours` deliveries carry both signatures, so switch the receiver then.

## Rules

- Delivery is at least once and unordered: deduplicate on `webhook-id`, order by
  `occurred_at`.
- Answer within 10 seconds with any 2xx; failures retry with backoff, then dead-letter.
