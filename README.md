# Sleek MCP server

Describe a mobile app in plain language and get real, editable screens back, rendered as an image right in your chat.

Sleek is an AI mobile app design tool. This is its remote MCP server, which lets Claude, ChatGPT, Codex, Cursor, and any other MCP client design for you: you say "design a plant shop app with onboarding, home, product detail, and cart", and your assistant creates a project, runs the design agent, shows you the screens, and hands you a link where you can open them, edit them, and export them to Figma or code.

There is nothing to install. The server runs on Sleek's infrastructure; you point your client at one URL and sign in.

```text
https://sleek.design/api/mcp
```

- **New to MCP?** It's the open standard that lets an AI client connect to an outside tool. "Remote" means the tool is hosted, so you only need the URL above.
- **Need an account?** Create one free at [sleek.design](https://sleek.design).
- **Want a skill instead of an MCP connection?** See [sleekdotdesign/agent-skills](https://github.com/sleekdotdesign/agent-skills).

This repository holds only the manifests and docs that point at that server: no server code, no scripts, no hooks, no telemetry.

## Quickstart

1. Connect your client using the section for it below, and sign in to Sleek when prompted.
2. Ask for a whole app in one sentence: *"Design a plant shop app in Sleek: onboarding, home with featured plants, plant detail, cart, and profile. Warm, minimal style."*
3. Your assistant shows you the rendered screens and gives you a project link. Open it to edit, or to export to Figma, HTML, or React.

Send the whole app as **one** request. Sleek's design agent plans the screen set together, so one message for five screens produces a coherent app; five separate messages produce five unrelated screens and charge you five times.

## Connect your client

Every client uses the same URL.

**Sign in with OAuth.** This is the way to connect Sleek. Add the URL, your client opens a browser, you approve access, done. Nothing to copy, nothing to store, and you can revoke it from either side. Every client on this page supports it, so follow the steps for yours and you are finished.

**Do not reach for an API key first.** Keys exist only for clients that cannot do OAuth at all. A key is a long-lived secret with the same power over your workspace as a full OAuth grant, it does not expire on its own, and configuring one stops your client from ever offering the browser sign-in. If OAuth is failing, fix that rather than working around it; the [troubleshooting](#troubleshooting) section covers the usual causes. The API-key steps below are folded away on purpose.

### Claude.ai, Claude Desktop, Cowork, and mobile

Custom connectors work on Free (one connector), Pro, Max, Team, and Enterprise plans. Claude connects to Sleek from Anthropic's cloud, not from your device.

On a Pro or Max plan:

1. Open **Customize → Connectors**.
2. Click **+**, then **Add custom connector**.
3. Enter `https://sleek.design/api/mcp` as the remote MCP server URL.
4. Leave **Advanced settings** empty. Claude registers itself with Sleek automatically.
5. Click **Add**, sign in to Sleek, and approve access.

On Team or Enterprise, an Owner adds it once under **Organization settings → Connectors → Add** (hover **Custom**, select **Web**, same URL). Members then open **Customize → Connectors**, find the entry the Owner added, and click **Connect**.

To disconnect, open **Customize → Connectors** and choose **Remove**.

<details>
<summary>Optional: pre-registered OAuth client ID</summary>

If a form insists on a client ID, use the one below and leave the client secret blank. It is a public client, so there is no secret.

```text
7ef50bea-9fe8-47bb-8163-68a0e21395ab
```

</details>

### Claude Code

```bash
claude mcp add --transport http sleek https://sleek.design/api/mcp
claude mcp login sleek
```

You can also sign in from `/mcp` inside Claude Code.

Or install it as a plugin from the marketplace in this repository:

```text
/plugin marketplace add sleekdotdesign/mcp
/plugin install sleek@sleek
```

Then sign in the same way. The plugin's server appears as `plugin:sleek:sleek`. It ships only the server definition in `.mcp.json`: no hooks, skills, agents, or commands.

<details>
<summary>Last resort: API key instead of OAuth</summary>

Put the key in your shell profile first, so the value is never written into Claude Code's config:

```bash
export SLEEK_API_KEY=sk_...   # in ~/.zshrc or ~/.bashrc
claude mcp add --transport http sleek https://sleek.design/api/mcp \
  --header 'Authorization: Bearer ${SLEEK_API_KEY}'
```

The single quotes matter: they store the `${SLEEK_API_KEY}` reference and Claude Code reads it from your environment on each connection. With double quotes your shell substitutes the key and Claude Code saves it in plain text.

</details>

### Codex CLI and ChatGPT desktop

```bash
codex mcp add sleek --url https://sleek.design/api/mcp
codex mcp login sleek
```

The ChatGPT desktop app, the Codex CLI, and the IDE extension share this configuration.

<details>
<summary>Last resort: API key instead of OAuth</summary>

Set `SLEEK_API_KEY` in your environment, then either:

```bash
codex mcp add sleek --url https://sleek.design/api/mcp --bearer-token-env-var SLEEK_API_KEY
```

or add this to `~/.codex/config.toml` to apply it to every project:

```toml
[mcp_servers.sleek]
url = "https://sleek.design/api/mcp"
bearer_token_env_var = "SLEEK_API_KEY"
```

</details>

### ChatGPT

1. Open **Settings → Security and login** and turn on **Developer mode**. Availability depends on your account and workspace policy.
2. Go to the ChatGPT Plugins page and click the plus button.
3. Give it a name (Sleek) and a description.
4. Under **Connection**, enter `https://sleek.design/api/mcp`.
5. Create the connection and sign in to Sleek.

If it asks for an authentication method, choose OAuth and leave the client ID and secret blank.

### Cursor

Paste this into your browser's address bar and Cursor offers to install the server:

```text
cursor://anysphere.cursor-deeplink/mcp/install?name=sleek&config=eyJ1cmwiOiJodHRwczovL3NsZWVrLmRlc2lnbi9hcGkvbWNwIn0=
```

Or add it by hand to `~/.cursor/mcp.json` (all projects) or `.cursor/mcp.json` (one project):

```json
{ "mcpServers": { "sleek": { "url": "https://sleek.design/api/mcp" } } }
```

Cursor runs the OAuth sign-in when you enable the server, which is all you need. Only if that is impossible, add a header; Cursor resolves `${env:NAME}`:

```json
{ "mcpServers": { "sleek": { "url": "https://sleek.design/api/mcp", "headers": { "Authorization": "Bearer ${env:SLEEK_API_KEY}" } } } }
```

### Any other MCP client

Add a remote server with the Streamable HTTP transport. The exact `type` value varies by client (`http`, `streamable-http`, or omitted):

```json
{ "mcpServers": { "sleek": { "type": "streamable-http", "url": "https://sleek.design/api/mcp" } } }
```

Clients that implement MCP authorization discover everything they need from the [protected resource metadata](https://sleek.design/.well-known/oauth-protected-resource/api/mcp) and register themselves dynamically, so this is usually the whole configuration. Only a client with no OAuth support at all needs `"headers": { "Authorization": "Bearer sk_..." }` added to it.

## What you can ask for

**Design a whole app in one message.**

> Design a plant shop app in Sleek: onboarding, home with featured plants, plant detail, cart, and profile. Warm, minimal style.

Your assistant creates a project, sends the whole brief as a single design run, renders the new screens into one image, and shares the project link. One run, charged once.

**Look at what you already have, for free.**

> Show me the cart and profile screens from my plant shop project.

Listing and rendering cost nothing, so browsing your projects never spends credits.

**Edit one screen.**

> On the plant detail screen, move the Add to cart button into a sticky bottom bar and make the price larger.

The agent edits that screen in place instead of creating new ones. This is a design run, so it spends credits.

## Export

The screens are real design files, not pictures of an app. Open the project link your assistant gives you and export from there:

- **Figma.** Click the Figma button and paste into any Figma file. The screens arrive as native, fully editable Figma layers. No plugin needed.
- **Code.** Export screens as HTML or as React with Tailwind CSS.

Both are on every plan, including Free. You own everything you make, designs and code, and you can use it commercially.

Your assistant can also pull a screen's generated HTML straight into the conversation with `get_component`, which is handy if you want it to adapt a screen into your own codebase without you leaving the chat.

## Pricing

Design runs spend your Sleek workspace's AI credits. Listing, reading, and rendering are free.

A new screen costs roughly 30 credits; editing one costs 5 to 30, depending on how much changes.

Free accounts get one-time trial credits, enough for about one design run. Sustained use needs the Pro plan or higher: $69/month, or $30/month billed yearly at $360/year, with 20,000 monthly AI credits, roughly 650 screens. See [sleek.design/pricing](https://sleek.design/pricing) for current plans.

## Tools

The server exposes eleven tools. Your assistant picks them for you; this table is for reference.

| Tool | What it does | Cost |
| :--- | :--- | :--- |
| `list_projects` | List the projects in your workspace, newest first | Free |
| `create_project` | Create an empty project to design into | Free |
| `get_project` | Fetch one project by id | Free |
| `delete_project` | **Permanently** delete a project and every screen in it | Free, cannot be undone |
| `list_components` | List the screens in a project | Free |
| `get_component` | Fetch one screen with its generated HTML, to reuse in your own code | Free |
| `list_references` | List Sleek's curated design references to imitate | Free |
| `design` | Run the design agent to create or edit screens from a brief | Spends credits |
| `get_design_run` | Check the status of a design run | Free |
| `cancel_design_run` | Stop a queued or running design run | Free (credits already spent are not refunded) |
| `screenshot` | Render up to four screens into one image, shown inline | Free |

Details that matter if you're driving the tools directly:

- `design` takes the whole brief as one `message`. Optional: `screenId` to edit an existing screen, `referenceId` from `list_references` to imitate a style, `imageUrls` for visual input, `idempotencyKey` so a retry returns the original run instead of charging for a second one, and `wait` (default true). The wait is capped at 270 seconds; after that the call returns `running` and you poll `get_design_run`. Set `wait` to false to poll from the start if your client times out sooner.
- Every screen has **two ids**. The `screenId` is the frame on the canvas, and it is what `design` takes to edit a screen. The component id, returned as `id` by `list_components`, is what `screenshot` and `get_component` take. A design run returns both for every screen it touches, and `list_components` and `get_component` return `screenId` next to the component `id` (`null` when a component isn't on a canvas). Never pass a component id as `screenId`.
- Only one design run can be active per project. On a conflict, poll `get_design_run` until the current run settles, or call `cancel_design_run`, then retry.
- A failed run can include an `error.url`. That's where you resolve the problem, usually topping up credits or upgrading. Clients should pass it along.
- `delete_project` requires `confirm: true`, and should only run after you've named the project you want deleted.
- `screenshot` renders up to four screens per call. Render more in batches.

## Troubleshooting

**Claude Code shows `sleek` as connected before I signed in.** That's expected. Sleek lets any client read the server's name and tool list without a login, so the connection succeeds and authentication is only checked when a tool runs. If you skip the sign-in, the first tool call is rejected and Claude Code then marks `sleek` as needing authentication. Signing in first avoids the round trip.

**My API key doesn't work and I never get an OAuth prompt.** When you configure an `Authorization` header, Claude Code treats a rejected key as a failed connection rather than a reason to sign in, so it won't fall back to OAuth. Check the key, or remove the header and use OAuth.

**Opening the URL in a browser returns 405.** By design. The endpoint accepts `POST` only; the server is stateless and holds no connection open between calls.

**I already have Sleek connected on claude.ai and in Claude Code.** Claude Code deduplicates entries pointing at the same URL and keeps one active in `/mcp`.

To check a key without any client:

```bash
curl -s -X POST https://sleek.design/api/mcp \
  -H "Authorization: Bearer $SLEEK_API_KEY" \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"list_projects","arguments":{}}}'
```

## Authentication and privacy

The server implements standard MCP authorization with OAuth 2.1: authorization code with PKCE (S256), dynamic client registration, and the scopes `openid email profile offline_access`. Access tokens last one hour and clients refresh them automatically. Sleek's authorization server is its Supabase Auth instance, named in the [protected resource metadata](https://sleek.design/.well-known/oauth-protected-resource/api/mcp).

**What you're granting.** Approving the consent screen lets the client act as you in your Sleek workspace: list, create, and delete projects, design and edit screens, and spend the workspace's credits on design runs. It can't do anything your own account can't do.

**How to revoke.** On the client side, remove the connector (claude.ai: **Customize → Connectors → Remove**; Claude Code: **Clear authentication** in `/mcp`, or `claude mcp logout sleek`, or `claude mcp remove sleek`, which also deletes the stored tokens). On the Sleek side, revoke the grant or the API key from your account at [sleek.design](https://sleek.design), or email support@sleek.design.

**API keys are the fallback, not the default.** A key from [sleek.design/agents/setup](https://sleek.design/agents/setup) is a secret carrying the same access as an OAuth grant, with none of its safeguards: it does not expire, it is not bound to a device, and anyone who reads it has your workspace until you revoke it. Use one only where OAuth cannot run. Keep it in an environment variable, never in a file you commit, and revoke it the moment it leaks.

**Network.** The only destinations are `sleek.design` and its authorization server. This repository ships no code, no hooks, and no telemetry, and the plugin manifest adds nothing beyond the server definition. What Sleek stores and for how long is in the [privacy policy](https://sleek.design/privacy).

## Support

Email support@sleek.design, or read the [docs for agents](https://sleek.design/agents). For security reports see [SECURITY.md](SECURITY.md). Maintainers: see [docs/publishing.md](docs/publishing.md).

Licensed under the MIT License. See [LICENSE](LICENSE).
