---
name: wabi-templates
description: Create WhatsApp message templates in Wabi, translate them into other languages with AI, validate them against Meta's rules, submit them for approval and track status, quality and analytics per language — including the starter library and templates kept as YAML in a repository. Use for "make an order confirmation template", "translate our templates to Spanish", "why was our template rejected", "which templates perform"; do not use for sending messages (wabi-messaging).
---

# Templates

A template is one name and category (marketing, utility, authentication) with
a localization per language; Meta approves each language separately. Wabi
validates before Meta sees it, drafts translations with AI (protected
variables, a brand glossary) and keeps every version's history.

## Workflow

1. **Start from what exists.** `list_templates` (filter by status or
   category); `import_templates` brings templates already on a connected WABA
   under management; `list_template_library` and `install_template_library`
   add ready-made ones (OTP, booking confirmation, reminders, receipts…).
2. **Create:** `create_template` with a Meta-style name
   (`booking_confirmation`), category, default locale and content using
   `{{variables}}` with sample values. It returns validation issues right away.
   `list_template_languages` gives the locale codes Meta accepts.
3. **Translate:** `translate_template` with `locales: ["es", "pt_BR"]`. Every
   AI draft has a confidence and a reviewer note and must be approved with
   `review_template_localization` (`decision: "approve"`) before submission.
   Edit a locale with `save_template_localization`.
4. **Validate and submit:** `validate_template` (errors block, warnings are
   common rejection reasons), then `submit_template` (or `submit_templates`
   for many). Meta decides in minutes to a day; the sandbox in about a minute.
   `sync_templates` reconciles now; webhooks update statuses as they happen.
5. **Send the right one:** `resolve_template` with a locale or a contact
   returns the approved localization to send (with fallbacks) and Meta's name
   and language for `send_message`.
6. **Watch:** `list_template_alerts` (rejections, pauses, quality drops,
   category changes); `get_template_analytics` per template and locale
   (sent, delivered, read, clicks, replies, opt-outs, cost), first-party next to
   Meta's numbers with discrepancies labeled.

Templates as code: `export_templates` writes a YAML file to commit;
`apply_templates` with `dry_run: true` shows the plan, then applies it
(idempotent, never deletes).

## Rules

- Never submit on your own initiative when the human asked only for a draft.
- Choose the category honestly: Meta re-files mislabelled templates, and
  marketing costs more than utility (`get_meta_rates`).
