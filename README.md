# SalesCloser MCP Server

Connect your AI agent to SalesCloser — the AI calling platform where AI phone
agents make and receive sales calls, book demos, and log every conversation.

This is a **remote MCP server**: nothing to install locally. Point any MCP
client at the endpoint URL, sign in (or use an API key for headless setups),
and your agent can work with your SalesCloser account.

## What your agent can do

On `/mcp` your agent can both read and act on your SalesCloser account:

- List calls with status, type, agent, lead and campaign
- List and inspect leads, agents and knowledge bases, read campaign stats, and summarize call and campaign activity
- Create and update leads, agents, campaigns and knowledge bases
- Place calls and launch outbound campaigns — spending actions run through a confirm-token step the tool descriptions spell out
- Read your account's metadata keys

That is 33 tools. Reads run immediately. Archiving an agent, deleting a knowledge base file,
updating a campaign, creating a call and calling a lead right away never act on the
first call — the first call returns a preview and a single-use
`confirm_token`, and the action runs when the tool is called again with that
token.

## Endpoint

```
https://app.salescloser.ai/mcp
```

Transport: Streamable HTTP (MCP protocol version 2025-06-18).

## Authentication

Two ways to authenticate:

1. **OAuth sign-in (recommended)** — the client opens a browser, you sign in
   to SalesCloser and approve the connection; no token to paste. The endpoint
   advertises protected-resource metadata (RFC 9728) and authorization-server
   metadata (RFC 8414) with dynamic client registration (RFC 7591) and PKCE,
   so any MCP client that supports OAuth connects with no manual token
   handling.
2. **API key (headless clients)** — create a per-user API token in SalesCloser
   (Integrations → API Tokens → Connect AI Assistant), then send it as a Bearer
   token:

   ```
   Authorization: Bearer <YOUR_SALESCLOSER_API_KEY>
   ```

## Install in your MCP client

- **Cline / any MCP client with remote-server support:** see
  [llms-install.md](llms-install.md) — a step-by-step written for AI agents.
- **Claude Code:** `claude mcp add --transport http salescloser https://app.salescloser.ai/mcp` and follow the browser sign-in prompt (or add `--header "Authorization: Bearer YOUR_SALESCLOSER_API_KEY"` for headless use).
- **Cursor / Windsurf:** add a remote MCP server with the URL above; if the UI asks for a transport, choose Streamable HTTP (`streamableHttp` in Cline).

## No public repo?

SalesCloser is a commercial product; this repository is the minimal public
home the MCP directories require. The full product is at
[salescloser.ai](https://salescloser.ai). Questions: use the issue tracker on
this repository or the contact in the marketplace listing.
