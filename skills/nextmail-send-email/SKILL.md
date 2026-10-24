---
name: nextmail-send-email
description: Send transactional email from an app, script or CI job with Nextmail, through its SDKs (TypeScript, Python, Ruby, PHP, Go, Java, C#), its REST API or its CLI. Covers API keys, sending from your own domain, retries that never send twice, and signed webhooks for delivery events. Use when adding email sending to code, or when the user mentions Nextmail.
---

# Send email with Nextmail

Nextmail sends from addresses on domains the workspace has verified. Every call below needs an API key, and a `from` address on one of those domains.

## 1. Get an API key

1. In Nextmail, open Settings, then API keys, and create a key. Sending only is the default scope, and a key can be narrowed to some domains or addresses.
2. The key is shown once. Keep it in the environment as `NEXTMAIL_API_KEY` or in a secret manager. Never commit it, and never ship it to a browser or a mobile app.

## 2. Send

TypeScript, with `npm install @nextmail/sdk`:

```typescript
import { init, nextmail } from '@nextmail/sdk'

init({ apiKey: process.env.NEXTMAIL_API_KEY })

const email = await nextmail.emails.send(
  {
    from: 'Acme Billing <billing@acme.com>',
    to: 'ada@example.com',
    subject: 'Your September invoice',
    html: '<p>Invoice attached.</p>',
  },
  { idempotencyKey: 'invoice-2026-09-ada' },
)

console.log(email.id, email.status)
```

Python, with `pip install nextmail`:

```python
import os

from nextmail import init, nextmail

init(os.environ['NEXTMAIL_API_KEY'])

email = nextmail.emails.send({
    'from': 'Acme Billing <billing@acme.com>',
    'to': 'ada@example.com',
    'subject': 'Your September invoice',
    'html': '<p>Invoice attached.</p>',
})
```

Any language, over HTTP:

```bash
curl -X POST https://api.nextmail.sh/emails \
  -H "Authorization: Bearer $NEXTMAIL_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: invoice-2026-09-ada" \
  -d '{"from":"Acme Billing <billing@acme.com>","to":["ada@example.com"],"subject":"Your September invoice","html":"<p>Invoice attached.</p>"}'
```

From a shell or a CI job, with `npm install -g @nextmail/cli` and `nextmail login`:

```bash
nextmail emails send \
  --from 'Acme Billing <billing@acme.com>' \
  --to ada@example.com \
  --subject 'Your September invoice' \
  --html '<p>Invoice attached.</p>'
```

The same send in Ruby, PHP, Go, Java and C#: https://nextmail.sh/docs/api/quickstart.md

## 3. Retry without sending twice

Give every send an idempotency key that is unique to the message, such as the invoice or order id: the `Idempotency-Key` header over HTTP, or `idempotencyKey` in the SDK. Retrying the same request with the same key returns the message that already exists instead of sending a second one, so a timeout can always be retried safely.

## 4. Send from your own domain

Add the domain in Settings, then Domains (or with the SDK and the API), publish the DNS records it lists, and wait for it to verify. Until then, a `from` on that domain is refused. Steps and records: https://nextmail.sh/docs/get-started/domain.md

## 5. Hear back about delivery

Register an https endpoint in Settings, then Webhooks (or `POST https://api.nextmail.sh/webhooks`), choose the events it wants, such as `email.delivered`, `email.bounced`, `email.failed` and `email.received`, and keep its `whsec_` signing secret as `NEXTMAIL_WEBHOOK_SECRET`. Check the signature on every delivery, with the raw body:

```typescript
import { verifyWebhookSignature } from '@nextmail/sdk'

export async function POST(request: Request) {
  try {
    const event = await verifyWebhookSignature({
      payload: await request.text(),
      headers: request.headers,
      secret: process.env.NEXTMAIL_WEBHOOK_SECRET!,
      toleranceSeconds: 300,
    })
    console.log(event.type, event.data)
  } catch {
    return new Response('bad signature', { status: 400 })
  }
  return new Response(null, { status: 204 })
}
```

Parsing the body and serialising it again changes it, and the signature stops matching. A delivery that fails is tried up to 8 times over about 27 and a half hours.

## Reference

- Quickstart: https://nextmail.sh/docs/api/quickstart.md
- Sending an email, every field: https://nextmail.sh/docs/api/emails/send.md
- Installing the SDKs: https://nextmail.sh/docs/sdk/install.md
- Webhooks and their events: https://nextmail.sh/docs/knowledge/developers/webhooks.md
- Verifying a delivery: https://nextmail.sh/docs/sdk/webhooks/verify.md
- The CLI: https://nextmail.sh/docs/cli/emails.md
