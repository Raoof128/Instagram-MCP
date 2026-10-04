# Instagram MCP

**Instagram for AI assistants, built as an actor of the [comms](https://github.com/Raoof128/telegram-mcp) MCP gateway.**
Read your posts, insights, comments and DMs, moderate comments, reply to DMs and publish, from
Claude Code or any MCP client. Every write is audited, replay-safe and confirmed by the host.

![Status: proposed](https://img.shields.io/badge/status-proposed%20amendment%20A49-orange)
![Gate: passing](https://img.shields.io/badge/gate-passing%20(simulated%20Graph)-brightgreen)
![Live gates: pending](https://img.shields.io/badge/live%20gates-pending-lightgrey)
![Licence: MIT](https://img.shields.io/badge/licence-MIT-blue)

> **Where the code lives.** This repository holds the design: the specification, the
> implementation plan, the verification records and the architecture notes. The implementation
> is in [`Raoof128/telegram-mcp`](https://github.com/Raoof128/telegram-mcp) under
> `src/comms/transports/instagram/`, on `main`. Stories publishing is on branch
> [`comms-instagram-stories`](https://github.com/Raoof128/telegram-mcp/tree/comms-instagram-stories)
> until it merges.

## Contents

- [Status](#status)
- [What it does](#what-it-does)
- [The 22 tools](#the-22-tools)
- [Security model](#security-model)
- [Quick start](#quick-start)
- [Verification](#verification)
- [Repository map](#repository-map)
- [Design history](#design-history)
- [Contributing, security and licence](#contributing-security-and-licence)

## Status

| | |
|---|---|
| **Design** | Specification v0.6 rev 3, gauntleted twice ([`docs/gauntlet/`](docs/gauntlet/)) |
| **Implementation** | Plan tasks IG-0 to IG-6 and IG-8 (Stories) done in comms, each commit gated on its own |
| **Tests** | 273 new tests; the full comms suite and the end-to-end smoke pass |
| **Real Meta** | Not yet exercised. Live gates GI-1 to GI-8 are owner-run and pending |
| **Adoption** | Proposed amendment A49 to the comms spec; adopted only by ruling R-IG0 |

Nothing here has called the real Instagram API yet. Every test runs against a scripted
`graph.instagram.com`. Treat it as a well-tested candidate, not a production system.

## What it does

- **Accounts.** One or more Instagram professional accounts (Business or Creator), each named by
  an alias in `comms.json`. Each account has its own `writes` and `dms` ceiling, so a read-only
  account cannot post even if the model asks.
- **Reads.** Profile, media, media and account insights, comments and replies, tagged media,
  DM conversations and messages, and the publishing quota.
- **Publishing.** Feed posts (single images and carousels), Reels and Stories, each from a
  public `https` URL, then publish. A Story takes only its image or video: Meta accepts no
  caption, location or sticker on one. Every container is recorded in a ledger that enforces
  Meta's 400-per-day budget.
- **Moderation.** Reply to, hide, unhide and delete comments; turn comments on or off per post.
- **DMs.** Reply to a person who wrote within the last 24 hours, as Meta allows. Outside the
  window the tool answers `WINDOW_CLOSED` and sends nothing.

It uses the **Instagram API with Instagram Login** (`graph.instagram.com`). No Facebook Page is
needed.

## The 22 tools

All tools are named `comms_instagram_<name>`.

| Group | Tools | Kind |
|---|---|---|
| Accounts | `account_list`, `whoami`, `profile_get` | read |
| Media | `media_list`, `media_get`, `media_insights`, `account_insights`, `tag_list` | read |
| Comments | `comment_list`, `comment_replies` | read |
| DMs | `conversation_list`, `conversation_messages` | read |
| Publishing | `publish_quota`, `publish_preview` | read |
| Publishing | `container_create`, `carousel_create`, `publish` | write |
| Moderation | `comment_reply`, `comment_hide`, `comments_enabled_set`, `comment_delete` | write |
| DMs | `message_send` | write |

Every write needs an explicit `account` and a `request_id`. Four writes are public or
irreversible: `publish`, `comment_reply`, `comment_delete` and `message_send`. They carry
`requiresUserInteraction`, so Claude Code asks before every call, in every permission mode.

## Security model

The full mapping is in the [specification, section 12](docs/SPEC.md#12-security-a44-mapping) and
[`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md). In short:

- **Tokens never leave the daemon.** Each account's token is typed at a hidden prompt and kept in
  the daemon's 0600 secret store. It travels only in the `Authorization` header, never in a URL,
  argument, log, environment variable or this repository.
- **One pinned origin.** The Graph client can reach `https://graph.instagram.com` and nothing
  else. A media URL the model supplies is checked and handed to Meta. comms never fetches it.
- **No raw ids reach the model.** Accounts, containers, media, comments and people become opaque
  refs (`iga_`, `igk_`, `igm_`, `igc_`, `igp_`). A ref from one account is `NOT_FOUND` under
  another.
- **Content is fenced.** Captions, comments and DMs appear only in `untrusted_text`, so injected
  instructions in a comment cannot become authority.
- **Writes are deliberate.** Writes need a named account, pass that account's identity check and
  its `writes` or `dms` ceiling, and sit in the host's ask list.
- **Replay-safe and audited.** A repeated `request_id` returns the recorded outcome and never
  repeats an effect. Every write lands on comms' signed, anchored audit chain, without bodies or
  identities.
- **Honest outcomes.** Meta's documented errors map to fixed codes. Anything undocumented or
  ambiguous is `OUTCOME_UNKNOWN` and is never retried.

## Quick start

These steps run in comms. Its [install runbook](https://github.com/Raoof128/telegram-mcp/blob/main/docs/runbooks/install.md)
covers provisioning the daemon first.

1. **Describe the account** in `comms.json`. It holds no token and no ids.

   ```json
   {"instagram": {"default": "main",
                  "accounts": {"main": {"label": "My studio", "writes": false, "dms": false}}}}
   ```

2. **Add it.** Paste the dashboard token at the hidden prompt. comms proves it with `GET /me`
   before it becomes active.

   ```bash
   comms transport instagram account add main
   comms transport instagram doctor
   ```

3. **Connect Claude Code** through the stdio proxy. The
   [Claude Code runbook](https://github.com/Raoof128/telegram-mcp/blob/main/docs/runbooks/clients-claude-code.md)
   sets up the ask rules.

   ```json
   {"mcpServers": {"comms": {"command": "comms",
     "args": ["mcp", "--stdio", "--client-seed", "/Users/<you>/.config/comms/claude-code.seed"]}}}
   ```

4. **Refresh tokens** before they expire. Dashboard tokens last 60 days, and `comms doctor`
   warns 10 days ahead.

   ```bash
   comms transport instagram token refresh --all
   ```

Start with `writes: false`. Turn writes and DMs on per account once you trust the setup.

## Verification

Each plan task was committed separately and passed the full comms gate on its own. The final
state was then driven end to end through the installed binary.

| Check | Result |
|---|---|
| Unit and integration suite | All pass on the reference host (7 failures here are host-only and pre-existing) |
| End-to-end smoke | 116 of 118 (2 host-only); all 22 Instagram tools and a Story on a real daemon |
| Formal models, lint, types, build | Pass |
| Live gates against real Meta | Pending (owner-run) |

The smoke adds an account through a real terminal, drives every tool over HTTP, replays a DM,
runs both doctors, refreshes the token and verifies the audit chain. The exact record is the gate
ledger in comms' `AGENT.md`. Deviations from the plan are rulings R-IG1 to R-IG10 in comms'
`docs/verification/comms-v0.3-rulings.md`.

## Repository map

| Path | What |
|---|---|
| [`docs/SPEC.md`](docs/SPEC.md) | The specification, v0.6 rev 3 (a mirror of the canonical copy in comms) |
| [`docs/PLAN.md`](docs/PLAN.md) | The implementation plan, tasks IG-0 to IG-8 |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | How a call flows, the publishing ledger and the trust boundaries |
| [`docs/gauntlet/`](docs/gauntlet/) | The verification records: v0.5 (canonical, plus an independent second run) and v0.6 |
| [`CHANGELOG.md`](CHANGELOG.md) | Every version of the design |
| [`SECURITY.md`](SECURITY.md) | How to report a vulnerability, and what is in scope |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | How to propose a change |

## Design history

The project began as a standalone TypeScript MCP server (spec v0.5). Its gauntlet confirmed the
Meta facts but showed that a second server would duplicate what comms already had: a secret
store, an audit chain, replay safety, host permissions and a daemon. v0.6 rewrote the design in
Python as an actor inside comms. See [`CHANGELOG.md`](CHANGELOG.md) for the full sequence.

## Contributing, security and licence

- **Contributing:** see [`CONTRIBUTING.md`](CONTRIBUTING.md) and the
  [Code of Conduct](CODE_OF_CONDUCT.md).
- **Security:** please report vulnerabilities privately, as described in
  [`SECURITY.md`](SECURITY.md). Never put a token in an issue.
- **Licence:** [MIT](LICENSE) © 2026 Raouf.

Instagram is a trademark of Meta Platforms, Inc. This project is independent and is not
affiliated with, endorsed or sponsored by Meta.
