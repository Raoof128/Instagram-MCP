# Changelog

Every version of the Instagram MCP design, newest first. Dates are Australia/Sydney. The
implementation's own audit trail is `AGENT.md` and `CHANGELOG.md` in
[comms](https://github.com/Raoof128/comms-mcp).

## 2026-10-05: reading Stories (revision 4)

- At the owner's request, the actor reads back the Stories it publishes. Two read tools, so 24:
  `story_list` returns the account's live Stories as ordinary `igm_` refs, and `story_insights`
  asks a Story's own metrics (`navigation`, `replies`, `profile_activity` and the rest).
- Story insights are checked before any call. A Story seen by fewer than five people answers
  `NOT_ENOUGH_DATA`. No new scope, host or table.
- Meta lists the stories edge under Instagram Login but shows only `graph.facebook.com` examples,
  so live gate GI-3 proves it on `graph.instagram.com`.
- Specification revision 4: D-I10 keeps only other accounts' Stories and highlights out. Ruling
  R-IG11 in comms; plan task IG-9; implemented on comms branch `comms-instagram-story-reads`,
  gated, and merged into comms `main` (merge `1f66f3e`, which also carries the rename).

## 2026-10-05: comms is now comms-mcp

- The comms repository and its Python package were renamed from `telegram-mcp` to `comms-mcp`.
  Every link here now points at [`Raoof128/comms-mcp`](https://github.com/Raoof128/comms-mcp).
  The gauntlet records keep the names they were written with; GitHub redirects the old links.

## 2026-10-04: Stories publishing (revision 3)

- At the owner's request, the actor publishes Stories as well as posts. `container_create` gains
  the kinds `story_image` and `story_video`; `publish` publishes a Story like any container.
- Checked first against Meta's Instagram Login publishing guide and the IG User Media reference:
  a Story takes only its image or video URL, supports no stickers, and a Story video runs 3 to 60
  seconds.
- Specification revision 3: D-I10 now excludes only *reading* Stories. Ruling R-IG10 in comms;
  plan task IG-8; schema migration v11 lets the container ledger record a Story.
- Implemented on comms branch `comms-instagram-stories`, gated, published by the real-daemon
  smoke, and merged into comms `main` at `9374c6c`.

## Repository presentation

- A README covering status, the 22 tools, the security model, a quick start and verification.
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md): components, a write step by step, the
  publishing ledger, storage and trust boundaries.
- [`SECURITY.md`](SECURITY.md), [`CONTRIBUTING.md`](CONTRIBUTING.md), the
  [Code of Conduct](CODE_OF_CONDUCT.md) and an [MIT licence](LICENSE).
- The specification, plan and gauntlet records moved under [`docs/`](docs/).
- The mirrors now match comms at `3874771`.
- Every branch merged into `main`. The independent v0.5 gauntlet from branch
  `claude/gallant-volta-qcs46n` is kept as
  [`docs/gauntlet/GAUNTLET-v0.5-independent.md`](docs/gauntlet/GAUNTLET-v0.5-independent.md).

## 2026-10-04: implemented and gated in comms

- Plan tasks IG-0 to IG-6 implemented in comms on branch `comms-instagram-spec`. Each commit
  passed the full gate on its own.
- 22 tools (14 reads, 8 writes), schema v10, the `igk_` publishing ledger, and
  `requiresUserInteraction` on the four public or irreversible writes.
- An end-to-end smoke through the installed binary covers all 22 tools on a real daemon.
- Deviations from the plan are rulings R-IG1 to R-IG9 in comms. Two defects were found and fixed
  on the way: a replayed request re-ran its pre-checks, and the conformance contracts had no
  cases.
- Mirrors updated at `3203831`.

## 2026-10-04: v0.6 revision 2 and the plan (`da9563e`)

- Revision 2 answers the v0.6 gauntlet. Publishing became three single-effect CREATEs on a
  durable container ref, so the existing mutation executor fits without changes. Every code
  change the design needs is listed in section 14.
- The implementation plan (tasks IG-0 to IG-7) added as `PLAN.md`.

## 2026-10-04: v0.6 gauntlet (`7e5995d` to `85cf0b1`)

- Verified the Python SDK and stdio-proxy claims live, checked every transcribed fact, and tested
  the fit against comms' code. Verdict: the actor design holds, and the publish flow needed
  re-cutting before adoption.

## 2026-10-04: v0.6, rewritten as a comms actor (`52759c5`)

- The design moved from a standalone TypeScript server to a Python actor inside comms, reusing
  its secret store, audit chain, replay safety, host permissions and daemon.

## 2026-10-04: v0.5 gauntlet (`5dcfff9`, `cdae96e`)

- Every external claim in v0.5 checked against Meta's developer documentation, the MCP
  specification and the Claude Code documentation, with live SDK runs. The record became the
  verification authority for the Meta facts.

## 2026-10-04: v0.5, the TypeScript specification (`7096036`)

- The first specification: a standalone Instagram MCP server in TypeScript.
