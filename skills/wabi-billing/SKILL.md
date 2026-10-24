---
name: wabi-billing
description: Answer what Wabi costs and what an organization uses and spends — plans and prices with no sign-in, usage against the plan's messages and AI replies with a month-end forecast, the overage budget cap, Meta's per-message fees (estimates, reconciled actuals, rates by country, spend budgets and alerts), and a Stripe Checkout link for a human to upgrade. Use for "how much is Wabi", "are we near our limit", "what did WhatsApp cost us this month", "what does a marketing message to Brazil cost", "upgrade us to Growth"; do not use for support metrics (wabi-audit).
---

# Wabi plans, usage, costs and upgrades

Two different bills: **Wabi's plan** (messages and AI replies, overage up to a
budget cap, billed by Wabi through Stripe) and **Meta's per-message fees**
(billed by Meta to the customer's WhatsApp Business Account, shown by Wabi at
cost, never marked up).

## Plan and usage

1. **Prices** (no credential): `get_pricing` returns every plan with its
   monthly and yearly price, included messages and AI replies, limits, overage
   rates and the default budget cap.
2. **Usage:** `get_usage` returns messages and AI replies against the
   allowance (percent and month-end forecast), overage and its forecast against
   the cap, whether sends are stopped, and message volume by day, country and
   failure code. `get_account` adds the next plan with a checkout link.
3. **Upgrade:** `create_checkout_link` with `{ "plan": "growth", "interval":
   "month" }` returns a Stripe Checkout URL for a human; nothing is charged
   until they pay. `subscribe` charges the saved card only with
   `confirm: true` after the human approved the price.
4. **Invoices and card:** `get_billing_link` (Stripe portal, for a human).
   `set_budget_cap` changes the overage cap when the human asks.

## Meta's fees

- `get_costs` for a range: estimates (one per delivered message, from Meta's
  published rate card in force that day: market, category, volume tier; free
  messages at 0) and, separately, Meta's reconciled actuals with discrepancies;
  by category, template, country, account, campaign and agent; cost per
  delivered message; this month's spend and forecast; spend budgets.
- `get_meta_rates` with a recipient's number (`wa_id`) or `country` gives
  Meta's per-message rates by category and volume tier before sending.
  Service replies, and utility templates inside an open 24-hour window, are free.
- Spend budgets: `list_spend_budgets`; `set_spend_budget` (admin, only when the
  human asks) sets a monthly limit for the organization, an account or a
  campaign, with alert emails at thresholds and an optional hard stop for
  marketing and bulk sends; `delete_spend_budget` removes one.

Always label Meta numbers as estimates or Meta actuals, and never present
Meta's fees as Wabi charges.
