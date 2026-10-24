---
name: wabi-connect
description: Connect a business's own WhatsApp Business Account and phone number to Wabi on Meta's Cloud API — Embedded Signup or a manual connect with a system user token, registering the number with its PIN, a first test message, health checks and repairs. Use for "connect our WhatsApp number", "why can't we send from our number", "is our WhatsApp account healthy"; do not use for trying things on the sandbox (wabi) or for connecting the CLI or MCP itself (wabi-setup).
---

# Connect a WhatsApp number

The customer's WhatsApp Business Account (WABA), numbers and Meta billing stay
theirs; Wabi subscribes to the account's webhooks, so there is no server to
run. Everything below can be rehearsed on the sandbox number first.

## Workflow

1. **Where are we?** `get_whatsapp_onboarding` returns the current step
   (connect → register a number → send a test → done), the accounts and numbers
   involved, and a hint naming the op for the next step.
2. **How to connect.** `get_connect_config` says whether Meta's Embedded
   Signup is available and lists the manual steps and the token's permissions.
   - **Embedded Signup** happens in the human's browser (Settings → WhatsApp
     accounts → Connect). Send them there; do not try to drive Facebook login.
   - **Manual connect**, when the human gives you a WABA id and a system user
     access token with `whatsapp_business_management` and
     `whatsapp_business_messaging`: `connect_whatsapp_account` with
     `{ "method": "manual", "waba_id": "…", "access_token": "…" }`. The token
     is stored encrypted and never returned. Never echo it back.
3. **Register the number.** Numbers come back `pending` until
   `register_phone_number` with `{ "phone_number_id": "<our id>", "pin": "123456" }`.
   For a number with no two-step PIN yet, the PIN given becomes its PIN: tell
   the human to keep it.
4. **Test send.** `send_message` from the new number to the human's own
   phone (a template such as `hello_world`, since no window is open), then
   `get_message` for the delivery timeline.
5. **Health.** `check_account_health` checks token and permissions, webhook
   subscription, Meta review and business verification, and each number's
   registration, quality rating and messaging limit. Every check says who
   controls it (`meta`, `setup`, `platform`) and the fix; `repair: true`
   re-subscribes missing webhooks (admin). `sync_whatsapp_account` re-reads
   the account after changes in Meta Business Suite.

`list_whatsapp_accounts` and `get_whatsapp_account` show what is connected;
a number's `id` is the `phone_number_id` every send uses.
`disconnect_whatsapp_account` stops using an account (history kept, nothing
changes at Meta): only when the human asks.

## Rules

- Plan limits apply to live numbers and accounts (Free: 1 number); a 402 says
  which plan allows more (`get_pricing`, `create_checkout_link`).
- Meta decides account review, business verification, quality and messaging
  limits; say so instead of retrying.
