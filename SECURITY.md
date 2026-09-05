# Security policy

## Reporting a vulnerability

Email **support@sleek.design** with the details. Please do not open a public
GitHub issue for security problems.

Include what you can of the following:

- What you found and where: the Sleek MCP server (`https://sleek.design/api/mcp`),
  the OAuth flow, an API key, or a file in this repository.
- Steps to reproduce, including any request and response you captured. Redact
  access tokens and API keys before sending.
- The impact you believe it has.

We will reply to your report by email. Please give us reasonable time to fix
the issue before you publish anything about it.

## Scope

This repository contains no server code. It holds only manifests and
documentation that point at Sleek's hosted MCP server. Reports about the server,
its authentication, or the Sleek service itself go to the same address.

## Keeping your account safe

- Prefer OAuth. It never gives the client your password, and you can revoke the
  grant from the client's connector settings or from your Sleek account.
- Treat an API key (`sk_...`) as a secret. Keep it in an environment variable,
  never in a file you commit, and revoke it from your Sleek account if it leaks.
