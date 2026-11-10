<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/seal-cream.svg">
  <img src="assets/seal-ink.svg" alt="Faxette" width="96" height="96">
</picture>

# Faxette plugins

Send a fax from Claude Code, Codex or GitHub Copilot. The plugin adds Faxette's MCP server and a `send-a-fax` skill: your agent finds the official fax number, uploads your files and gives you a link to Faxette's status page, where you check the fax and pay to send it. If you're signed in, it can instead send the fax from your Faxette credit once you agree to the price. In an organization, it can also read the faxes your office received, block a sender, and send as the organization on its plan or card. You pay per fax, with no subscription. You don't need an account; if you have one, sign in when your app asks and your faxes stay in it.

Faxette faxes to the US, Canada, Mexico, Puerto Rico, Costa Rica, the EU, the UK, Switzerland, Norway, Iceland, Liechtenstein, Monaco, Montenegro, Ukraine and the Isle of Man.

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

Claude, ChatGPT and other MCP clients can connect directly, without the skill, at `https://mcp.faxette.com/mcp`. It faxes to the US, Canada, Mexico, Puerto Rico, Costa Rica, the EU, the UK, Switzerland, Norway, Iceland, Liechtenstein, Monaco, Montenegro, Ukraine and the Isle of Man.

Setup steps for each app are on [faxette.com/mcp](https://faxette.com/mcp).

## What's inside

| Path | What it is |
| --- | --- |
| `plugins/faxette/skills/send-a-fax/SKILL.md` | How the agent finds the number, uploads files and hands over the status page |
| `plugins/faxette/.mcp.json` | The MCP server, for Claude Code |
| `plugins/faxette/mcp.json`, `plugin.json` | The same plugin in the Agent Plugins format, for Codex and GitHub Copilot |
| `.claude-plugin/marketplace.json` | The Claude Code and GitHub Copilot marketplace |
| `.agents/plugins/marketplace.json` | The Codex marketplace |
| `registry/server.json` | The server's entry in the [MCP Registry](https://registry.modelcontextprotocol.io) |

## Tools

| Tool | What it does |
| --- | --- |
| `get_capabilities` | Each copy's countries, languages, file limits and prices |
| `lookup_fax_number` | Searches Faxette's checked directory for an office's fax number, with its official source |
| `create_document` | Reserves an upload for one file (PDF, JPG, PNG, WEBP or HEIC, up to 20 MB) and returns a URL to PUT it to |
| `create_fax` | Creates the fax, optionally with text or Markdown printed as pages and a cover page, and returns its `status_url`, where the person checks it and pays |
| `get_fax_status` | Reads a fax from its `status_url`: status, pages, price, payment and attempts |
| `resend_fax` | Resends a failed fax, optionally to a corrected number, with the same payment |
| `get_fax_report` | A delivery report for a finished fax |
| `get_account` | Shows which Faxette account is signed in. Needs sign-in |
| `get_faxes` | Lists the account's faxes, 25 at a time, newest first, with a status filter. Needs sign-in |
| `get_fax` | One of the account's faxes, with its documents' file names, history and the quote to approve. Needs sign-in |
| `open_fax_review` | Shows the exact PDF that will be faxed, in apps that can show interactive screens, or as a review link. Needs sign-in |
| `send_fax` | Sends one of the account's faxes, paid from its credit, once the person approves that exact fax and price. For an organization's fax, pays with its plan or holds the amount on its card. Needs sign-in |
| `get_organizations` | The organizations the person let the app use, with each one's plan and fax numbers. Needs sign-in |
| `get_inbound_faxes` | Lists the faxes an organization received, 25 at a time, newest first, with an unread filter. Needs sign-in |
| `get_inbound_fax` | One received fax, with a link to its PDF for 5 minutes. Marks it read. Needs sign-in |
| `read_inbound_fax` | A received fax's pages as images, up to 10, to read or summarize it. Marks it read. Needs sign-in |
| `get_blocked_senders` | Lists the numbers an organization blocked, 25 at a time. Needs sign-in |
| `block_sender` | Blocks a number for the organization: its future faxes are deleted as they arrive, and nobody is emailed. Needs sign-in |
| `unblock_sender` | Lets a blocked number's faxes reach the organization again. Needs sign-in |

Signing in is optional. The connector asks your app to sign you in the first time a tool needs it, through `www.faxette.com`, which sends European accounts on to `eu.faxette.com`. When you connect, you choose which of your organizations the app can use; reconnect to change that. Disconnect an app any time in Settings → Connected apps.

Nothing is sent or charged without the person's agreement. Guests pay on the status page, which shows the fax number, documents and price first. Signed in, the agent can send a fax from the person's credit: it shows them the exact PDF and price and waits for their yes before calling `send_fax`, and Faxette refuses the send if the fax or its price changed since.

## Privacy and support

- Files you send go straight from your computer to Faxette's file storage: `create_document` returns a short-lived upload link and the skill uploads the file to it. Everything else goes through Faxette's MCP servers.
- Privacy policy: [faxette.com/legal/privacy](https://faxette.com/legal/privacy)
- Terms: [faxette.com/legal/terms](https://faxette.com/legal/terms)
- Support: [support@faxette.com](mailto:support@faxette.com)

## Publishing

Bump `version` in both `plugin.json` files when the plugin changes.

The server is listed in the MCP Registry as `com.faxette/faxette`. Bump `version` in `registry/server.json` with the server's, then publish it. The `com.faxette/*` names are proven with a DNS TXT record on `faxette.com` ([how](https://modelcontextprotocol.io/registry/authentication.md#dns-authentication)):

```sh
mcp-publisher login dns --domain faxette.com --private-key <key>
(cd registry && mcp-publisher publish)
```

The key is the hex Ed25519 private key for the `v=MCPv1` record. To replace it, make a new key and swap the `v=MCPv1` TXT record on `faxette.com` for the new public key:

```sh
openssl genpkey -algorithm Ed25519 -out key.pem
echo "v=MCPv1; k=ed25519; p=$(openssl pkey -in key.pem -pubout -outform DER | tail -c 32 | base64)"  # the TXT record
openssl pkey -in key.pem -outform DER | tail -c 32 | xxd -p -c 64                                  # the hex key for login
```

## License

MIT
