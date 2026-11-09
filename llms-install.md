# Install the SalesCloser MCP server (agent instructions)

You are installing a remote MCP server for SalesCloser, an AI calling
platform. This is a remote server: nothing to clone, no build step. You add
one entry to the MCP client's configuration, then authenticate — either
through the browser sign-in the client opens, or with an API key if the
client cannot do OAuth.

## What you need from the user

- Nothing if the client supports OAuth: the sign-in flow opens in the user's
  browser and they approve the connection.
- For headless clients, their SalesCloser **API key**. Ask them for it if you
  do not have it: "Please paste your SalesCloser API token (Integrations → API
  Tokens → Connect AI Assistant in the SalesCloser dashboard)."
- Do not invent a key. Do not proceed without a credential when the client
  cannot run the OAuth flow.

## Step 1 — Add the server entry

The endpoint is `https://app.salescloser.ai/mcp` (Streamable HTTP transport).

If the user runs Cline outside VS Code, the same JSON lives at
`~/.cline/data/settings/cline_mcp_settings.json` (CLI; path overridable via
`CLINE_MCP_SETTINGS_PATH`), or they can add the server from the Remote
Servers tab without touching JSON at all.

For Cline, edit `cline_mcp_settings.json` (Cline panel → MCP Servers →
Configure MCP Servers) and add this inside the top-level `"mcpServers"`
object:

```json
{
  "mcpServers": {
    "salescloser": {
      "url": "https://app.salescloser.ai/mcp",
      "type": "streamableHttp",
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Then trigger the connection: the client opens the SalesCloser sign-in page in
the browser, the user signs in and approves, and the client completes the
OAuth flow (dynamic client registration + PKCE) on its own.

For clients without OAuth support, add the API-key header instead:

```json
{
  "mcpServers": {
    "salescloser": {
      "url": "https://app.salescloser.ai/mcp",
      "type": "streamableHttp",
      "disabled": false,
      "headers": {
        "Authorization": "Bearer PASTE_API_KEY_HERE"
      },
      "autoApprove": []
    }
  }
}
```

Replace `PASTE_API_KEY_HERE` with the user's key. Keep the `Bearer ` prefix
and the space — the header must read `Bearer <key>`.

The transport type must be exactly `"streamableHttp"` (camelCase, no hyphen).
`"streamable-http"` or a missing `"type"` makes Cline fall back to SSE and
fail with an HTTP 405.

For clients that name the transport differently:

- Claude Code / generic "http": transport type `http`
- Cursor / Windsurf remote servers: paste the URL and the header; if the UI
  asks for a transport, choose Streamable HTTP

Windows note: `cline_mcp_settings.json` lives at
`%APPDATA%\Code\User\globalStorage\saoudrizwan.claude-dev\settings\cline_mcp_settings.json`.
Forward slashes in the `url` value are correct on all operating systems — do
not escape them.

## Step 2 — Verify the connection

After saving (and after the sign-in completes, if OAuth), ask the client to
list the server's tools (in Cline: the MCP Servers panel shows a green dot
and a tool count for `salescloser`). Then call the `list_calls` tool with
empty arguments. A working connection returns call records as JSON (an object with a
`summary` string and an `items` array). Do not mark the install done until a tool call
has returned real data.

## Troubleshooting

- **HTTP 401 Unauthorized** — no credential was sent, or the API key is
  missing, wrong, revoked or expired. If the client supports OAuth, trigger
  the sign-in again. If it uses an API key, ask the user to re-copy the token
  from Integrations → API Tokens → Connect AI Assistant, paying attention to leading/trailing spaces,
  and update the `Authorization` header. Only tokens created with Connect
  AI Assistant are accepted, and the key must be sent exactly as generated.
- **HTTP 405 on connect** — the transport type is wrong. Set `"type"` to
  `"streamableHttp"` exactly (camelCase).
- **`list_calls` returns no items** — the account has no calls yet. Say so; do
  not retry.
- **Timeouts / corporate proxy** — the client must reach
  `app.salescloser.ai:443` directly. If the user is behind a corporate proxy,
  the MCP client's process needs `HTTPS_PROXY` set in its environment before
  the editor starts; the MCP client does not read proxy settings from the
  config file.

## Security

- The OAuth sign-in grants access to that user's SalesCloser account; an API
  key grants read and write access to that user's SalesCloser data. Never
  echo a key back to the user in plain text after they provide it, never
  write it to a file other than the MCP settings file, and never include it
  in a commit.
- The server can change SalesCloser state. Archiving an agent, deleting a
  knowledge base file, updating a campaign, creating a call and calling a lead right away
  require an explicit confirmation step: the first call returns a
  preview and a `confirm_token`, and the action runs only when the tool is
  called again with that token. Read that preview to the user before
  confirming.
