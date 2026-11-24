# Pinlyx Agent Kit

<img src="assets/logo-512.png" alt="Pinlyx" width="96" height="96" />

Connect your coding agent to [Pinlyx](https://pinlyx.com): CRM, unified inbox, deals and tasks, social publishing, outbound sequences and Google, Meta and TikTok ad reporting, all through one remote MCP server.

- MCP server: `https://api.pinlyx.com/mcp` (Streamable HTTP)
- Auth: OAuth 2.1 with dynamic client registration, or an API key sent as an `X-Api-Key` header
- One connection sees exactly one workspace: the one the sign-in or API key belongs to

This repository packages that server for four ecosystems:

| Ecosystem | What you get | Where |
| --- | --- | --- |
| Claude Code | Plugin marketplace, plugin with the MCP server and six skills | `.claude-plugin/`, `plugins/pinlyx/` |
| Gemini CLI | Extension with the MCP server and context file | `gemini-extension.json`, `GEMINI.md` |
| Kiro | Power with the MCP server and workflows | `kiro/pinlyx/` |
| skills.sh (any agent that supports skills) | The six skills on their own | `plugins/pinlyx/skills/` |

## Plans

The MCP server is available on every Pinlyx plan. The Free plan is read-only: you can read contacts, conversations, deals, posts, sequences and ad reports. Pro and above can also write (send messages, create and move deals, schedule posts, change ads). `crm_whoami` shows the plan and scopes of a connection.

## Install

### Claude Code

```
/plugin marketplace add Pinlyx/pinlyx-agent-kit
/plugin install pinlyx@pinlyx
```

Run `/mcp` and sign in to Pinlyx when prompted. The skills are invoked as `/pinlyx:inbox-triage`, `/pinlyx:pipeline-review` and so on, and Claude also loads them on its own when a request matches.

To use an API key instead of OAuth, add the server yourself (and leave the plugin's server disabled):

```
claude mcp add --transport http pinlyx-key https://api.pinlyx.com/mcp --header "X-Api-Key: YOUR_KEY"
```

### Gemini CLI

```
gemini extensions install https://github.com/Pinlyx/pinlyx-agent-kit
```

The extension registers the `pinlyx` MCP server and loads `GEMINI.md` as context. Sign in through the browser when Gemini CLI asks. To list it: `gemini extensions list`.

### Kiro

1. Open the Powers panel, choose Add Custom Power, then import from a repository URL.
2. Enter `https://github.com/Pinlyx/pinlyx-agent-kit/tree/main/kiro/pinlyx` (the power lives in the `kiro/pinlyx` directory).
3. Kiro runs the OAuth sign-in on first use. For an API key, add `"headers": { "X-Api-Key": "${PINLYX_API_KEY}" }` to `mcp.json` and approve the variable in Kiro's "Mcp Approved Env Vars" setting.

### skills.sh and other agents

```
npx skills add Pinlyx/pinlyx-agent-kit
```

This installs the six skills (listed in `skills.sh.json`). Connect the MCP server in your agent separately: the skills call Pinlyx tools such as `crm_list_deals`, so they need `https://api.pinlyx.com/mcp` configured.

### Any other MCP client

Point it at `https://api.pinlyx.com/mcp` (Streamable HTTP). If the client cannot do OAuth, send the header `X-Api-Key: YOUR_KEY`.

## Skills

| Skill | What it does | Writes? |
| --- | --- | --- |
| `inbox-triage` | Ranks waiting social DMs, Telegram conversations and email threads, and drafts replies | Sends only after you approve each message |
| `pipeline-review` | Reviews deals by stage, finds stalled or overdue ones, proposes next steps | Stage and task changes only after approval |
| `contact-cleanup` | Audits tags, stages, scores and owners and proposes a change list | Applies only approved changes |
| `social-post-planner` | Builds a content calendar and schedules posts | Schedules only after approval, never publishes now unless asked |
| `ads-weekly-report` | Weekly Google, Meta and TikTok report with comparison, pacing, alerts and CRM attribution | Read-only unless you approve a budget or status change |
| `sequence-status` | Shows running sequences, failures and send jobs | Pause and resume only after approval |

Every skill uses real Pinlyx tool names and asks for your approval before anything that sends a message, publishes a post, spends money or removes something.

## Limit what a connection can do

The server URL accepts query parameters: `https://api.pinlyx.com/mcp?toolsets=ads,contacts&readonly=true` shows only the ads and contacts tools and hides every tool that writes.

## Format notes

Verified against the official documentation: the Claude Code marketplace, plugin, `.mcp.json` and skill formats (code.claude.com), the Gemini CLI extension manifest and release rules (geminicli.com), the Kiro remote MCP configuration (kiro.dev/docs/mcp/configuration), and the skills.sh discovery rules.

Not fully verified: Kiro's documentation recommends the newer `plugin.json` package format for powers and says the original `POWER.md` format keeps working. This kit uses `POWER.md` plus `mcp.json`, written to match Kiro's published example powers. The exact Gemini CLI field names for remote servers (`httpUrl`) follow the Gemini CLI MCP configuration and were not shown on the extension pages.

## Support

- Site: https://pinlyx.com
- Docs: https://docs.pinlyx.com
- Email: support@pinlyx.com

## License

MIT, see [LICENSE](LICENSE).
