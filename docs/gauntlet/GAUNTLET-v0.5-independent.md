> **An independent second run.** This gauntlet of SPEC v0.5 was made in a separate session (commit `5dcfff9`, branch `claude/gallant-volta-qcs46n`) on the same day as [`GAUNTLET-v0.5.md`](GAUNTLET-v0.5.md). That report is the canonical record the specification cites; this one is kept as a cross-check. Where the two differ, the canonical record and the v0.6 gauntlet govern.

# SPEC.md v0.5 gauntlet against 2026 developer docs

**Run:** 4 October 2026. **Target:** `SPEC.md` v0.5 (646 lines). **Method:** every external claim checked against the current official page, the npm registry, the installed package typings, or a live run of the real SDK. Line numbers refer to `SPEC.md`.

**How to read the verdicts**

| Mark | Meaning |
|---|---|
| ✅ | Confirmed against the live source, or proven by a run |
| 🔴 | Wrong. Edit the spec |
| 🟠 | Imprecise, incomplete or over-claimed. Tighten the wording |
| 🧪 | Still needs a live account or a live client. Unchanged |
| ⚠️ | Sourcing caveat (see below) |

**Sourcing caveat (⚠️):** `developers.facebook.com` is blocked by this cloud session's network policy, so every Meta verdict below rests on Context7's indexed copy of the current `/documentation/instagram-platform/` pages (quotes are verbatim page text, each with its source URL) rather than a direct fetch. Claude Code, MCP, SDK and npm verdicts are all direct. To make the Meta checks first-hand, add `developers.facebook.com` under Allowed domains in the environment's network settings (https://code.claude.com/docs/en/cloud-environments#network-access) and rerun section A.

---

## 0. Headline

| Area | Result |
|---|---|
| MCP SDK v2 usage (`serveStdio`, dual-era, `registerTool`, `_meta`) | ✅ Proven by live runs. **G6 and G7 are closed at the SDK level** (section C) |
| Keychain (`@napi-rs/keyring` 2.1.0) | ✅ Every API claim matches the shipped typings. Windows limit now sourced (2,560 bytes) |
| Claude Code integration | ✅ Every claim confirmed from the live docs. Four useful additions (section D) |
| Meta API facts | ⚠️ 5 corrections, 9 tightenings. Two live gates (G1, G2) can be mostly closed from the docs |
| Spec internal consistency | ✅ Tool count (25), section cross-references and gate list all line up |

**Readiness verdict stands:** nothing found blocks M1. The corrections below are wording and table fixes plus two gates that shrink.

---

## A. Meta Instagram API (sections 3, 7.2, 7.3, 7.5, 7.6, 10, 15) ⚠️

### A1. Corrections (🔴)

| Line | Spec says | Docs say | Edit |
|---|---|---|---|
| 74, 257 | "Latest documented is `v25.0`" | Graph API changelog: "The latest Graph API version is: v26.0" (released 29 July 2026). v25.0 is supported until 29 July 2028. Instagram Platform pages still show `v25.0` in examples | Keep default `v25.0` (it is current and long-lived). Change the claim to "latest Graph API is v26.0; Instagram Platform examples still use v25.0". Add a note to test `v26.0` before switching |
| 78, 564 (G2) | "Whether dashboard tokens are already long-lived: docs silent" | Get-started page: "Access tokens from the App Dashboard are long-lived and are valid for 60 days" | Mark ✅. Shrink G2 to: refresh works at 24 h, and whether the refreshed `access_token` value changes (still unstated) |
| 563 (G1) | `GET /me` fields "need a live check" | Get-started documents `/me?fields=`: `id` (app-scoped ID), `user_id` ("The Instagram professional account ID … the value of the id field received in webhook notifications"), `username`, `name`, `account_type` (`Business` or `Media_Creator`), `profile_picture_url`, `followers_count`, `follows_count`, `media_count` | Mark field names ✅. **The identity check must compare `user_id`, not `id`** (`id` is app-scoped and would differ per app). Store `user_id` in the registry. G1 shrinks to "Bearer header works on `graph.instagram.com`" |
| 298 | Feed and Reels metrics include `follows`, `profile_visits`, `profile_activity` | Media insights reference: `follows`, `profile_visits`, `profile_activity` are **FEED and STORY only**. `likes`, `comments`, `saved` are FEED and REELS | Move those three to a "Feed only" row. Requesting them on a Reel is one of the "vague unknown error" cases the spec warns about |
| 305 | breakdown `follow_type` | Account insights reference lists the `views` breakdown as `follower_type` | Rename. Also add the `timeframe` values: only `this_month` and `this_week` remain (the `last_14_days`, `last_30_days`, `last_90_days`, `prev_month` values were dropped from v20.0) |

### A2. Tightenings (🟠)

| Line | Issue | Edit |
|---|---|---|
| 97, 350 | "A carousel of N items uses N+1" and "Retries and failed attempts count" are **not in the docs**. The docs say only "400 containers within a rolling 24 hour period" and "containers expire after 24 hours" | Keep both as design assumptions, but mark them 🧪 (inference) rather than ✅ |
| 96, 565 (G3) | Spec doubts whether `content_publishing_limit` works on `graph.instagram.com` | The reference page's requirements table lists Instagram Login with host `graph.instagram.com` and scopes `instagram_business_basic` + `instagram_business_content_publish`. The 50 vs 100 conflict is real: the guide's intro says 100 (and its sample shows `"quota_total": 100`), the guide's carousel section and the reference's field text say 50 | Mark the host ✅. G3 becomes "read the live value" only |
| 101, 99 | Send API 100 per second | Instagram Platform overview says 100/s. The Messenger Platform changelog (8 October 2024) says 300/s for text, links, reactions and stickers. Private replies to **Live** comments have a separate 100/s limit | Note the conflict. Design to 100/s (the lower number) |
| 174, 508 | "App Roles, Roles, Instagram Tester, accept the invite in Instagram" | Current docs say only: "App testers must have a role on your app … and have a role on the Instagram professional account that owns the app". The Tester-invite walkthrough is sourced from a community thread and the ikas guide | Mark 🧪 (secondary source). Verify in the dashboard during `accounts add` for the second account |
| 76 | Dashboard path "API setup with Instagram login" | Get-started now says "Instagram > API setup with Instagram **business** login" | Update the label |
| 295 | `media_product_type`, `saved_count`, `shares_count`, `total_*`, `boost_*` documented as Facebook-Login-only | Confirmed for `media_product_type` and `boost_*`. `reposts_count`, `saved_count`, `shares_count` were added 22 April 2026 "for apps using Facebook Login". **`total_*` not verified** as Facebook-Login-only; the docs only say they are "not available for carousel child media" | Keep them out of the default set. G4 stays |
| 306 | "Demographics need 100+ followers or engagements" | `follower_demographics` and `follows_and_unfollows` need 100 followers. `engaged_audience_demographics` needs 100 engagements in the timeframe | Split the sentence |
| 297 | "period is always `lifetime`" | The media insights reference lists `period` values `day`, `week`, `days_28`, `month`, `lifetime`, `total_over_range` | Reword to "v1 always sends `lifetime`" (a design choice, not an API rule) |
| 345 | Reel spec | Missing: audio bitrate 128 kbps, aspect ratio 0.01:1 to 10:1. Also: plain **videos can be carousel items** (only Reels cannot) | Add. Decide whether v1 carousels accept video children (the spec currently implies images only) |
| 468, 567 (G5) | "Parse Meta's usage headers if present (G5 notes which)" | Graph API rate-limiting page: Instagram Platform calls are Business Use Case limited. Header is `X-Business-Use-Case-Usage` with `type: instagram` and fields `call_count`, `total_cputime`, `total_time`, `estimated_time_to_regain_access`. BUC throttle error code is `80002` | Name the header. Add `80002` to the backoff list (`4`, `17`, `32`, `613`, `80002`) |
| 493, 495 | "Code `10` … on this page" and "code `4` not on this page" | Code `10` lives on the media insights page, not the error-codes page. Code `4` **is** on the error-codes page (4/2207051) | Fix both footnotes |
| 609 | Error codes page "updated 2 June 2026" | Date not recoverable from the indexed copy | Drop the date or re-check once the host is reachable |
| 564 (G2) | "a third-party guide claims refresh fails for private accounts" | No official text supports it. The only private-account limitation found concerns webhooks on mentions | Keep as third-party, low weight |
| 353, 358 | Human Agent 7 days ✅. Disclosure "California and Germany named" | Disclosure wording was only found in a search snippet of the messaging page | Keep. Re-check verbatim once the host is reachable |

### A3. Confirmed (✅)

Lines 67, 71 to 73, 75 (one token per account, add-account flow), 77 (60 days, 24 h, `instagram_business_basic`, response fields), 84 to 88 (scopes, including the Overview omitting `manage_insights`), 98 (`4800 x impressions`, messaging excluded), 100, 102 (10,000), 104 ("strongly recommend" webhooks), 58 and 37 (delete is Facebook-Login-only, and now also needs the new `instagram_manage_contents` scope), 283 to 293 (all read endpoints), 300 to 302 (`impressions` cut-off 2 July 2024, no `engagement`, 48 h lag, no child insights), 304, 307, 312 to 320 (all write endpoints on the Instagram-Login path), 342 to 349 (URL-only, image, caption, carousel, container states, poll cadence, `is_ai_generated` rule), 354 to 357 (1,000 bytes, no groups, 20 most recent, 30-day requests folder), 472 to 492 (every error row, including the official 2207008 "try again 1 to 2 times in the next 30 seconds to 2 minutes" wording).

### A4. New since the spec was written (optional)

- `DELETE /<IG_MEDIA_ID>` exists for Facebook Login only, with the new `instagram_manage_contents` permission. Non-goal stands.
- Changelog 2026: `enable_fb_login` OAuth param (6 February), `reposts_count` / `saved_count` / `shares_count` (22 April, Facebook Login), multi-image messaging (6 May), oEmbed without a token (15 May), AI Info Label note on `is_ai_generated` (22 June).
- "Insights webhook for Instagram API with Instagram Login is not supported" (irrelevant to v1 polling, relevant to the v2 webhooks question).
- Media insight metrics are stored for up to 2 years; user metrics 90 days.
- Extra container params you have not listed as non-goals: `audio_name` (Reels), `trial_params.graduation_strategy`, `branded_content_sponsor_ids`, `is_paid_partnership`, `upload_type=resumable` (Facebook Login). Extra error rows: 2207035/36/37 (product tags), 2207073 (media type not supported).

---

## B. Keychain and npm (sections 4.4, 4.7, 9, 12) ✅

Checked directly against the npm registry and the shipped `index.d.ts` of `@napi-rs/keyring@2.1.0`.

| Line | Claim | Verdict |
|---|---|---|
| 26, 178 | 2.1.0, MIT, updated 13 September 2026, prebuilt for macOS x64/arm64, Windows x64/ia32/arm64, Linux x64/arm64 glibc and musl, arm, riscv64 | ✅ Registry: `2.1.0`, modified `2026-09-13`, MIT. Optional deps match exactly, plus `freebsd-x64`. `engines.node >= 10` |
| 178 | "Its repo README still shows an older version" | 🟠 The README shows no version number at all. It does document Linux store pinning and mentions `AsyncEntry`, but not `AbortSignal` |
| 179 | `keytar` archived 15 December 2022, needs `libsecret` | ✅ "This repository was archived by the owner on Dec 15, 2022." README: "this library uses `libsecret`" |
| 179 | `cross-keychain` less used | ✅ exists (1.1.0, last published 7 October 2025) |
| 183 | Windows per-credential limit "about 2.5 KB from memory" | ✅ now sourced: `CRED_MAX_CREDENTIAL_BLOB_SIZE = 5 * 512 = 2,560 bytes` (wincred.h). Replace "from memory" with the constant. The 500-byte JSON value is far under it, so the G9 size sub-check can be dropped |
| 187 | `AsyncEntry` not `Entry`; `AbortSignal` on every call | ✅ `Entry` methods are synchronous. `AsyncEntry.setPassword`, `getPassword`, `getSecret`, `deleteCredential`, `findCredentialsAsync` all take `signal?: AbortSignal` |
| 188 | `getPassword()` resolves `undefined` when absent, rejects when locked | ✅ Returns `Promise<string | undefined>` ("Returns no password if there isn't one"). The explicit "Rejects if the credential store cannot be read, for example when it is locked or inaccessible" sentence is on `getSecret` and `deleteCredential`; `getPassword` shares the path. Keep the rule; the mocked unit test covers it |
| 189 | `deleteCredential()` is honest | ✅ verbatim: "Resolves `true` if a credential was deleted, and `false` if there was no credential to delete. Rejects if the credential exists but could not be deleted … A failed deletion is never reported as `false`" |
| 191 | Linux auto-fallback to kernel keyring; pin `secret-service`; pinning fails instead of falling back | ✅ `LinuxEntryOptions.store?: 'secret-service' | 'keyutils'`: "When absent, the default auto-fallback selection is used (Secret Service, falling back to the kernel keyring). Requiring a store that is unavailable throws instead of falling back." |
| 619 | Inspector secret-storage doc | ✅ exists. It names the OS stores but not the Node binding it uses, so cite it for the probe idea, not for the library choice |
| 621 | presubmit PR 9 | ✅ exists: "Replace keytar with @napi-rs/keyring". Review flagged native errors mapping to `undefined` ("logged-out states"), fixed by bumping to 2.x, which is exactly rule 2 |

---

## C. MCP spec and TypeScript SDK (sections 5, 7, 12, 15) ✅

Checked against the spec repo (`2026-07-28` revision), `ts.sdk.modelcontextprotocol.io/v2`, the npm registry, the installed typings of `@modelcontextprotocol/server@2.3.0`, and three live runs.

### C1. Live runs (new evidence)

**G6 is closed at the SDK level.** `registerTool` config accepts `_meta?: Record<string, unknown>` and it reaches `tools/list` unchanged:

```
tools/list → { "name": "noargs", "_meta": { "anthropic/requiresUserInteraction": true, "custom": 1 }, ... }
```

Also proven in the same run: invalid args return `isError: true` with "Input validation error: …" and the handler never runs; valid args return `structuredContent`; `instructions` reaches the client; a tool with no `inputSchema` is listed with `{"type":"object","properties":{}}`.

**G7 is closed at the SDK level.** One `serveStdio(factory)` process, driven with raw JSON-RPC on stdin:

| Opening | Result |
|---|---|
| Legacy: `initialize` (2025-06-18) → `notifications/initialized` → `tools/list` → `tools/call` | 3 responses, 0 errors, tools listed, `pong` |
| Modern: `server/discover` with `_meta` envelope → `tools/list` → `tools/call` | `supportedVersions: ["2026-07-28"]`, tools listed, `pong` |
| Modern without discover: `tools/list` with `_meta` envelope straight away | tools listed, `pong` |

The factory ran exactly once per connection in all three cases. What remains of G7 is only "Claude Code and Claude Desktop actually connect" (section D).

### C2. Verdicts

| Line | Claim | Verdict |
|---|---|---|
| 21, 625 | Spec revision 2026-07-28 | ✅ `docs.json`: "Version 2026-07-28 (latest)"; schema `LATEST_PROTOCOL_VERSION = "2026-07-28"` |
| 5, 238 | SDK v2, `McpServer` from `@modelcontextprotocol/server`, `serveStdio` from `@modelcontextprotocol/server/stdio` | ✅ `server`, `client`, `core` all 2.3.0 (2 October 2026). `@modelcontextprotocol/sdk` 1.32.0 still ships and is not deprecated ("bug fixes and security updates for at least 6 months"). Signature: `serveStdio(factory: McpServerFactory, options?: { legacy?: 'serve' | 'reject'; transport?; onerror?; maxSubscriptions? })` |
| 33, 242 | Factory may be called per connection; state outside the factory | ✅ "ONE instance from the factory is pinned for the connection lifetime". The SDK source shows the factory can be called twice on one connection in the probe-fallback case (probe instance discarded), so the rule is necessary, not just cautious. The factory receives `{ era }` |
| 34, 243 | Dual-era mandatory | 🟠 The spec says servers **MUST** implement `server/discover` and **MAY** also serve legacy clients. `serveStdio` serves both by default (`legacy: 'serve'`). Add one rule: **never pass `legacy: 'reject'`**. A hand-wired `StdioServerTransport` serves only the 2025 era, so the spec's choice is right |
| 243 | Modern clients send per-request `_meta` | ✅ Required keys: `io.modelcontextprotocol/protocolVersion` and `io.modelcontextprotocol/clientCapabilities`; `clientInfo` is SHOULD. Missing keys give `-32602`; unsupported revision gives `-32022`. Use these exact keys in the G7 test client (line 529) |
| 239 | ES modules | 🟠 "ESM-first, though it ships concurrent CommonJS builds". `"type": "module"` is still the right choice |
| 239 | Zod v4 | ✅ server depends on `zod ^4.2.0`; "Zod v3 is no longer supported". `zod/v4` and plain `zod` (4.x) both work |
| 239 | Node 20+, `tsx` | ✅ `engines.node >= 20`; "There is no build step" |
| 240 | `registerTool` config fields | ✅ plus `icons`, `scopeChallenge`, `_meta`. **Add:** an `outputSchema` mismatch is **not** `isError`; the SDK throws a protocol error (`-32602`, "Output validation error"). Validate `structuredContent` yourself before returning, or that becomes a client-visible protocol failure |
| 244 | `notifications/cancelled`; polling loops honour the abort signal | ✅ Handler context exposes `ctx.mcpReq.signal: AbortSignal` ("cancelled from the sender's side"). 2026-07-28 restricts **servers** from sending `notifications/cancelled` except for `subscriptions/listen`; clients still send it |
| 268 | Tool names: letters, digits, underscore | 🟠 MCP also allows `-` and `.`, length 1 to 128, all SHOULD. Your stricter rule is fine and matches Claude Code's `mcp__server__tool` names |
| 467 | `isError` for failures, protocol errors for malformed requests | ✅ with the `outputSchema` caveat above |
| 526 | In-memory `Client` tests | 🟠 `InMemoryTransport.createLinkedPair()` "connects 2025-era instances only". Use it for the legacy half; for the modern half drive `serveStdio` over a pipe with raw JSON-RPC (as in C1) or use `handler.fetch` |
| 138 | "MCP has no connection session in the modern era" | ✅ 2026-07-28 "Remove protocol-level sessions and the `Mcp-Session-Id` header" |
| 633, 635 | Security best practices and connect-local-servers URLs | ✅ pages exist; canonical paths are versioned under `/docs/2026-07-28/…`, the unversioned URLs are redirects |

### C3. 2026-07-28 facts worth one line each in section 5

- Every result carries `resultType: "complete"` (or `"input_required"`). The SDK adds it; your handlers do not.
- `tools/list` results carry `ttlMs` and `cacheScope`. The SDK adds `0` and `private` by default, which is right for a policy-gated tool list.
- Logging, roots and sampling are deprecated. Your stderr-only logging is unaffected.
- `structuredContent` may be any JSON value; servers "MUST provide structured results that conform to" `outputSchema`, and SHOULD also put the serialised JSON in a text block (the spec already does this).
- Streamable HTTP now requires `Mcp-Method` and `Mcp-Name` headers. Irrelevant for stdio, relevant to open question 6.

---

## D. Claude Code and Claude Desktop (sections 7.4, 8, 9, 12) ✅

Checked directly against the live pages at code.claude.com (mcp, permissions, env-vars) and the MCP connect-local-servers page.

### D1. Verdicts

| Line | Claim | Verdict |
|---|---|---|
| 327 | `_meta["anthropic/requiresUserInteraction"]: true` prompts on every call; not skipped by `acceptEdits`, `auto`, `bypassPermissions`, allow rules or hooks; no "don't ask again"; `dontAsk` denies | ✅ verbatim on the MCP page. Permissions page adds: "also still prompt when a hook returns `allow`". The value "must be the JSON boolean `true`; any other value is ignored" |
| 327 | Headless runs deny the call | ✅ "The prompt has to reach a person." With `--permission-prompt-tool`, an allow is converted to a deny with the message "MCP tool requires user interaction". **Nuance:** the Agent SDK's `canUseTool` callback *can* approve them. Scheduled runs through `claude -p` stay read-only as the spec says |
| 268 | Descriptions truncated at 2,048 characters | ✅ and **the server `instructions` field is truncated at 2,048 characters too**. Your instructions must fit that budget. Override: `CLAUDE_CODE_MAX_MCP_DESCRIPTION_LENGTH` (v2.1.280+) |
| 268, 412 | Tool search defers MCP tools | ✅ "Only tool names and server instructions load at session start". No per-server cap |
| 270 | Root-level `anyOf`/`oneOf`/`allOf` flattened | ✅ "The Claude API doesn't accept those keywords at the schema root … Claude Code flattens the schema into a single object and prepends a sentence to the tool's description". Nested combinators inside `properties` pass unchanged |
| 394 | `.mcp.json` servers load without asking in `claude -p` and cloud sessions | ✅ verbatim |
| 396 | deny, then ask, then allow; globs after literal `mcp__instagram__`; bare deny removes the tool | ✅ "Allow rules accept tool-name globs only after a literal `mcp__<server>__` prefix". Deny and ask rules accept globs anywhere (`"mcp__*"`). Rule specificity does not change the order |
| 407 | Flagged write tools still prompt even when allowed | ✅ |
| 413 | 10,000 warn, 25,000 limit, `MAX_MCP_OUTPUT_TOKENS` | ✅ **Add:** a tool can raise its own limit with `_meta["anthropic/maxResultSizeChars"]` (ceiling 500,000) |
| 414 | Calls over 2 minutes move to background | ✅ verbatim |
| 415 | Stdio servers not auto-reconnected | ✅ "Claude Code doesn't reconnect them automatically" |
| 416 | Idle 30 min, `MCP_TIMEOUT` | ✅ `CLAUDE_CODE_MCP_TOOL_IDLE_TIMEOUT` (default 1,800,000 ms for stdio, v2.1.187+). Also `MCP_TOOL_TIMEOUT` and a per-server `"timeout"` field in `.mcp.json` |
| 417 | `CLAUDE_PROJECT_DIR` set for the server | ✅ "Claude Code sets `CLAUDE_PROJECT_DIR` in the spawned server's environment to the project root" |
| 418 | v2 runtime negotiates 2026-07-28; `MCP_PROTOCOL_NEGOTIATION=legacy` and `auto` | ✅ env-vars page: "`auto` to probe HTTP, claude.ai connector, and stdio servers, or `legacy` to probe none". **Add `MCP_SDK_GENERATION=v1|v2`** to the matrix: v1 is "built on MCP TypeScript SDK 1.x" and never probes; stdio probing arrives on v2.1.285+ "as Anthropic rolls that change out", so `auto` is the only way to force the modern path today |
| 419 | Server names | ✅ |
| 420 | Plugin names `mcp__plugin_<plugin>_<server>__<tool>` | ✅ |
| 423, 438 | Desktop config and log paths | ✅ `claude_desktop_config.json` paths and `mcp-server-<name>.log` confirmed on the MCP connect-local-servers page |
| 440 | `requiresUserInteraction` is a Claude Code feature | ✅ The Desktop guide does not mention it. Your preview-and-token flow remains the control there |
| 441 | `add-from-claude-desktop` on macOS and WSL | ✅ "This feature only works on macOS and Windows Subsystem for Linux (WSL)" |
| 444 | claude.ai web needs a remote HTTP server with OAuth | ✅ Optional note: Claude Desktop also installs local servers from `.mcpb` desktop-extension bundles, a packaging option for M8 |
| 454 | `Read(...)` deny rules best-effort | ✅ word for word: they cover built-in file tools, recognised Bash file commands and redirections, "not … `grep -r pattern .` … or arbitrary subprocesses … like a Python or Node script". OS-level enforcement is the sandbox, which "applies only to Bash, PowerShell, and Monitor commands and their child processes" |
| 531 | Matrix items | ✅ all testable as written. Add the `MCP_SDK_GENERATION` axis |

### D2. Four lines to add to section 8.1

1. Keep `instructions` under 2,048 characters (same cap as descriptions).
2. Test matrix axis: `MCP_SDK_GENERATION=v1` (legacy only) and `v2` with `MCP_PROTOCOL_NEGOTIATION=auto` (modern) and `legacy`.
3. `_meta["anthropic/maxResultSizeChars"]` is available if insights output ever needs more room.
4. Per-server `"timeout"` in `.mcp.json` for the resume-publish path.

---

## E. Gate list after the gauntlet (section 15)

| Gate | Before | After |
|---|---|---|
| G1 | `/me` field names and Bearer auth | Field names ✅ from docs (compare `user_id`). Only Bearer-on-`graph.instagram.com` stays live |
| G2 | Dashboard token lifetime, exchange, refresh at 24 h, private accounts | Lifetime ✅ (60 days, long-lived). Live: refresh at 24 h, whether the returned token value changes |
| G3 | Live `quota_total` and host | Host ✅. Live: the number (50 or 100) |
| G4 | `caption`, `media_product_type`, `is_comment_enabled` | Unchanged (`media_product_type` is Facebook-Login-only per docs, so expect it absent) |
| G5 | Metrics, `impressions`, usage headers, rate-limit codes | Header name ✅ (`X-Business-Use-Case-Usage`, code `80002`). Live: the rest. Add the Feed-only metric split to the test |
| G6 | `registerTool` passes `_meta` | **Closed** (live run). Keep the `ask`-rule fallback in the README as defence in depth |
| G7 | Legacy and modern clients both connect | **Closed at the SDK level** (live run). Live: Claude Code under `MCP_SDK_GENERATION` x `MCP_PROTOCOL_NEGOTIATION`, and Claude Desktop |
| G8 | Conversation and message shapes | Unchanged |
| G9 | Real keychain on macOS and Windows | Drop the Windows size sub-check (2,560-byte limit is documented). The rest unchanged |

---

## F. Reproduction

Everything in sections B, C and D can be re-run without an Instagram account:

```bash
npm view @napi-rs/keyring version time.modified          # 2.1.0, 2026-09-13
npm view @modelcontextprotocol/server version            # 2.3.0
npm i --ignore-scripts @napi-rs/keyring@2.1.0 @modelcontextprotocol/server@2.3.0 @modelcontextprotocol/client@2.3.0 zod@4
cat node_modules/@napi-rs/keyring/index.d.ts             # AsyncEntry, AbortSignal, LinuxEntryOptions
grep -n "legacy?: 'serve' | 'reject'" node_modules/@modelcontextprotocol/server/dist/stdio.d.mts
```

The G6 and G7 scripts used for section C1 are three short `.mjs` files (an in-memory client/server pair, a `serveStdio` server, and a raw JSON-RPC driver). They belong in `test/` once M1 starts; the driver already covers the three openings in the table above.

---

## References

Anthropic (2026) *Connect Claude Code to tools via MCP*. Available at: https://code.claude.com/docs/en/mcp (Accessed: 4 October 2026).

Anthropic (2026) *Configure permissions*. Available at: https://code.claude.com/docs/en/permissions (Accessed: 4 October 2026).

Anthropic (2026) *Environment variables*. Available at: https://code.claude.com/docs/en/env-vars (Accessed: 4 October 2026).

Anthropic (2026) *Network access, Cloud environments*. Available at: https://code.claude.com/docs/en/cloud-environments#network-access (Accessed: 4 October 2026).

Atom (2022) *node-keytar* [archived 15 December 2022]. Available at: https://github.com/atom/node-keytar (Accessed: 4 October 2026).

Brooooooklyn (2026) *keyring-node*. Available at: https://github.com/Brooooooklyn/keyring-node (Accessed: 4 October 2026).

Meta Platforms (2026) *Graph API changelog*. Available at: https://developers.facebook.com/docs/graph-api/changelog (Accessed: 4 October 2026, via indexed copy).

Meta Platforms (2026) *Instagram Platform changelog*. Available at: https://developers.facebook.com/documentation/instagram-platform/changelog (Accessed: 4 October 2026, via indexed copy).

Meta Platforms (n.d.) *Get started, Instagram API with Instagram Login*. Available at: https://developers.facebook.com/documentation/instagram-platform/instagram-api-with-instagram-login/get-started (Accessed: 4 October 2026, via indexed copy).

Meta Platforms (n.d.) *Rate limits, Graph API*. Available at: https://developers.facebook.com/docs/graph-api/advanced/rate-limiting (Accessed: 4 October 2026, via indexed copy).

Meta Platforms (n.d.) *Instagram Media Insights*. Available at: https://developers.facebook.com/documentation/instagram-platform/reference/instagram-media/insights (Accessed: 4 October 2026, via indexed copy).

Meta Platforms (n.d.) *Instagram Account Insights*. Available at: https://developers.facebook.com/documentation/instagram-platform/api-reference/instagram-user/insights (Accessed: 4 October 2026, via indexed copy).

Meta Platforms (n.d.) *IG User Content Publishing Limit*. Available at: https://developers.facebook.com/documentation/instagram-platform/instagram-graph-api/reference/ig-user/content_publishing_limit (Accessed: 4 October 2026, via indexed copy).

Model Context Protocol (2026) *Specification 2026-07-28: changelog, versioning, tools, cancellation*. Available at: https://modelcontextprotocol.io/specification/2026-07-28 (Accessed: 4 October 2026, via the specification repository).

Model Context Protocol (2026) *TypeScript SDK v2 documentation: serving legacy clients, protocol versions, testing, upgrade guide*. Available at: https://ts.sdk.modelcontextprotocol.io/v2 (Accessed: 4 October 2026).

Model Context Protocol (n.d.) *Connect to local MCP servers*. Available at: https://modelcontextprotocol.io/docs/develop/connect-local-servers (Accessed: 4 October 2026, via the specification repository).

npm (2026) *@napi-rs/keyring*, *@modelcontextprotocol/server*, *@modelcontextprotocol/client*, *@modelcontextprotocol/core*, *@modelcontextprotocol/sdk*. Available at: https://registry.npmjs.org (Accessed: 4 October 2026).

Wine Project (n.d.) *include/wincred.h* [`CRED_MAX_CREDENTIAL_BLOB_SIZE`]. Available at: https://github.com/wine-mirror/wine/blob/master/include/wincred.h (Accessed: 4 October 2026).
