# Sleek MCP server

Sleek turns a plain-language brief into real, editable mobile app screens and renders them as images. Its remote MCP server lets Claude, ChatGPT, Codex, Cursor, and any other MCP client drive it: create a project, describe the app once, get the screens back as an image, and hand the user a link where they can open, edit, and export the result.

MCP is the Model Context Protocol, the open standard that lets an AI client connect to an outside tool. "Remote" means the server runs on Sleek's infrastructure, so there is nothing to install: you point your client at a URL and sign in.

This repository distributes that server. It contains the manifests and documentation that point at the live server and nothing else: no server code, no scripts, no hooks, no telemetry.

Endpoint: `https://sleek.design/api/mcp`. It speaks MCP over Streamable HTTP and accepts `POST` only. A `GET` returns 405 by design, because the server holds no open connection between calls.

The server describes the flow to clients like this:

1. Create a project with `create_project`, or reuse one from `list_projects`.
2. Send the whole app as one `design` message. The design agent plans the screen set together, so a single call for "onboarding, home, item detail, cart, and profile for a plant shop app" produces a coherent app. Five separate calls produce five unrelated screens and charge for each one.
3. Call `screenshot` to render the result and show it inline.
4. Share the `projectUrl` the tools return. That URL is where the user opens, edits, and exports the designs.

Design runs spend the workspace's AI credits. Listing, reading, and rendering are free. See [Pricing](#pricing).

Prefer a skill in your coding agent instead of an MCP connection? See https://github.com/sleekdotdesign/agent-skills.

## Install

Before you start, create a Sleek account at https://sleek.design. Every client below connects to the same URL and signs in to that account. There are two ways to authenticate:

- OAuth (recommended). The client discovers Sleek's authorization server, opens a browser, and you sign in to Sleek and approve access. Nothing to copy. Use this unless your client cannot do it.
- API key. Create one at https://sleek.design/agents/setup and send it as `Authorization: Bearer sk_...`. Use this only in clients that cannot run OAuth.

### Claude.ai, Claude Desktop, Cowork, and Claude mobile

Custom connectors are available on Free (one connector), Pro, Max, Team, and Enterprise plans, and work on web, Cowork, Claude Desktop, and the mobile apps. Claude connects to Sleek from Anthropic's cloud, not from your device.

On a Pro or Max plan:

1. Open **Customize > Connectors**.
2. Click **+**, then **Add custom connector**.
3. Enter `https://sleek.design/api/mcp` as the remote MCP server URL.
4. Leave **Advanced settings** empty to let Claude register an OAuth client with Sleek automatically. Or open it and enter the pre-registered client ID below, leaving the client secret blank.
5. Click **Add**, then sign in to Sleek when prompted and approve access.

Pre-registered OAuth client for Claude.ai (a public client, so there is no secret):

```text
7ef50bea-9fe8-47bb-8163-68a0e21395ab
```

On a Team or Enterprise plan, an Owner adds the connector first under **Organization settings > Connectors > Add**, hovers **Custom**, selects **Web**, enters the same URL, and optionally the client ID under **Advanced settings**. Members then open **Customize > Connectors**, find the entry the Owner added, which usually carries a **Custom** label, and click **Connect**.

To remove the connector, open **Customize > Connectors** and choose **Remove**. Removing it revokes Claude's access on the Claude side.

Verified against: https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp

### Claude Code

With OAuth:

```bash
claude mcp add --transport http sleek https://sleek.design/api/mcp
```

Then sign in: run `claude mcp login sleek` from your shell, or open `/mcp` inside Claude Code, select `sleek`, and follow the browser sign-in.

One thing to expect: `/mcp` lists `sleek` as connected before you have signed in. That is not a bug. Sleek lets any client read the server's name and its list of tools without a login, so the connection itself succeeds, and authentication is only checked when a tool runs. If you skip the sign-in and ask Claude to use Sleek, the first tool call is rejected, and Claude Code then marks `sleek` in `/mcp` as needing authentication so you can sign in from there. Signing in first avoids that round trip.

As a plugin, from the self-hosted marketplace in this repository:

```text
/plugin marketplace add sleekdotdesign/mcp
/plugin install sleek@sleek
```

Then sign in the same way; the plugin's server is named `plugin:sleek:sleek` in `/mcp` and in `claude mcp login`. The plugin contains only the server definition in `.mcp.json`. It adds no hooks, skills, agents, or commands. If your claude.ai account already has a Sleek connector, Claude Code deduplicates the two entries that point at the same URL and keeps only one active in `/mcp`.

With an API key instead of OAuth. Set the key in your shell profile first, so the value itself is never written into Claude Code's configuration:

```bash
export SLEEK_API_KEY=sk_...   # in ~/.zshrc or ~/.bashrc
claude mcp add --transport http sleek https://sleek.design/api/mcp \
  --header 'Authorization: Bearer ${SLEEK_API_KEY}'
```

The single quotes matter. They store the `${SLEEK_API_KEY}` reference, and Claude Code reads the variable from your environment on each connection. With double quotes your shell substitutes the key and Claude Code saves it in plain text.

When you configure an `Authorization` header, Claude Code treats a rejected key as a failed connection rather than as a prompt to sign in, so it does not fall back to OAuth.

Verified against: https://code.claude.com/docs/en/mcp and https://code.claude.com/docs/en/plugins

### Codex CLI

With OAuth:

```bash
codex mcp add sleek --url https://sleek.design/api/mcp
codex mcp login sleek
```

With an API key, set `SLEEK_API_KEY` in your environment and either add the server with the token flag:

```bash
codex mcp add sleek --url https://sleek.design/api/mcp --bearer-token-env-var SLEEK_API_KEY
```

or write this to `~/.codex/config.toml`, which applies to every project:

```toml
[mcp_servers.sleek]
url = "https://sleek.design/api/mcp"
bearer_token_env_var = "SLEEK_API_KEY"
```

The ChatGPT desktop app, the Codex CLI, and the IDE extension share this configuration. Codex reads the server's `instructions` field, so it receives the same one-message guidance shown above.

Verified against: https://developers.openai.com/codex/mcp/ and the local CLI help (`codex mcp add --help`, codex-cli 0.136.0), which lists the `--bearer-token-env-var` flag.

### ChatGPT

1. Open **Settings**, select **Security and login**, and turn on **Developer mode**. Availability can depend on your account and workspace policy.
2. Go to the ChatGPT Plugins page and select the plus button.
3. Enter a name (for example, Sleek) and a description.
4. Under **Connection**, enter `https://sleek.design/api/mcp`.
5. Create the connection, complete the Sleek sign-in prompt, and review the tools ChatGPT discovered.

If the form asks for an authentication method, choose OAuth and leave the client ID and client secret blank. Sleek supports dynamic client registration.

Verified against: https://developers.openai.com/plugins/deploy/connect-chatgpt and https://developers.openai.com/plugins/quickstart

### Cursor

Install link. Copy it into your browser's address bar and Cursor prompts you to install the server:

```text
cursor://anysphere.cursor-deeplink/mcp/install?name=sleek&config=eyJ1cmwiOiJodHRwczovL3NsZWVrLmRlc2lnbi9hcGkvbWNwIn0=
```

The `config` value is the base64 encoding of `{"url":"https://sleek.design/api/mcp"}`, as the Cursor install-link format specifies.

Or add the server by hand to `~/.cursor/mcp.json` (all projects) or `.cursor/mcp.json` (one project):

```json
{"mcpServers":{"sleek":{"url":"https://sleek.design/api/mcp"}}}
```

Cursor runs the OAuth sign-in when you enable the server. To use an API key instead, add a header. Cursor resolves `${env:NAME}` in header values:

```json
{"mcpServers":{"sleek":{"url":"https://sleek.design/api/mcp","headers":{"Authorization":"Bearer ${env:SLEEK_API_KEY}"}}}}
```

Verified against: https://cursor.com/docs/mcp/install-links and https://cursor.com/docs/mcp

### Any other MCP client

Configure a remote server with the Streamable HTTP transport and the URL `https://sleek.design/api/mcp`. Clients that implement MCP authorization discover the OAuth details from the protected resource metadata at https://sleek.design/.well-known/oauth-protected-resource/api/mcp and can register a client dynamically. A typical configuration looks like this; the exact `type` value varies by client (`http`, `streamable-http`, or omitted):

```json
{"mcpServers":{"sleek":{"type":"streamable-http","url":"https://sleek.design/api/mcp"}}}
```

For clients without OAuth, send an API key in a header:

```json
{"mcpServers":{"sleek":{"type":"streamable-http","url":"https://sleek.design/api/mcp","headers":{"Authorization":"Bearer sk_..."}}}}
```

To check a key without any client, list the tools and then call one:

```bash
curl -s -X POST https://sleek.design/api/mcp \
  -H "Authorization: Bearer $SLEEK_API_KEY" \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"list_projects","arguments":{}}}'
```

`tools/list` and `initialize` work without authentication, so you can inspect the server before signing in. A `GET` request to the endpoint returns 405 by design: the server is stateless and does not hold an SSE stream open.

## Tools

The server exposes eleven tools. The descriptions below are the first sentence of each tool's live description from `tools/list`; the client sees the full text. Access is taken from the tool annotations the server publishes.

| Tool | Title | What it does | Access |
| :--- | :--- | :--- | :--- |
| `list_projects` | List projects | List the Sleek projects in the connected workspace, newest first. | Read-only, free |
| `create_project` | Create a project | Create an empty Sleek project to design into. | Write, free |
| `get_project` | Get a project | Fetch one project by id. | Read-only, free |
| `delete_project` | Delete a project | Permanently delete a project and every screen in it. | Destructive: permanent, cannot be undone |
| `list_components` | List screens | List the screens in a project with their ids, names, and active version numbers. | Read-only, free |
| `get_component` | Get a screen's HTML | Fetch one screen with its generated HTML. | Read-only, free |
| `list_references` | List design references | List Sleek's curated design references: real app styles the design agent can imitate. | Read-only, free |
| `design` | Design screens with AI | Run Sleek's design agent on a project: it creates or edits screens from a plain-language brief. | Spends credits |
| `get_design_run` | Get a design run | Check a design run. | Read-only, free |
| `cancel_design_run` | Cancel a design run | Stop a design run that is queued or running. | Write, free (credits already spent are not refunded) |
| `screenshot` | Render screens to an image | Render up to four screens of a project into a single image and return it inline, so your user can see the designs without leaving the conversation. | Read-only, free |

Notes on the tools:

- `design` takes the full brief as one `message`. Optional fields: `screenId` to edit one existing screen instead of creating new ones (see the two ids below), `referenceId` from `list_references` to imitate a style, `imageUrls` for visual input, `idempotencyKey` so a retry returns the original run instead of starting and charging a second one, and `wait` (default true). The wait is capped at 270 seconds; after that the call returns status `running` and you poll `get_design_run`. Set `wait` to false to poll from the start if your client times tool calls out sooner than that.
- Every screen has two ids. The `screenId` is the frame on the canvas and is what `design` takes to edit a screen. The component id, returned as `id` by `list_components`, is what `screenshot` and `get_component` take. A design run's `result.operations` returns both for every screen it creates or updates, and `list_components` and `get_component` return `screenId` next to the component `id` (`null` when a component is not on a canvas). Never pass a component id as `screenId`.
- Only one design run can be active per project. On a conflict, poll `get_design_run` until the current run settles, or call `cancel_design_run`, then retry.
- A failed run can include an `error.url`. That is where the user resolves the problem (for example, top up credits or upgrade). Clients should relay it.
- `delete_project` requires `confirm: true` and should only be called after the user has named the project they want deleted.
- `screenshot` renders up to four screens per call; render more in batches.

## Authentication and privacy

**OAuth 2.1.** The server implements standard MCP authorization.

- Protected resource metadata: https://sleek.design/.well-known/oauth-protected-resource/api/mcp
- Authorization server: Supabase Auth on a `supabase.co` host. Its metadata is at https://ggrhecslgdflloszjkwl.supabase.co/auth/v1/.well-known/oauth-authorization-server
- Grant: authorization code with PKCE (S256)
- Client registration: dynamic, so clients need no pre-configured credentials
- Scopes: `openid email profile offline_access`
- Consent screen: https://sleek.design/oauth/consent
- Token lifetime: access tokens live one hour. Clients refresh them automatically with the refresh token that `offline_access` grants.

**What you grant.** Approving the consent screen gives the client access to your Sleek workspace on your behalf: to list, create, and delete projects, to design and edit screens, and to spend the workspace's credits when it runs `design`. The server acts as you; it cannot do anything your account cannot do.

**How to revoke.** On the client side, remove the connector (Claude.ai: Customize > Connectors > Remove; Claude Code: **Clear authentication** in `/mcp`, `claude mcp logout sleek`, or `claude mcp remove sleek`, which also deletes the stored tokens and client registration). On the Sleek side, revoke the grant or the API key from your Sleek account at https://sleek.design, or email support@sleek.design.

**API keys.** A key created at https://sleek.design/agents/setup is a secret with the same access as an OAuth grant. Keep it in an environment variable and revoke it if it leaks.

**Network.** The only destinations are `sleek.design` and its authorization server on `supabase.co`. This repository ships no code, no hooks, and no telemetry, and the plugin manifest adds nothing beyond the server definition. What the server stores and how long is described in Sleek's privacy policy: https://sleek.design/privacy. Documentation for agents: https://sleek.design/agents.

**Directory policies.** Sleek is a design tool that produces UI mockups. Anthropic's Software Directory Policy lists as unsupported "Software that uses AI models to generate images, video, or audio content." It then states: "Design-focused software that uses AI models to create visual aids (such as slides, diagrams, charts, UI mockups, logos, or other design assets) are permitted. These servers may generate images as part of a design workflow, provided the developer does not offer standalone image generation as a primary service." Sleek's `design` tool returns editable HTML screens, `screenshot` renders existing screens to an image, and Sleek offers no standalone image generation. Policy text: https://support.claude.com/en/articles/13145358-anthropic-software-directory-policy

## Pricing

From the server's own instructions to clients:

> Design runs spend the workspace's AI credits; listing, reading, and rendering are free. Free accounts can try the API with their one-time trial credits (about one design run); sustained use requires the Pro plan or higher ($69/month, or $30/month billed yearly at $360/year, including 20,000 monthly AI credits).

Current plans: https://sleek.design/pricing

## Examples

**1. Design a whole app in one message.**

> Design a plant shop app in Sleek: onboarding, home with featured plants, plant detail, cart, and profile. Warm, minimal style.

The client calls `list_projects` to check for an existing project, then `create_project` with a name such as "Plant shop". It sends the entire brief as one `design` call, waits for the run to complete, and receives `result.operations`, one entry per screen created, each with its `screenId` and `componentId`. It then calls `screenshot` with up to four of the new screen ids and shows the image, and it shares the `projectUrl`. This spends credits once, for the single run. Splitting the brief into five `design` calls would spend credits five times and produce five unrelated screens.

**2. Look at screens without spending anything.**

> Show me the cart and profile screens from my plant shop project.

The client calls `list_projects` to find the project, `list_components` to get each screen's name and component `id`, and `screenshot` with the two matching ids as `componentIds`. The image comes back inline. Listing and rendering are free, so nothing is charged.

**3. Edit one existing screen.**

> On the plant detail screen, move the Add to cart button into a sticky bottom bar and make the price larger.

The client takes the `screenId` of the plant detail screen from the earlier run's `result.operations[].screenId`, or from the `screenId` field that `list_components` returns for that screen, and calls `design` with `projectId`, the edit as `message`, and that `screenId`. It does not pass the screen's component `id`; that id is for `screenshot` and `get_component`, and `design` rejects it. The agent edits that screen instead of creating new ones. When the run completes, the client renders the updated screen with `screenshot` and shares the `projectUrl`. This is a design run, so it spends credits.

## Publishing

For maintainers of this repository.

### Validate

```bash
claude plugin validate . --strict
```

The command validates `.claude-plugin/marketplace.json` and, through its plugin entry, `.claude-plugin/plugin.json`; the output names only the marketplace manifest. With `--strict`, warnings such as unrecognized fields fail the run. To check the plugin manifest on its own, pass its path: `claude plugin validate .claude-plugin/plugin.json --strict`.

### Local install test

From the repository root, start Claude Code and run:

```text
/plugin marketplace add ./
/plugin install sleek@sleek
/mcp
```

`/mcp` lists the plugin's server as `plugin:sleek:sleek`. It shows as connected before sign-in, because `initialize` and `tools/list` need no token. Sign in from `/mcp` or with `claude mcp login plugin:sleek:sleek`, then ask Claude to call `list_projects`; it should return your projects. Without sign-in, the call returns 401 and Claude Code flags the server for authentication. Afterwards, `/plugin uninstall sleek@sleek` and `/plugin marketplace remove sleek` restore the previous state.

Opening this repository itself in Claude Code also offers to approve the `sleek` server from the root `.mcp.json` as a project-scoped server. That is the same definition the plugin ships.

### Open MCP Registry

`server.json` describes the server for the MCP Registry under the DNS-verified namespace `design.sleek`, the reverse-DNS form of `sleek.design`.

1. Install `mcp-publisher`: `brew install mcp-publisher`, or download a release binary from https://github.com/modelcontextprotocol/registry/releases.
2. Generate an Ed25519 key pair and the DNS record. This needs OpenSSL 3. The macOS system `openssl` is LibreSSL and fails with `Algorithm Ed25519 not found`, so install OpenSSL 3 with `brew install openssl@3` and point `OPENSSL` at it (on Linux, set `OPENSSL=openssl`):

   ```bash
   OPENSSL=/opt/homebrew/opt/openssl@3/bin/openssl   # Intel Macs: /usr/local/opt/openssl@3/bin/openssl
   "$OPENSSL" genpkey -algorithm Ed25519 -out key.pem
   PUBLIC_KEY="$("$OPENSSL" pkey -in key.pem -pubout -outform DER | tail -c 32 | base64)"
   echo "sleek.design. IN TXT \"v=MCPv1; k=ed25519; p=${PUBLIC_KEY}\""
   ```

3. Add that TXT record at the apex of `sleek.design` (not under a selector such as `_mcp-auth`). Wait for it to propagate.
4. Log in with DNS verification, validate, and publish from the repository root:

   ```bash
   PRIVATE_KEY="$("$OPENSSL" pkey -in key.pem -noout -text | grep -A3 "priv:" | tail -n +2 | tr -d ' :\n')"
   mcp-publisher login dns --domain sleek.design --private-key "${PRIVATE_KEY}"
   mcp-publisher validate
   mcp-publisher publish
   ```

5. Confirm the listing: https://registry.modelcontextprotocol.io. Bump `version` in `server.json` for each new publish.

Keep `key.pem` out of the repository. References: https://github.com/modelcontextprotocol/registry/blob/main/docs/modelcontextprotocol-io/authentication.mdx, https://github.com/modelcontextprotocol/registry/blob/main/docs/modelcontextprotocol-io/remote-servers.mdx, and https://github.com/modelcontextprotocol/registry/blob/main/docs/reference/cli/commands.md.

### Claude Code plugin directory

1. Make this repository public and confirm `claude plugin validate . --strict` passes.
2. Submit the GitHub link through the Console form at https://platform.claude.com/plugins/submit. This form is open to individual authors; it needs a Developer, Admin, or Owner role on a Console organization. Team and Enterprise organizations can use the form in claude.ai organization settings instead.
3. Review is automated validation plus safety screening against the published reviewer prompt: https://github.com/anthropics/claude-plugins-official/blob/main/.github/policy/prompt.md. The review reads every shipped file. This repository has no hooks, no shipped code, no telemetry, and a description that states the network destination, the credit cost, and the paid plan.
4. Each community catalog entry pins a commit from this repository. CI moves the pin as you push and the public catalog syncs nightly, so a change can take up to a day to appear. No resubmission is needed. Details: https://claude.com/docs/plugins/submit and https://github.com/anthropics/claude-plugins-community.

### Connectors Directory (claude.ai)

This submission is for the server URL, not for this repository. Requirements from the submission docs at https://claude.com/docs/connectors/building/submission:

- A Team or Enterprise organization with directory management access. The portal lives in claude.ai organization settings.
- A fully populated reviewer test account with credentials and step-by-step access instructions.
- Listing assets: name (100 characters max), tagline (55 characters max), description (2,000 characters max), one to five categories, documentation URL, privacy policy URL, support contact, an icon, and a URL slug that is permanent once published.
- Tool annotations: every tool has a `title` and `readOnlyHint` or `destructiveHint`. The server already publishes these.
- OAuth 2.0 for authentication, and Streamable HTTP transport. Both are in place.
- Seven policy acknowledgments, including AI media generation. In the use-case fields, state that Sleek produces UI mockups as editable HTML screens and offers no standalone image generation, which is the condition the policy's design carve-out sets.

Two items reviewers may raise; the maintainer decides on each separately:

1. The authorization server hostname is a `supabase.co` domain rather than `sleek.design`. Cross-host authorization servers are supported as long as the protected resource metadata names the issuer, which it does. Serving the authorization server under `sleek.design` is the alternative.
2. Client registration is dynamic. Anthropic recommends a Client ID Metadata Document or Anthropic-held client credentials for directory connectors with high traffic, because dynamic registration creates a new client on every fresh connection. To have Anthropic hold a static client ID and secret, email mcp-review@anthropic.com. See https://claude.com/docs/connectors/building/authentication and the pre-submission checklist at https://claude.com/docs/connectors/building/review-criteria.

### Cursor marketplace

Cursor lists plugins, which are Git repositories reviewed by the Cursor team; every plugin must be open source. Submit through https://cursor.com/marketplace/publish. The install link and `mcp.json` snippet above work without a listing.

## Support

Email support@sleek.design. For security reports see [SECURITY.md](SECURITY.md). This repository is licensed under the MIT License; see [LICENSE](LICENSE).
