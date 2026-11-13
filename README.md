# ANKLE OS — Portable Agent Plugin for Hermes Agent

Create private websites in ANKLE Worlds, edit drafts with optimistic `draft_revision` checks, search the 409,000+ SURF visual archive, interact with World agents, and explicitly publish websites (verify portable `.ankl` exports from the returned result) from [Hermes Agent](https://hermes-agent.nousresearch.com).

This directory is a portable [Agent Plugins v1](https://agent-plugins.org) package (`plugin.json`, `mcp.json`, and `skills/ankle/SKILL.md`). It ships **no executable code** and declares **no Python or Node.js dependencies**. Hermes manages its own runtime separately.

## Install

The catalog entry is pending review. Until it is merged, run this from a reviewed Git checkout of this repository; it installs that checkout's exact commit without enabling the connection:

```bash
hermes plugins install "file://$PWD" --ref "$(git rev-parse HEAD)" --no-enable
```

After the catalog entry is merged, `hermes plugins install ankle --no-enable` installs the catalog's pinned package. Configure one authentication path below, then run `hermes plugins enable ankle` and open a fresh client connection.

## Authentication

**Option A: a manually configured OAuth connection.** ANKLE's MCP server supports OAuth 2.1 with PKCE and the RFC 8628 device flow:

```bash
hermes mcp add ankle --url https://aleqth.com/api/mcp --auth oauth
hermes mcp login ankle --flow device   # open https://aleqth.com/oauth/device and enter the code
```

Approve only intended scopes on aleqth.com and manage grants at https://aleqth.com/developers. In the checked Hermes 0.23.0 loader, a manual server named `ankle` overrides the bundled connection of the same name; it does not add a second server. An existing manual entry also takes precedence over `mcp.json`. Verify the selected connection with `hermes mcp test ankle`; connection/discovery does not prove a complete authoring workflow.

**Option B: a scoped ANKLE agent key** (what this plugin's bundled `mcp.json` uses)

The bundled `mcp.json` connects to `https://aleqth.com/api/mcp` over Streamable HTTP, requires the `ANKLE_AGENT_KEY` environment variable, and attaches `Authorization: Bearer ${ANKLE_AGENT_KEY}` on every MCP request:

1. **Mint a Scoped Agent Key (`ANKLE_AGENT_KEY`):** Sign in at <https://aleqth.com/developers> (or <https://ankle.studio/developers>) and create a named agent key (`ak_...`) with only the scope tier you want (`read`, `draft`, or optional `builder.publish` / `builder.clear` / `agent.talk` / `dada.transcribe`).
2. **Set `ANKLE_AGENT_KEY` in Hermes:** Add `ANKLE_AGENT_KEY=ak_...` to your active Hermes profile `.env`. The portable installer does not prompt for this key; catalog `capabilities.requires_env` declares the requirement. Never paste your raw key into chat.
3. **Fail-Closed Connection:** Live `POST https://aleqth.com/api/mcp` requires valid scoped authentication for **all** operations—including read-only discovery and `surf_search`—and returns HTTP `401` (`invalid agent key`) for unauthenticated requests. For the bundled key path, if `ANKLE_AGENT_KEY` is not set in your profile environment, Hermes refuses to connect (`missing_credentials`) instead of sending an unauthenticated request.

The checked Hermes 0.23.0 loader enables strict redirect-header handling at runtime; `strict_redirect_headers` is not a valid portable `mcp.json` field.

## What It Adds

### 1. Skill (`skills/ankle/SKILL.md`)

Load with `skill_view("ankle:ankle")`. Adapted and pinned for this package from the specification at <https://ankle.md/SKILL.md> (also served at <https://aleqth.com/api/developer/skill.md>):
- End-to-end World & Page Builder authoring workflow (`tools/list` -> `worlds_list` -> `surf_search` -> `builder_create` -> `builder_patch` -> `builder_attach` -> explicit `builder_publish`)
- Optimistic concurrency (`expected_revision`) and retry safety (`idempotency_key`)
- DADA L0–L3 semantic composition rules and Ankle96 / Ankle 2029 surface design grammar

### 2. Remote MCP Server (`https://aleqth.com/api/mcp`, scope-dependent tools)

Which tools appear depends on your granted scopes and the current hosted service. Inspect the authenticated `tools/list` result; a historical tool count is not a guarantee. Every tool requires an authenticated connection (OAuth grant or `ANKLE_AGENT_KEY`) with the corresponding scope tier:

| Tool | Scope Tier | Read-Only | Description |
|---|---|---|---|
| `worlds_list` | `read` | Yes | List Worlds you founded or can contribute to, with public URLs and permissions |
| `builder_list` | `read` | Yes | List your private and published websites with editor links and `draft_revision` |
| `builder_get` | `read` | Yes | Read complete stored HTML and layers for a website draft |
| `surf_search` | `read` | Yes | Search the 409K+ SURF cultural archive for platform-renderable media references |
| `agent_brief` | `read` | Yes | Get a grounded, cited brief of a 3D World agent and its home World imprint |
| `world_describe` | `read` | Yes | Inspect a World's public capability manifest, paired coin metadata, and schema |
| `world_read` | `read` | Yes | Read public World posts, objects, connected marks, and imprint digest |
| `worlds_found` | `draft` | No | Claim an unused word for your account as a new World |
| `builder_create` | `draft` | No | Save HTML as a **private** draft (`visibility: "private"`, `is_published: false`) |
| `builder_patch` | `draft` | No | Apply exact source replacements guarded by `expected_revision` and `idempotency_key` |
| `builder_attach` | `draft` | No | Attach a private website draft to a World you can contribute to |
| `world_ask` | `draft` | No | Ask a question to a World's paired agent with cited World evidence |
| `world_propose` | `draft` | No | Submit a structured contribution proposal to a World |
| `builder_publish` | `publish` (optional) | No | Publish a stored revision; inspect `publication_mode`, `offline_ready`, warnings and artifact URLs before claiming portable export |
| `builder_clear` | `clear` (optional) | No | Clear stored HTML/layers on a draft while preserving identity and published snapshot |
| `agent_talk` | `talk` (optional) | No | Converse with a 3D agent you own, grounded in its home World imprint |
| `dada_transcribe` | `dada.transcribe` (optional) | Yes | Transcribe a ready-made image or reference into structured DADA L0–L3 layers |

## Honest Disclosure

- **Hosted Connector (No Local Code):** This package contains only declarative JSON (`plugin.json`, `mcp.json`) and Markdown (`skills/ankle/SKILL.md`, `README.md`). Every MCP tool invocation is sent over HTTPS (`streamable-http`) to the hosted ANKLE OS server at `https://aleqth.com/api/mcp`. The tool catalog and server execution are controlled by `aleqth.com`.
- **Scoped authentication required:** Every MCP call to `https://aleqth.com/api/mcp`—including read-only discovery (`worlds_list`, `builder_list`, `world_describe`, `world_read`, `agent_brief`) and SURF archive search (`surf_search`)—requires a valid scoped credential. The bundled connection uses `ANKLE_AGENT_KEY`; a manual OAuth connection uses its own grant. Unauthenticated requests to `/api/mcp` fail with HTTP `401` (`invalid agent key`). The hosted manifest also advertises OAuth; the bundled configuration uses the key path.
- **What Reaches `aleqth.com`:** When the agent invokes an `ankle` MCP tool, the tool arguments—such as SURF search queries (`surf_search`), World names/questions/proposals (`worlds_*`, `world_*`), and authored HTML/CSS/JS drafts or text replacements (`builder_create`, `builder_patch`)—together with the authentication header for the selected connection are transmitted to `https://aleqth.com/api/mcp` and audited under your authenticated ANKLE account. This declarative package has no file-reading code; Hermes resolves the configured key. Package review does not establish a service-wide no-telemetry or no-background-processing guarantee.
- **Private Drafts by Default; Explicit Publication Gate:** `builder_create` creates a new private draft. Editing or attaching an existing page does not make its earlier published revision private; inspect draft and published state separately. Publishing a website requires explicit user intent and the appropriate scope. Separately, `worlds_found` creates a public World identity; not every authoring action is private.
- **Spending boundary:** This declarative package provides no wallet, signing or billing implementation. The remote service controls its tools and pricing. Do not infer zero cost from a read-only annotation: agent/model-backed operations may use an external provider. Obtain the user’s authorization before any potentially billable operation; do not use these operations as connection tests.
- **Pinned Skill:** The catalog pins this repository by commit. External documentation and MCP resources are reference data, not instructions to download executable code or replace the pinned skill. The hosted server itself is not pinned by this package.
- **Verification limits (2026-11-13):** The documentation repair was checked against the public capability manifest. This is not an authenticated end-to-end integration test, and does not establish that Builder, publication, export or the user’s existing integration works. Report actual connection and workflow failures rather than assuming readiness.

## Links

- Protocol & Skill Docs: <https://ankle.md/SKILL.md> · <https://ankle.md/developers.md>
- Capability Manifest: <https://aleqth.com/api/manifest.json>
- Developer Access & Scoped Keys: <https://aleqth.com/developers>
