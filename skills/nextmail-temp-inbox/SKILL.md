---
name: nextmail-temp-inbox
description: Create a disposable email address that receives mail, then read the messages, verification codes and links that arrive, with no account and no API key. Use for end-to-end tests of sign-up, password reset and email verification flows, or whenever an agent needs a throwaway address.
---

# Disposable inboxes for tests

A disposable inbox is an address on one of the shared domains Nextmail runs. It receives mail and nothing else, needs no account and no key, and its lease lasts 60 minutes by default, extendable an hour at a time up to 24 hours.

Creating one returns its `id`, its `address` and a `token`. The token is the only thing that can read the inbox, so keep it: no other call returns it again.

## In a test

```typescript
import { createTempMail } from '@nextmail/sdk'

const temp = createTempMail()
const inbox = await temp.create({ ttlMinutes: 60 })
const scope = { inboxToken: inbox.token }

await signUpWith(inbox.address)

const waitForMail = async () => {
  for (let attempt = 0; attempt < 30; attempt++) {
    const { items } = await temp.listMessages(inbox.id, scope)
    if (items.length > 0) return items[0]
    await new Promise((done) => setTimeout(done, 2000))
  }
  throw new Error(`Nothing reached ${inbox.address}`)
}

const arrived = await waitForMail()
const { message } = await temp.getMessage(inbox.id, arrived.id, scope)

const code = message.verificationCode
await temp.delete(inbox.id, scope)
```

`message.verificationCode` is the sign-in or verification code when the email carries one, and null otherwise. For a link, read `message.decodedBody`, the HTML of the message.

Over HTTP, with no key at all. The first call creates the inbox, and the second reads it with the `token` the first call returned:

```bash
curl -s -X POST https://api.nextmail.sh/temp-mail/inboxes -H "Content-Type: application/json" -d '{}'

curl -s https://api.nextmail.sh/temp-mail/inboxes/{id}/messages -H "Authorization: Bearer <token from the first call>"
```

## Over MCP

An agent connected to Nextmail has the same calls as tools: `createTempInbox`, `listTempMessages`, `readTempMessage`, `extendTempInbox` and `deleteTempInbox`.

## Limits worth designing around

- One network can create 10 inboxes every 10 minutes. The next answers 429 with `Retry-After`, so share one inbox across the tests of a run.
- Mail to `name+anything@` lands in the same inbox, so one address can sign up several times: `name+test1@`, `name+test2@`.
- An inbox only receives. There is no sending from it.
- Attachments are listed by name, type and size, without their bytes.
- A conversation is deleted a day after its newest message, whether the lease is still running or not.

## Reference

- Disposable inboxes, the whole API: https://nextmail.sh/docs/api/temp-mail.md
- Creating one: https://nextmail.sh/docs/api/temp-mail/create.md
- Listing its messages: https://nextmail.sh/docs/api/temp-mail/messages.md
