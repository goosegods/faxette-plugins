<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/seal-cream.svg">
  <img src="assets/seal-ink.svg" alt="Faxette" width="96" height="96">
</picture>

# Faxette plugins

Send a fax from Claude Code, Codex or GitHub Copilot. The plugin adds Faxette's two MCP servers, one for each copy of Faxette, and a `send-a-fax` skill: your agent finds the official fax number, uploads your files and gives you a link to Faxette's status page, where you check the fax and pay to send it. If you're signed in, it can instead send the fax from your Faxette credit once you agree to the price. You pay per fax, with no subscription. You don't need an account; if you have one, sign in when your app asks and your faxes stay in it.

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

GitHub Copilot CLI:

```sh
copilot plugin marketplace add goosegods/faxette-plugins
copilot plugin install faxette@faxette
```

VS Code finds plugins you install with Copilot CLI. Without the CLI, run **Chat: Open Customizations**, add the `goosegods/faxette-plugins` marketplace under **Browse → Configure Marketplaces**, and install Faxette.

Then ask, for example: "Please fax referral.pdf to Dr. Lee's office."

## Only the connectors

Claude, ChatGPT and other MCP clients can connect directly, without the skill. Each connector faxes to its own copy's countries:

| Connector | Faxes to |
| --- | --- |
| `https://mcp.faxette.com/mcp` | the US and Canada |
| `https://mcp.eu.faxette.com/mcp` | the EU, the UK, Switzerland, Norway, Iceland, Liechtenstein, Monaco, Montenegro, Ukraine and the Isle of Man |

Setup steps for each app are on [faxette.com/mcp](https://faxette.com/mcp).

## What's inside

| Path | What it is |
| --- | --- |
| `plugins/faxette/skills/send-a-fax/SKILL.md` | How the agent finds the number, uploads files and hands over the status page |
| `plugins/faxette/.mcp.json` | The two MCP servers, for Claude Code |
| `plugins/faxette/mcp.json`, `plugin.json` | The same plugin in the Agent Plugins format, for Codex and GitHub Copilot |
| `.claude-plugin/marketplace.json` | The Claude Code and GitHub Copilot marketplace |
| `.agents/plugins/marketplace.json` | The Codex marketplace |
| `registry/us/server.json`, `registry/eu/server.json` | Each server's entry in the [MCP Registry](https://registry.modelcontextprotocol.io) |

## Tools

| Tool | What it does |
| --- | --- |
| `create_document` | Reserves an upload for one file (PDF, JPG, PNG, WEBP or HEIC, up to 20 MB) and returns a URL to PUT it to |
| `create_fax` | Creates the fax and returns its `status_url`, where the person checks it and pays |
| `get_fax_status` | Reads a fax from its `status_url`: status, pages, price, payment and attempts |
| `get_account` | Shows which Faxette account is signed in. Needs sign-in |
| `get_faxes` | Lists the account's faxes, 25 at a time, newest first, with a status filter. Needs sign-in |
| `get_fax` | One of the account's faxes, with its documents' file names and history. Needs sign-in |
| `send_fax` | Sends one of the account's faxes, paid from its credit, once the person agrees to the price. Needs sign-in |

Signing in is optional. The connector asks your app to sign you in the first time a tool needs it, through your copy of Faxette (`www.faxette.com` or `eu.faxette.com`). Disconnect an app any time in Settings → Connected apps.

Nothing is sent or charged without the person's agreement. Guests pay on the status page, which shows the fax number, documents and price first. Signed in, the agent can send a fax from the person's credit: it tells them the price and waits for their yes before calling `send_fax`.

## Privacy and support

- Files you send go straight from your computer to Faxette's file storage: `create_document` returns a short-lived upload link and the skill uploads the file to it. Everything else goes through Faxette's MCP servers.
- Privacy policy: [faxette.com/legal/privacy](https://faxette.com/legal/privacy)
- Terms: [faxette.com/legal/terms](https://faxette.com/legal/terms)
- Support: [support@faxette.com](mailto:support@faxette.com)

## Publishing

Bump `version` in both `plugin.json` files when the plugin changes.

The servers are listed in the MCP Registry as `com.faxette/faxette` and `com.faxette/faxette-eu`. Bump `version` in both `registry/*/server.json` files with the server's, then publish each. The `com.faxette/*` names are proven with a DNS TXT record on `faxette.com` ([how](https://modelcontextprotocol.io/registry/authentication.md#dns-authentication)):

```sh
mcp-publisher login dns --domain faxette.com --private-key <key>
(cd registry/us && mcp-publisher publish)
(cd registry/eu && mcp-publisher publish)
```

The key is the hex Ed25519 private key for the `v=MCPv1` record. To replace it, make a new key and swap the `v=MCPv1` TXT record on `faxette.com` for the new public key:

```sh
openssl genpkey -algorithm Ed25519 -out key.pem
echo "v=MCPv1; k=ed25519; p=$(openssl pkey -in key.pem -pubout -outform DER | tail -c 32 | base64)"  # the TXT record
openssl pkey -in key.pem -outform DER | tail -c 32 | xxd -p -c 64                                  # the hex key for login
```

## License

MIT
