# Publishing

Maintainer notes for this repository. Nothing here is needed to *use* the Sleek MCP server; see the [README](../README.md) for that.

## Validate the manifests

```bash
claude plugin validate . --strict
```

This validates `.claude-plugin/marketplace.json` and, through its plugin entry, `.claude-plugin/plugin.json`; the output names only the marketplace manifest. `--strict` makes warnings such as unrecognized fields fail the run. To check the plugin manifest on its own: `claude plugin validate .claude-plugin/plugin.json --strict`.

## Local install test

From the repository root, start Claude Code and run:

```text
/plugin marketplace add ./
/plugin install sleek@sleek
/mcp
```

`/mcp` lists the plugin's server as `plugin:sleek:sleek`. It shows as connected before sign-in, because `initialize` and `tools/list` need no token. Sign in from `/mcp` or with `claude mcp login plugin:sleek:sleek`, then ask Claude to call `list_projects`; it should return your projects. Without sign-in the call returns 401 and Claude Code flags the server for authentication.

Clean up with `/plugin uninstall sleek@sleek` and `/plugin marketplace remove sleek`.

Opening this repository in Claude Code also offers to approve the `sleek` server from the root `.mcp.json` as a project-scoped server. That is the same definition the plugin ships.

## Open MCP Registry

`server.json` describes the server under the DNS-verified namespace `design.sleek`, the reverse-DNS form of `sleek.design`.

1. Install `mcp-publisher`: `brew install mcp-publisher`, or grab a release binary from https://github.com/modelcontextprotocol/registry/releases.
2. Generate an Ed25519 key pair and the DNS record. This needs OpenSSL 3. The macOS system `openssl` is LibreSSL and fails with `Algorithm Ed25519 not found`, so run `brew install openssl@3` and point `OPENSSL` at it. On Linux, set `OPENSSL=openssl`.

   ```bash
   OPENSSL=/opt/homebrew/opt/openssl@3/bin/openssl   # Intel Macs: /usr/local/opt/openssl@3/bin/openssl
   "$OPENSSL" genpkey -algorithm Ed25519 -out key.pem
   PUBLIC_KEY="$("$OPENSSL" pkey -in key.pem -pubout -outform DER | tail -c 32 | base64)"
   echo "sleek.design. IN TXT \"v=MCPv1; k=ed25519; p=${PUBLIC_KEY}\""
   ```

3. Add that TXT record at the apex of `sleek.design` (not under a selector such as `_mcp-auth`). Wait for propagation.
4. Log in with DNS verification, validate, and publish from the repository root:

   ```bash
   PRIVATE_KEY="$("$OPENSSL" pkey -in key.pem -noout -text | grep -A3 "priv:" | tail -n +2 | tr -d ' :\n')"
   mcp-publisher login dns --domain sleek.design --private-key "${PRIVATE_KEY}"
   mcp-publisher validate
   mcp-publisher publish
   ```

5. Confirm the listing at https://registry.modelcontextprotocol.io. Bump `version` in `server.json` for each publish.

`key.pem` is gitignored. Keep it out of the repository.

References: [authentication](https://github.com/modelcontextprotocol/registry/blob/main/docs/modelcontextprotocol-io/authentication.mdx), [remote servers](https://github.com/modelcontextprotocol/registry/blob/main/docs/modelcontextprotocol-io/remote-servers.mdx), [CLI commands](https://github.com/modelcontextprotocol/registry/blob/main/docs/reference/cli/commands.md).

## Claude Code plugin directory

1. Make this repository public and confirm `claude plugin validate . --strict` passes.
2. Submit the GitHub link at https://platform.claude.com/plugins/submit. The form is open to individual authors and needs a Developer, Admin, or Owner role on a Console organization. Team and Enterprise organizations can use the form in claude.ai organization settings instead.
3. Review is automated validation plus safety screening against the [published reviewer prompt](https://github.com/anthropics/claude-plugins-official/blob/main/.github/policy/prompt.md). The review reads every shipped file. This repository has no hooks, no shipped code, and no telemetry, and its description states the network destination, the credit cost, and the paid plan.
4. Each community catalog entry pins a commit from this repository. CI moves the pin as you push and the public catalog syncs nightly, so a change can take up to a day to appear; no resubmission needed. See https://claude.com/docs/plugins/submit and https://github.com/anthropics/claude-plugins-community.

## Connectors Directory (claude.ai)

This submission is for the server URL, not for this repository. Requirements from the [submission docs](https://claude.com/docs/connectors/building/submission):

- A Team or Enterprise organization with directory management access. The portal is in claude.ai organization settings.
- A fully populated reviewer test account with credentials and step-by-step access instructions.
- Listing assets: name (100 chars max), tagline (55 chars max), description (2,000 chars max), one to five categories, documentation URL, privacy policy URL, support contact, an icon, and a URL slug that is permanent once published.
- Tool annotations: every tool needs a `title` and `readOnlyHint` or `destructiveHint`. The server already publishes these.
- OAuth 2.0 and Streamable HTTP transport. Both are in place.
- Seven policy acknowledgments, including AI media generation. See the note below.

### Open items

Two things a reviewer may raise. Both are facts already visible in the public protected resource metadata; decide on each separately.

1. **The authorization server is on a `supabase.co` host, not `sleek.design`.** Cross-host authorization servers are supported as long as the protected resource metadata names the issuer, which it does. Serving the authorization server under `sleek.design` is the alternative.
2. **Client registration is dynamic.** Anthropic recommends a Client ID Metadata Document or Anthropic-held client credentials for high-traffic directory connectors, because dynamic registration creates a new client on every fresh connection. To have Anthropic hold a static client ID and secret, email mcp-review@anthropic.com. See [authentication](https://claude.com/docs/connectors/building/authentication) and the [review criteria](https://claude.com/docs/connectors/building/review-criteria).

### AI media generation policy

Anthropic's [Software Directory Policy](https://support.claude.com/en/articles/13145358-anthropic-software-directory-policy) lists "Software that uses AI models to generate images, video, or audio content" as unsupported, then carves out: "Design-focused software that uses AI models to create visual aids (such as slides, diagrams, charts, UI mockups, logos, or other design assets) are permitted. These servers may generate images as part of a design workflow, provided the developer does not offer standalone image generation as a primary service."

Sleek fits the carve-out: `design` returns editable HTML screens, `screenshot` renders existing screens to an image, and Sleek offers no standalone image generation. State this in the use-case fields.

## Cursor marketplace

Cursor lists plugins, which are Git repositories reviewed by the Cursor team; every plugin must be open source. Submit at https://cursor.com/marketplace/publish. The install link and `mcp.json` snippet in the README work without a listing.

## Source docs

Client instructions in the README were verified against:

- Claude connectors: https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp
- Claude Code: https://code.claude.com/docs/en/mcp and https://code.claude.com/docs/en/plugins
- Codex: https://developers.openai.com/codex/mcp/ and `codex mcp add --help` (codex-cli 0.136.0)
- ChatGPT: https://developers.openai.com/plugins/deploy/connect-chatgpt and https://developers.openai.com/plugins/quickstart
- Cursor: https://cursor.com/docs/mcp/install-links and https://cursor.com/docs/mcp
