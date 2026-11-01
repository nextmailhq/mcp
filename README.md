# Nextmail for AI agents

Email for AI agents and for the people who build them, from [Nextmail](https://nextmail.sh). This repository connects your agent to your own Nextmail mailbox, and teaches it the jobs people most often hand to an agent: sending email from code, working through an inbox, catching verification emails in tests, and moving from Resend.

## What is in it

- **One MCP server**, `https://api.nextmail.sh/mcp`, the remote server Nextmail runs. Your client signs in with OAuth in the browser, and on the consent page you choose which permissions and which addresses the agent may use. You can narrow or remove the connection later under Account, Connected apps.
- **Four skills**, in `skills/`: sending email with the SDKs, the API or the CLI; working in a mailbox over MCP; disposable inboxes for sign-up tests; and moving from Resend.

## Install

**Claude Code**

```text
/plugin marketplace add nextmailhq/mcp
/plugin install nextmail@nextmail
```

**Claude, ChatGPT and other chat apps.** Add `https://api.nextmail.sh/mcp` as a custom connector.

**Codex.** Add this to `~/.codex/config.toml`:

```toml
[mcp_servers.nextmail]
url = "https://api.nextmail.sh/mcp"
```

**Gemini CLI**

```bash
gemini extensions install https://github.com/nextmailhq/mcp
```

**Any agent that reads skills**

```bash
npx skills add nextmailhq/mcp
```

**Cursor, VS Code, Windsurf and other MCP clients.** See https://nextmail.sh/docs/mcp/connect.md

**Any other agent.** Ask it to read this one Markdown page, which says how to connect over MCP, the command line or the API, and what to ask you before it sends anything: https://nextmail.sh/agent.md

## No account yet

An agent can create the account for you, and your only part is the six-digit code emailed to you. It starts a sign-up with your address at `POST https://api.nextmail.sh/signups`, you give it the code, and `POST https://api.nextmail.sh/signups/{id}/verify` creates the account and gives the agent an API key for the command line and the API. Neither call needs a key, and `npx @nextmail/cli signup --email <address>` does both steps. To connect over MCP as well, set a password from the welcome email, then sign in as above.

## What it runs, sends and fetches

Nothing in this repository runs on your machine. The skills are written instructions, and the plugin adds one remote MCP server, which your client reaches over HTTPS at `https://api.nextmail.sh/mcp` once you have signed in. Its tools act on your Nextmail workspace, with the permissions you approved, and every tool is marked as reading or as making changes, so your client can ask before anything is sent or changed. The skills link to the documentation at https://nextmail.sh, and to the Nextmail SDK and CLI packages, which are installed only if you choose to.

## Privacy, terms and support

- Privacy policy: https://nextmail.sh/privacy
- Terms: https://nextmail.sh/terms
- Support: support@nextmail.sh
- Documentation: https://nextmail.sh/docs/mcp/overview.md

## License

MIT. See `LICENSE`.
