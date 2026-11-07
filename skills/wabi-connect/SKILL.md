---
name: wabi-connect
description: Connect a business's own WhatsApp Business Account and phone number to Wabi on Meta's Cloud API — a connect link the person opens in their browser, registering the number there, a first test message, health checks and repairs. Use for "connect our WhatsApp number", "why can't we send from our number", "is our WhatsApp account healthy"; do not use for trying things on the sandbox (wabi) or for connecting the CLI or MCP itself (wabi-setup).
---

# Connect a WhatsApp number

The customer's WhatsApp Business Account (WABA), numbers and Meta billing stay
theirs; Wabi subscribes to the account's webhooks, so there is no server to
run. Everything below can be rehearsed on the sandbox number first.

## Workflow

1. **Where are we?** `get_whatsapp_onboarding` returns the current step
   (connect → register a number → send a test → done), the accounts and numbers
   involved, and a hint naming the op for the next step. Pass `new_connection: true`
   to add another account, or `account_id` to resume a specific account. The
   dashboard Connect button starts a new connection; Finish setup resumes
   the selected account.
2. **How to connect.** The person connects in their own browser, and nothing
   they enter there passes through you. `create_connect_link` returns a link
   (24 hours, only for the person you act for) to a page that walks them
   through Facebook sign-in (Meta's Embedded Signup) and the number's PIN, then
   tells them to come back to you. Give them the link; do not try to drive
   Facebook login. The same page has a manual option for a business with its
   own Meta system user: they enter the account id and access token on the page.
   `get_connect_config` lists those steps and the permissions needed
   (`whatsapp_business_management`, `whatsapp_business_messaging`), so you can
   explain them. From the dashboard, the same wizard is at
   https://getwabi.dev/dashboard/settings/accounts/connect?ref=plugin. Poll
   `get_whatsapp_onboarding` until the number is registered.
3. **Register the number.** Numbers come back `pending` until the person
   registers them on that page with the number's PIN. For a number with no
   two-step PIN yet, the PIN they choose becomes its PIN and they should keep it.
   To resume, make a new link with `create_connect_link` and `account_id`.
4. **Test send.** `send_message` from the new number to the human's own
   phone (a template such as `hello_world`, since no window is open), then
   `get_message` for the delivery timeline.
5. **Health.** `check_account_health` checks token and permissions, webhook
   subscription, app callback URL and `messages` subscription, recent live
   inbound delivery, Meta review and business verification, and each number's
   registration, quality rating and messaging limit. Every check says who
   controls it (`meta`, `setup`, `platform`) and the fix; `repair: true`
   re-subscribes missing webhooks (admin). `sync_whatsapp_account` re-reads
   the account after changes in Meta Business Suite. A WABA subscription alone
   does not prove inbound delivery: the platform's Meta app must have a verified
   callback, subscribe to `messages`, and be Published. An unpublished app only
   receives dashboard test webhooks, including for admins and testers. Missing
   app configuration is a platform error; `repair` only repairs the WABA
   subscription. Without a real inbound message in the last 24 hours, delivery
   and publication remain unverified (warning), even if sandbox tests work.

`list_whatsapp_accounts` and `get_whatsapp_account` show what is connected;
a number's `id` is the `phone_number_id` every send uses.
`disconnect_whatsapp_account` stops using an account (history kept, nothing
changes at Meta): only when the human asks.

## Rules

- Never ask for an access token or a PIN in chat. If someone pastes one, do not
  repeat it or use it: ask them to replace it at Meta and enter the new one on
  the page.
- Plan limits apply to live numbers and accounts (Free: 1 number); a 402 says
  which plan allows more (`get_pricing`, `create_checkout_link`).
- Meta decides account review, business verification, quality and messaging
  limits; say so instead of retrying.
