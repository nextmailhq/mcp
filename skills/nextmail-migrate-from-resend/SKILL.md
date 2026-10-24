---
name: nextmail-migrate-from-resend
description: Move an app's email from Resend to Nextmail without rewriting the sending code, and bring audiences, contacts, suppressions, templates and webhooks across. Use when the user wants to switch from Resend, or asks what moving would take.
---

# Move from Resend

## Keep the Resend SDK, change two variables

Nextmail answers the Resend API at `https://api.nextmail.sh/compat/resend`. Point the Resend SDK there and give it an API key from Nextmail:

```bash
RESEND_BASE_URL=https://api.nextmail.sh/compat/resend
RESEND_API_KEY=<an API key from Nextmail>
```

Send, batch, get, list, reschedule and cancel answer in Resend's own shapes and error names. Attachments given as a buffer or a URL, tags, scheduled sends and idempotency keys come across.

Set up the sending domains in Nextmail first: a signing key cannot leave the service that made it, so a domain is added and verified again rather than moved. Steps: https://nextmail.sh/docs/get-started/domain.md

## Bring the data across

In Nextmail, open Migrations, then From a provider, and give it a full-access Resend key. It brings:

- every segment as an audience, and every subscribed contact with their name. Contacts who unsubscribed are counted and left out.
- the suppression list, with each address's reason: bounce, complaint or manual.
- templates, with their variables and fallbacks. A published template arrives published.
- webhook endpoints, with the same URLs, switched off. Deliveries from Nextmail are signed differently, so change the handler to check the new signature before turning each one on.

The Resend key is held encrypted only while the import runs. Running the import twice duplicates nothing. Domains and API keys come back as a checklist instead, and broadcasts, topics and contact properties do not move.

## Later, if you want it

Moving to the native SDK, `@nextmail/sdk`, is optional. It adds what the compatibility endpoint does not cover, such as threads, contacts and webhooks with typed events. The `nextmail-send-email` skill covers it.

## Reference

- Moving from Resend: https://nextmail.sh/docs/knowledge/developers/switch-from-resend.md
- Connecting a domain: https://nextmail.sh/docs/get-started/domain.md
- Verifying webhook deliveries: https://nextmail.sh/docs/sdk/webhooks/verify.md
