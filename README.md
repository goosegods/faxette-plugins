# Faxette plugins

Send a fax from Claude Code or Codex. The plugin adds Faxette's two MCP servers, one for each copy of Faxette, and a `send-a-fax` skill: your agent finds the official fax number, uploads your files and gives you a link to Faxette's status page, where you check the fax and pay to send it. You pay per fax, with no account or subscription.

Faxette faxes to the US, Canada, the EU, the UK, Switzerland, Norway, Iceland, Liechtenstein, Monaco, Montenegro, Ukraine and the Isle of Man.

## Install

Claude Code:

```sh
claude plugin marketplace add goosegods/faxette-plugins
claude plugin install faxette@faxette
```

Codex:

```sh
codex plugin marketplace add goosegods/faxette-plugins
```

Then open `/plugins` in Codex, switch to the Faxette marketplace and install Faxette. Start a new session to use it.

Then ask, for example: "Please fax referral.pdf to Dr. Lee's office."

## Only the connectors

Claude, ChatGPT and other MCP clients can connect directly, without the skill. Each connector faxes to its own copy's countries:

| Connector | Faxes to |
| --- | --- |
| `https://mcp.faxette.com/mcp` | the US and Canada |
| `https://mcp.eu.faxette.com/mcp` | the EU, the UK, Switzerland, Norway, Iceland, Liechtenstein, Monaco, Montenegro, Ukraine and the Isle of Man |

Setup steps for each app are on [faxette.com/mcp](https://www.faxette.com/mcp).

## What's inside

| Path | What it is |
| --- | --- |
| `plugins/faxette/skills/send-a-fax/SKILL.md` | How the agent finds the number, uploads files and hands over the status page |
| `plugins/faxette/.mcp.json` | The two MCP servers, for Claude Code |
| `plugins/faxette/mcp.json`, `plugin.json` | The same plugin in the Agent Plugins format, for Codex |
| `.claude-plugin/marketplace.json` | The Claude Code marketplace |
| `.agents/plugins/marketplace.json` | The Codex marketplace |
| `registry/us/server.json`, `registry/eu/server.json` | Each server's entry in the [MCP Registry](https://registry.modelcontextprotocol.io) |

## Tools

| Tool | What it does |
| --- | --- |
| `create_document` | Reserves an upload for one file (PDF, JPG, PNG, WEBP or HEIC, up to 20 MB) and returns a URL to PUT it to |
| `create_fax` | Creates the fax and returns its `status_url`, where the person checks it and pays |
| `get_fax_status` | Reads a fax from its `status_url`: status, pages, price, payment and attempts |

Nothing is sent or charged from the chat. The person pays on the status page, which shows the fax number, documents and price first.

## Privacy and support

- Privacy policy: [faxette.com/legal/privacy](https://www.faxette.com/legal/privacy)
- Terms: [faxette.com/legal/terms](https://www.faxette.com/legal/terms)
- Support: [support@faxette.com](mailto:support@faxette.com)

## Publishing

Bump `version` in both `plugin.json` files when the plugin changes.

The servers are listed in the MCP Registry as `com.faxette/faxette` and `com.faxette/faxette-eu`. Bump `version` in both `registry/*/server.json` files with the server's, then publish each. The `com.faxette/*` names are proven with a DNS TXT record on `faxette.com` ([how](https://modelcontextprotocol.io/registry/authentication.md#dns-authentication)):

```sh
mcp-publisher login dns --domain faxette.com --private-key <key>
(cd registry/us && mcp-publisher publish)
(cd registry/eu && mcp-publisher publish)
```

## License

MIT
