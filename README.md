# Adsterra plugins for AI agents

The `adsterra` plugin marketplace for Claude Code and Codex.

The `adsterra-advertiser` plugin connects your AI agent to the Adsterra connector: an MCP server that works with
your Adsterra advertiser account through the advertiser API. With it the agent can:

- read statistics, balance, campaigns, country bids, creatives, landings, blacklists, whitelists, custom bids and
  campaign requests;
- change blacklists, whitelists, custom bids and country bids, start and pause campaigns, pause creatives, replace the
  landing domain, create campaign requests and cancel verifications.

The plugin also gives the agent a skill: which tools and statistics breakdowns answer typical questions, what to
show you before a change, and what to tell you when the connector does not connect. The skill tells the agent to
show what will change and to wait for your agreement before every change. Claude Code also asks for permission
before each call that changes data.

## Create a token

The connector accepts only tokens with type MCP.

1. In your Adsterra advertiser account, open Settings → API and click Generate token.
2. Choose Token type: MCP. If there is no Token type field, MCP access is not enabled for your account: contact your
   Adsterra manager.
3. Tick Read-only if the agent should only read your data. Leave it unticked to let the agent make changes.
4. Accept the terms and generate the token. Copy it: you will put it into the AI client.

A token is valid for a year. You can have up to 5 active MCP tokens.

## Claude Code

Install the plugin in Claude Code:

```text
/plugin marketplace add https://github.com/adsterra-network/adsterra-agent-plugins.git
/plugin install adsterra-advertiser@adsterra
```

Then:

1. Enter your MCP token in the plugin options: `/plugin` → Installed → adsterra-advertiser → Configure options. Do
   not change the connector URL: the plugin sends your token to it, and Adsterra never asks you to change it.
2. Turn on updates: `/plugin` → Marketplaces → adsterra → Enable auto-update. Without it Claude Code does not update
   this plugin.
3. Run `/reload-plugins`.

Check: `/mcp` shows adsterra-advertiser as connected. Ask "What is my Adsterra balance?"

To change the token later, open Configure options again and run `/reload-plugins`.

To update the plugin by hand: `/plugin` → Marketplaces → adsterra → Update marketplace, then `/reload-plugins`.

## Codex

Install the plugin:

```bash
codex plugin marketplace add https://github.com/adsterra-network/adsterra-agent-plugins.git
codex plugin add adsterra-advertiser@adsterra
```

Codex reads the token from the `ADSTERRA_API_KEY` environment variable. Set it in the environment Codex starts from:

- macOS and Linux: add `export ADSTERRA_API_KEY=YOUR_MCP_TOKEN` to your shell profile, for example `~/.zshrc`, and
  open a new terminal. Do not commit or share this file: it now has your token;
- Windows: add the `ADSTERRA_API_KEY` user variable in System Properties → Environment Variables and open a new
  terminal.

If you run Codex from an IDE, start the IDE from that terminal so that Codex gets the variable. To change the token,
change the variable, open a new terminal and restart Codex or the IDE.

Check: `/mcp` in a Codex session shows adsterra-advertiser with its tools.

To update the plugin, run `codex plugin marketplace upgrade adsterra`.

## Other clients and connecting without the plugin

Any MCP client that can send a header connects to the connector directly: the address is
`https://mcp.adsterratools.com/advertiser`, the token goes in the `X-API-Key` header. Without the plugin the agent
gets the tools but not the skill. Claude Desktop and claude.ai cannot connect yet: their connectors do not send a
token in a header.

Claude Code, with the token in the `ADSTERRA_API_KEY` environment variable, set as for Codex above. Keep the single
quotes: Claude Code saves the name of the variable, not the token:

```bash
claude mcp add --scope user --transport http adsterra-advertiser https://mcp.adsterratools.com/advertiser --header 'X-API-Key: ${ADSTERRA_API_KEY}'
```

Codex, in `~/.codex/config.toml`, with the token in the `ADSTERRA_API_KEY` environment variable:

```toml
[mcp_servers.adsterra-advertiser]
url = "https://mcp.adsterratools.com/advertiser"
env_http_headers = { "X-API-Key" = "ADSTERRA_API_KEY" }
```

Cursor, in `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "adsterra-advertiser": {
      "url": "https://mcp.adsterratools.com/advertiser",
      "headers": {
        "X-API-Key": "YOUR_MCP_TOKEN"
      }
    }
  }
}
```

VS Code, in `.vscode/mcp.json` of your project or in the user configuration (Command Palette → MCP: Open User
Configuration). VS Code asks for the token on the first start and keeps it:

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "adsterra-token",
      "description": "Adsterra MCP token",
      "password": true
    }
  ],
  "servers": {
    "adsterra-advertiser": {
      "type": "http",
      "url": "https://mcp.adsterratools.com/advertiser",
      "headers": {
        "X-API-Key": "${input:adsterra-token}"
      }
    }
  }
}
```

## The connector does not connect

The connector refuses to connect when something is wrong with the token or the account, and the agent has no Adsterra tools. Check
in your Adsterra account:

1. Settings → API has a token with status Active and type MCP. A read-only token also has the type MCP and is marked
   Read-only.
2. If the Generate token window has no Token type field, contact your Adsterra manager: MCP access is not enabled for
   your account.
3. If there is no such token, or you are not sure that the AI client has this token, create a token with type MCP.
4. Put the token into the AI client: see the section of your client above.

The AI client shows the reason and a Request ID:

- Claude Code: `/mcp` or `claude mcp list`, for example "Failed to connect — -32010: Token not found Request ID: …".
  If adsterra-advertiser is not in the list at all, the token is not set in the plugin options;
- Codex: `/mcp` shows the server as failed. The reason is in the warning at the bottom of the screen, F2 opens it;
- Cursor: Output panel → MCP Logs. In `cursor-agent`, `cursor-agent mcp list` shows only "Connection failed". The
  reason is in `cursor-agent mcp list-tools adsterra-advertiser`;
- VS Code: Command Palette → MCP: List Servers → adsterra-advertiser → Show Output.

| Error | What to do |
|---|---|
| `Invalid token provided`, `Token not found` | the token is not set, mistyped or deleted: copy an active MCP token again or create a new one |
| `Token expired` | create a new MCP token |
| `This token type can't be used with AI clients…` | this is an API token: create a token with type MCP |
| `MCP access is not enabled for your account…` | contact your Adsterra manager |
| `Your account is inactive…` | contact your Adsterra manager |
| `Invalid owner type provided` | this is a publisher token: create a token in your advertiser account |
| a connection error without a Request ID | the request did not reach the connector: check the connector URL and your network |

If nothing helps, send the Request ID to Adsterra support.

## The agent cannot make changes

A token created with Read-only gives the agent only the read tools. For changes, create a token with type MCP
without Read-only and put it into the AI client.

## License

The plugin is licensed under the [Apache License 2.0](LICENSE).
