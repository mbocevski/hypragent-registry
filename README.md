# HyprAgent MCP Registry

Curated list of [Model Context Protocol][mcp] servers that
[HyprAgent][hypragent]'s "Tools → Registry" tab can install in one
click.

The registry is **opt-in** in HyprAgent (since 0.5.6) — the daemon
only fetches this file when the user explicitly enables it in
Settings, after acknowledging a consent dialog explaining the
network call.

[mcp]: https://modelcontextprotocol.io
[hypragent]: https://github.com/mbocevski/hypragent

## Where this is served

The canonical URL the daemon fetches is:

    https://registry.hypragent.dev/registry.toml

This repo's `main` branch is published via GitHub Pages, with a
custom domain mapped onto `registry.hypragent.dev` (a dedicated
subdomain so the registry can grow independently of any future
content on the apex). The daemon fetches with `If-None-Match`
against the previous ETag, caches locally for 24h, and falls
back to the cached copy when the network is unavailable.

## What the daemon does with this file

For each `[[servers]]` entry, HyprAgent shows:

- A card in the Registry tab with the icon, name, and description.
- An **Install** button that opens the existing MCP server form
  pre-populated with the entry's `command` / `args` / `env`.

The user fills in any `env_required` values (with secrets routed
to `secret.toml` via the per-key `secret = true` flag) and clicks
Save. HyprAgent writes a normal `[mcp.<id>]` block to the user's
config; the third-party server then runs as a separate process
under the user's account.

**HyprAgent does not audit, sandbox, or run these servers.** The
registry is a curated list of canonical configurations — nothing
more. The trust model is the same as `npm install <package>`
followed by `node ./bin/<package>`: you are running upstream code
with your credentials.

## Schema

```toml
[[servers]]
id                  = "github"             # slug, becomes the [mcp.<id>] key
name                = "GitHub"             # human-readable label
description         = "..."                # one-line, shown in the card
icon                = "🐙"                 # emoji or absolute image URL
homepage            = "https://..."        # upstream project / docs
type                = "stdio"              # "stdio" | "sse" | "http"
command             = "npx"                # stdio only
args                = ["-y", "..."]        # stdio only
url                 = ""                   # sse / http only
default_permission  = "ask"                # "allow" | "ask" | "deny" applied
                                           # to every mcp__<id>__* tool
signature           = ""                   # reserved for future ed25519 signing

env_required = [
    {name = "GITHUB_PERSONAL_ACCESS_TOKEN", description = "...", secret = true},
]
env_optional = []
```

`env_required` / `env_optional` entries:

- `name` — environment variable name.
- `description` — shown to the user in the install dialog.
- `secret` — `true` → value written to `secret.toml [mcp.<id>]`
  (mode 0600). `false` → written to `config.toml [mcp.<id>.env]`.

## Adding or updating a server

1. Open a PR against this repo with the new `[[servers]]` entry.
2. Verify locally with the daemon: point your `~/.config/hypragent/
   config.toml` at a local copy via `[registry] url =
   "file:///path/to/your/registry.toml"`, restart the daemon,
   open the Registry tab.
3. PR description should explain why the server is worth
   curating: real-world utility, upstream maintainership, no
   surprising network behaviour beyond what the description says.

Servers that have not been updated in 12+ months by their
upstream maintainers will be flagged for removal. The list is
not a graveyard.

## Reporting issues with a listed server

If a registry entry is broken (wrong `command`, env-var schema
drift upstream, server itself misbehaves), open an issue here and
include:

- Which entry (`id`).
- What you observed (error message, hang, wrong behaviour).
- Whether the upstream project's own README still matches what
  the registry says.

If the issue is with HyprAgent's handling of a registry entry
(install dialog crashes, env-vars not routed correctly), open
the issue in [the HyprAgent repo][hypragent] instead.
