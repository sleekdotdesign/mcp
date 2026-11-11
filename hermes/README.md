# Sleek for Hermes Agent

Describe a mobile app in plain language and get real, editable screens back. [Sleek](https://sleek.design) is an AI mobile app design tool, and this plugin connects Hermes Agent to Sleek's hosted MCP server so Hermes can design for you: you say "design a plant shop app with onboarding, home, product detail, and cart", and Hermes creates a project, runs Sleek's design agent, shows you the screens, and hands you a link where you can open them, edit them, and export them to Figma or code.

```text
https://sleek.design/api/mcp
```

The plugin is configuration only: the server definition, Hermes trust and cost settings, and one skill that teaches Hermes the Sleek workflow. There is nothing to run locally.

- **Need an account?** Create one free at [sleek.design](https://sleek.design).
- **Using another client?** Claude, ChatGPT, Codex, and Cursor setup is in the [main README](../README.md).

## Requirements

- Hermes Agent **0.22.3 or later**. Earlier releases do not read the plugin's credit-cost settings.
- A Sleek account. Sign-in is OAuth in your browser.

## Install

```bash
hermes plugins install sleekdotdesign/mcp/hermes
hermes plugins enable sleek
hermes mcp add sleek --url https://sleek.design/api/mcp --auth oauth
hermes config set mcp_servers.sleek.trust untrusted
```

Once Sleek is listed in the Hermes plugin catalog, the first line becomes `hermes plugins install sleek`.

What each step does:

1. `plugins install` adds the plugin. Portable plugins install disabled.
2. `plugins enable` turns it on: the `sleek` server, its cost table, and the `sleek` skill.
3. `mcp add … --auth oauth` tells Hermes to sign in to this server with OAuth. A portable plugin cannot turn on OAuth by itself, so this step is required. Without it, Hermes still lists Sleek's tools (Sleek serves its tool list without a login), but every tool call is rejected with 401.
4. `config set … trust untrusted` keeps Hermes asking before write-capable calls (see [Trust and approvals](#trust-and-approvals)).

## Sign in

Start a new session, or run `/reload-mcp` in an open one, then ask Sleek for something:

> List my Sleek projects.

> Design a plant shop app in Sleek: onboarding, home with featured plants, plant detail, cart, and profile. Warm, minimal style.

The first tool call opens your browser to sign in to Sleek. Sign in, approve access, and Hermes finishes the call. Hermes stores the token and refreshes it for you, so you sign in once.

`hermes mcp login sleek` does not open the browser for Sleek: Sleek lists its tools without a login, so there is nothing for that command to sign in to. Sign-in starts on the first tool call instead.

## Trust and approvals

The plugin marks the server `trust: untrusted`. Hermes asks you before every write-capable tool call (creating a project, running a design, deleting a project), and shows the credit cost on the prompt for `design`. Read-only tools such as `list_projects`, `show_project`, and `screenshot` run without asking. In cron jobs and gateways with nobody to approve, write-capable calls are refused rather than run.

`hermes mcp add` writes a native `mcp_servers.sleek` entry to `config.yaml`, without a `trust` line. A native entry replaces the plugin's trust setting (it keeps the plugin's declared costs), which is why the install sets `trust: untrusted` on it. The entry should read:

```yaml
mcp_servers:
  sleek:
    url: https://sleek.design/api/mcp
    auth: oauth
    trust: untrusted
```

Check with `hermes config get mcp_servers.sleek.trust`. Set `trust: full` only if you want Hermes to design without asking.

## Remote and headless hosts

When the first tool call starts sign-in, Hermes listens for the OAuth callback on `127.0.0.1` on the machine running Hermes. Open the sign-in link in a browser on that same machine. If Hermes runs on a server and your browser is on your laptop, either paste the final redirect URL back at the terminal prompt Hermes shows, forward the callback port with `ssh -N -L <port>:127.0.0.1:<port> user@host`, or sign in from Hermes Desktop, which relays the callback for you.

<details>
<summary>Last resort: API key instead of OAuth</summary>

Use a key only if OAuth cannot run where Hermes runs. A key is a long-lived secret with the same power over your workspace as an OAuth grant, and it does not expire on its own.

Create a key at [sleek.design/agents/setup](https://sleek.design/agents/setup), then:

```bash
hermes mcp add sleek --url https://sleek.design/api/mcp --auth header
```

Paste the key at the prompt, then run `hermes config set mcp_servers.sleek.trust untrusted`. Hermes saves the key to your profile's `.env` as `MCP_SLEEK_API_KEY` and references it from `config.yaml`, so the key itself is never written to the config:

```yaml
mcp_servers:
  sleek:
    url: https://sleek.design/api/mcp
    headers:
      Authorization: Bearer ${MCP_SLEEK_API_KEY}
    trust: untrusted
```

Revoke the key from your Sleek account the moment it leaks.

</details>

## What you can ask for

**Design a whole app in one message.** Hermes creates a project, sends the whole brief as a single design run, shows you the new screens, and shares the project link. Send the whole app as **one** request: Sleek's design agent plans the screen set together, so one message for five screens gives a coherent app, and five messages give five unrelated screens and charge you five times.

**Look at what you already have, for free.**

> Show me the cart and profile screens from my plant shop project.

**Edit one screen.**

> On the plant detail screen, move the Add to cart button into a sticky bottom bar and make the price larger.

**Build the app.** Hermes pulls each screen's generated HTML with `get_component` and uses it as the design source for your code.

Every result comes with a project link. Open it to edit, or to export: Figma (native, editable layers), HTML, or React with Tailwind CSS, on every plan. You own everything you make.

## Costs

Design runs spend your Sleek workspace's AI credits:

| Action | Credits |
| :--- | :--- |
| New screen | about 30 |
| Edit a screen | 5 to 30, depending on how much changes |
| Listing, reading, showing, rendering | Free |

Free accounts get one-time trial credits, enough for about one design run. Sustained use needs the Pro plan or higher. See [sleek.design/pricing](https://sleek.design/pricing) for current plans.

The plugin declares these costs to Hermes, so the approval prompt for `design` shows them. To cap spending per chat, add `spend_limit` to the `mcp_servers.sleek` entry (see the Hermes docs on tool costs and spend limits).

## Tools

The server provides these tools. Hermes discovers them at runtime from `https://sleek.design/api/mcp`.

| Tool | What it does | Cost |
| :--- | :--- | :--- |
| `list_projects` | List the projects in your workspace, newest first | Free |
| `create_project` | Create an empty project to design into | Free |
| `get_project` | Fetch one project by id | Free |
| `delete_project` | **Permanently** delete a project and every screen in it | Free, cannot be undone |
| `list_components` | List the screens in a project | Free |
| `get_component` | Fetch one screen with its generated HTML | Free |
| `list_references` | List Sleek's curated design references to imitate | Free |
| `design` | Run the design agent to create or edit screens from a brief | Spends credits |
| `get_design_run` | Check the status of a design run | Free |
| `cancel_design_run` | Stop a queued or running design run | Free (credits already spent are not refunded) |
| `show_project` | Show a project's screens, highlighting what a design run changed | Free |
| `screenshot` | Render up to four screens into one image | Free |

Tool details, ids, and run behaviour are documented in the [main README](../README.md#tools).

## Disclosures

- **Network.** The only destinations are `sleek.design` and Sleek's OAuth authorization server, the Supabase Auth host `ggrhecslgdflloszjkwl.supabase.co`.
- **Tools come from the server.** The plugin pins the server URL, not the tool list. Sleek serves its tools at runtime, so the set can change without a new plugin version.
- **Spending.** `design` spends your workspace's AI credits. Hermes asks before each call.
- **Destructive tool.** `delete_project` permanently deletes a project and every screen in it. The server requires `confirm: true`, and the skill only calls it after you name the project.
- **Credentials.** Hermes runs the OAuth flow (PKCE, dynamic client registration) and stores and refreshes the tokens itself, in `~/.hermes/mcp-tokens/`. The plugin never sees, stores, or writes a credential.
- **Nothing else.** No code, no scripts, no hooks, no shell commands, no background processes, and no telemetry. The package is `plugin.json`, `mcp.json`, one skill, this README, and the license.

**What you're granting.** Approving the consent screen lets Hermes act as you in your Sleek workspace: list, create, and delete projects, design and edit screens, and spend the workspace's credits on design runs. It can't do anything your own account can't do. What Sleek stores is in the [privacy policy](https://sleek.design/privacy).

**How to revoke.** `hermes mcp remove sleek` removes the server and deletes its stored tokens; `hermes plugins remove sleek` removes the plugin. On the Sleek side, revoke the grant from your account at [sleek.design](https://sleek.design).

## Support

Email support@sleek.design, or read the [docs for agents](https://sleek.design/agents). For security reports see [SECURITY.md](../SECURITY.md).

Licensed under the MIT License. See [LICENSE](LICENSE).
