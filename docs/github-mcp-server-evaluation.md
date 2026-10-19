# GitHub MCP Server — Technical Evaluation Findings

Working notes for the team's tooling review. Source of truth for these findings: [github/github-mcp-server](https://github.com/github/github-mcp-server) as observed on GitHub.

## Evaluated repository

- **Full name:** `github/github-mcp-server`
- **Description:** GitHub's official MCP Server
- **Go module path:** `github.com/github/github-mcp-server/v2`

## Docker image and how the README says to run it

- **Docker image reference:** `ghcr.io/github/github-mcp-server`
- Local server via Docker with a Personal Access Token:
  ```bash
  docker run -i --rm -e GITHUB_PERSONAL_ACCESS_TOKEN ghcr.io/github/github-mcp-server
  ```
- The official image also ships OAuth app credentials, so it can log in with OAuth instead of a token (publish a fixed callback port to loopback):
  ```bash
  docker run -i --rm -p 127.0.0.1:8085:8085 -e GITHUB_OAUTH_CALLBACK_PORT ghcr.io/github/github-mcp-server
  ```
- The remote (hosted) server is available at `https://api.githubcopilot.com/mcp/` — the easiest path if your MCP host supports remote servers.

## Releases

Three most recent releases at time of evaluation:

| Tag | Release name | Published (UTC) |
| --- | --- | --- |
| `v2.0.2` | GitHub MCP Server 2.0.2 | 2026-10-08 11:33 |
| `v2.0.1` | GitHub MCP Server 2.0.1 | 2026-10-07 08:13 |
| `v2.0.0` | GitHub MCP Server 2.0.0 | 2026-10-06 23:34 |

**Latest release tag:** `v2.0.2` (published 2026-10-08)

## Toolset definitions in the source

Where the available toolsets are defined:

- **`pkg/github/tools.go`** — the primary definition point. Its header comment reads "Toolset metadata constants - these define all available toolsets and their descriptions," and each toolset is declared as `inventory.ToolsetMetadata{ID: "...", Description: "..."}`.
- **`pkg/github/ui_tools.go`** — declares tools against the `context` toolset (e.g. the `ui_get` tool used by MCP Apps UI pickers).

Toolset names (IDs) visible in `pkg/github/tools.go`:

`all`, `default`, `context`, `repos`, `git`, `issues`, `pull_requests`, `users`, `orgs`, `governance`, `actions`, `code_quality`, `code_security`, `secret_protection`, `dependabot`, `notifications`, `discussions`, `gists`, `security_advisories`, `projects`, `stargazers`, `labels`, `copilot`, `copilot_issue_intents`, plus remote-only toolsets `copilot_spaces` and `github_support_docs_search`.

Toolsets enabled by default: `context`, `repos`, `issues`, `pull_requests`, `users`, `copilot`.

## Notes for the review

- Two deployment modes: hosted remote server (`https://api.githubcopilot.com/mcp/`) and self-run local server (Docker image at `ghcr.io/github/github-mcp-server`, or release binaries).
- Toolsets are opt-in per client configuration; when none are specified, the default toolsets are used. The `all` toolset enables everything.
- Version 2.0.0 introduced support for Code Mode / Programmatic Tool Calling via structured outputs and output schemas (advertised only to clients supporting MCP 2026-07-28 or later).
- Go module path is `github.com/github/github-mcp-server/v2` (import prefix for Go consumers); the `v2.0.0` tag predates the module-path correction.
