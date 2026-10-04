# Gauntlet report: SPEC.md v0.5

**Run:** 4 October 2026, against `SPEC.md` at commit `7096036`.
**Method:** every external claim in the spec was checked against the current primary source. Meta claims were fetched directly from developers.facebook.com (no network block this run). MCP claims were checked against the 2026-07-28 specification and the TypeScript SDK `main` branch plus the installed `@modelcontextprotocol/server` 2.3.0. Claude Code claims were checked against the raw markdown of code.claude.com. Keychain claims were checked against the shipped `@napi-rs/keyring` 2.1.0 typings and a native run on headless Linux. SDK behaviour was driven live over raw stdio (scripts in the appendix).

**Legend:** ✅ confirmed, ❌ wrong, 🔧 needs tightening, 🧪 stays a live gate, ℹ️ inference not doc text.

---

## 0. Verdict

**The spec holds. Nothing blocks M1.** About 110 external claims were checked.

| Outcome | Count |
|---|---|
| ❌ wrong | 7 |
| 🔧 tighten | 28 |
| ℹ️ inference stated as fact | 2 (lines 97 and 350) |
| Gates closed or narrowed | G1, G2, G6, G7 (SDK half), G9 (size sub-check). G3's host question answered by doc |
| New facts the spec should carry | 20 |

### The seven corrections

| Line | Spec says | Source says |
|---|---|---|
| 74 | Latest API version is `v25.0` | **`v26.0`**, introduced 29 July 2026. The Instagram Platform reference pages still show v25.0 in examples, which is where the spec got it. `v25.0` remains a valid default |
| 78, 564 | Dashboard token lifetime: "Docs silent" | Get Started page: **"Access tokens from the App Dashboard are long-lived and are valid for 60 days."** G2 shrinks to the refresh-at-24-hours check |
| 172, 503 | Exchange the dashboard token with the app secret, fall back to treating it as long-lived | Inverted. Dashboard tokens are already long-lived. The `ig_exchange_token` exchange is documented for **Business Login** short-lived tokens only. `IG_APP_SECRET` is not needed for the dashboard flow at all |
| 67 | `Authorization: Bearer <token>` | Not documented on any Instagram Platform page fetched. Every official example passes `access_token=` as a query parameter. Keep Bearer as the preferred mechanism but G1 must test it, with the query parameter as the fallback |
| 188 | `getPassword()` resolves `undefined` when absent | The typings say `string \| undefined` but the native binding resolves **`null`** (live run, both `AsyncEntry` and `Entry`, and `getSecret()` too). Compare with `== null`, never `=== undefined` |
| 450, 459 | "Validate inputs, enforce access, rate limit, sanitize outputs, audit" come from MCP Security Best Practices | They come from the **Tools** page of the spec (server MUSTs), and "log tool usage for audit" is a *client* SHOULD there. The Security Best Practices page covers confused deputy, token passthrough, SSRF, state-handle hijacking, local server compromise and stdio-only. Fix the citation |
| 298 | `follows`, `profile_visits`, `profile_activity` apply to Feed and Reels | **Feed and Story only.** The Media Insights table marks all three "FEED (posts), STORY". Reels get `reach`, `views`, `likes`, `comments`, `saved`, `shares`, `reposts`, `total_interactions` plus the three `ig_reels_*` metrics. A validator built from line 298 would send unsupported combinations and get "An unknown error has occurred" |

### Gates closed or narrowed by this run

- **G6 closed at the SDK level.** `registerTool` accepts `_meta` in its config and `tools/list` emits it verbatim in both eras (live run). Claude Code documents `anthropic/requiresUserInteraction` exactly as the spec describes. Only "Claude Code actually shows the prompt for this server" is left, and that is a smoke test, not a gate.
- **G7 closed at the SDK level.** One `serveStdio` process from one factory served a legacy `initialize` client, a modern client that opened with `server/discover`, and a modern client that opened with a bare `tools/list` carrying the envelope. All three listed tools and called a tool. What remains is "Claude Code under `MCP_SDK_GENERATION=v2` with `MCP_PROTOCOL_NEGOTIATION` unset, `auto` and `legacy`, plus Desktop".
- **G1 narrowed.** `/me` fields are documented: `id` is the app-scoped ID, `user_id` is the `<IG_ID>`, plus `username`, `name`, `account_type`, `profile_picture_url`, `followers_count`, `follows_count`, `media_count`. The identity check must compare **`user_id`**. G1 is now only "Bearer header accepted on graph.instagram.com".
- **G2 narrowed.** Dashboard tokens are long-lived by doc. Left: refresh succeeds once the token is 24 hours old, and the private-account question. Note the official rule that the dashboard-added account "must be public".
- **G9 size sub-check removed.** Windows `CRED_MAX_CREDENTIAL_BLOB_SIZE` is `5*512` = **2,560 bytes** (Microsoft wincred reference). Our entry is about 300 bytes. Drop "about 2.5 KB from memory".

---

## A. Meta: access, tokens, scopes, rate limits (sections 2, 3, 4.6, 11.2, G1, G2)

All direct fetches of developers.facebook.com.

| Line | Claim | Verdict | Evidence |
|---|---|---|---|
| 67 | Host `graph.instagram.com` | ✅ | "all endpoints are accessed via the graph.instagram.com host" (Overview) |
| 67 | `Authorization: Bearer` | ❌ undocumented | All examples use `access_token=` query param. Test in G1 |
| 71 | Comments, publishing, insights, mentions, messaging on this path | ✅ | Overview features table |
| 72 | No Facebook Page needed | ✅ | Page required only in the Facebook Login column |
| 73 | Standard Access enough for own accounts | ✅ | "If your app only serves your Instagram professional account or an account you manage, Standard Access is all your app needs." |
| 74 | `v25.0` latest | ❌ | "The latest Graph API version is v26.0" (Versioning guide). Changelog: v26.0 on 29 July 2026, v25.0 on 18 February 2026 |
| 75 | App Roles, Roles, "Instagram Tester" | 🔧 | Official text: "App Roles > Roles" and "Multiple accounts can be added for multiple testers". The literal role name "Instagram Tester" appears only in forum threads |
| 76 | Dashboard path "Generate access tokens, Add account" | 🔧 | Get Started: "Instagram > API setup with Instagram business login", button "Generate token". Two official pages disagree on the menu label |
| 77 | 60 days, 24-hour minimum age, `instagram_business_basic`, response fields | ✅ | Refresh reference, verbatim. Endpoint `GET /refresh_access_token?grant_type=ig_refresh_token&access_token=...` |
| 78 | Dashboard token lifetime unknown | ❌ | "Access tokens from the App Dashboard are long-lived and are valid for 60 days." |
| 84 | `instagram_business_basic` required with every other scope | ✅ | Each other permission lists it as a dependency |
| 85 | Insights scope omitted on Overview, listed on Insights reference | ✅ | Both pages checked. Note the Permissions reference has no page for this scope (404) |
| 86 | `content_publish` | ✅ | |
| 87 | `manage_comments` covers comments and commenter `username` | 🔧 | The Permissions reference entry is a copy-paste of the publishing text. Cite the IG Comment reference instead |
| 88 | `manage_messages` | ✅ | Conversations and Send pages both require it |
| 56 | No personal accounts | ✅ | "your app users must have an Instagram professional account" |
| 57 | Others' accounts need Advanced Access, App Review, Business Verification | ✅ | Verbatim on Overview |
| 58 | Delete media is Facebook-Login-only | ✅ | "This api only supports Instagram API with Facebook login only." |
| 98 | `4800 x impressions` per 24 h, per app and account pair | ✅ | Rate limiting page |
| 98 | "messaging excluded" | 🔧 | Messaging is not excluded, it has its own per-API counters. Also Business Discovery and Hashtag Search use ordinary Platform Rate Limits |
| 99 | Conversations 2 per second | ✅ | |
| 100 | Send 100 per second text, 10 per second audio and video | ✅ | |
| 101 | Private replies 750 per hour | ✅ | |
| 102 | Media list max 10,000 | ✅ on the Facebook-Login reference cited. The Instagram-Login media page is JavaScript-rendered and could not be read | |
| 172, 503 | Exchange first, long-lived fallback | ❌ | Inverted, see corrections |
| 174 | Tester role, accept invite, Business or Creator | 🔧 | Add the official requirement: the dashboard-added account "must be public" |
| 504 | Refresh every 50 days | ✅ | Also "Tokens unused for 60 days expire permanently" |
| 563 | G1 field names | ✅ documented | `id` app-scoped, `user_id` is the IG ID. Compare `user_id` |
| 564 | Private-account refresh limit | 🧪 stays | Refresh page silent. Create-an-app page requires a public account |

**New facts for the spec**
- Unversioned calls use the app dashboard's "Upgrade API Version" setting, not the latest. Always send an explicit version.
- `IG_APP_SECRET` becomes optional. Make the exchange an explicit `accounts add --exchange` opt-in for Business Login tokens.

---

## B. Meta: publishing, insights, comments, messaging, errors (sections 3 limits, 7.2, 7.3, 7.5, 7.6, 10)

All direct fetches of developers.facebook.com, HTML stripped and read in full, so quotes are verbatim from the live pages. Page stamps: Content Publishing 30 June 2026, IG User Media 28 September 2026, IG Media 12 August 2026, Media Insights 11 September 2026, Account Insights 16 June 2026, Messaging 28 September 2026, Error Codes 2 June 2026.

| Line | Claim | Verdict | Evidence |
|---|---|---|---|
| 96 | Guide says 100, carousel section and quota reference say 50 | ✅ conflict confirmed | "limited to 100 API-published posts within a 24-hour moving period" versus "limited to 50 published posts" and `quota_total` "(currently 50)". G3 stays. Host for `content_publishing_limit` is `graph.instagram.com` per the requirements table, so the G3 host question is answered by doc |
| 97 | 400 containers per rolling 24 h | ✅ | "An Instagram account can only create 400 containers within a rolling 24 hour period" |
| 97, 38 | Carousel of N costs N+1 | ℹ️ inference | No doc states it. Derivable: each child is a container plus the carousel container. Mark ℹ️ |
| 283 to 293 | Endpoints | ✅ | All present on the reference pages |
| 295 | `caption` in the default field set | 🔧 | IG Media node: "Caption. Excludes album children. **Available for Instagram API with Facebook Login only.**" `username`, `is_comment_enabled`, `thumbnail_url`, `alt_text`, `like_count`, `comments_count` carry no restriction. G4 must resolve `caption` or the default set breaks |
| 295 | Facebook-Login-only fields list | ✅ incomplete | Add `reposts_count` and the `collaborators` edge. `view_count` is Business Discovery only |
| 297 | Media insights period always `lifetime` | ✅ | Verbatim |
| 298 | `follows`, `profile_visits`, `profile_activity` on Feed **and Reels** | ❌ | All three are "FEED (posts), STORY". Correct Feed and Reels set: `comments`, `likes`, `saved` (Feed, Reels), and `reach`, `views`, `shares`, `reposts`, `total_interactions` (Feed, Reels, Story) |
| 298 | `profile_activity` breakdown `action_type` | ✅ | Values `BIO_LINK_CLICKED CALL DIRECTION EMAIL OTHER TEXT` |
| 299 | Reels-only metrics | ✅ | `ig_reels_video_view_total_time` is "Metric in development", `reels_skip_rate` "estimated and in development" |
| 300 | `impressions` only before 2 July 2024 | ✅ tighten | Also Feed and Story only, never Reels |
| 300 | `engagement` gone | ✅ | Absent from the metrics table. The Insights guide still shows a stale `metric=engagement,impressions,reach` example |
| 301 | 48 h lag, empty not zero, no child insights | ✅ | Verbatim |
| 302 | Unsupported combos give "unknown error" | ✅ | "the API will return an error ("An unknown error has occurred.")" |
| 304 | 14 account metrics | ✅ | Exact match |
| 305 | `period=day`, demographics `lifetime` plus required `timeframe`, `metric_type` | ✅ tighten | Only `reach` (and deprecated `impressions`) support `time_series`. Every other metric is `total_value` only |
| 305 | Breakdown `follow_type` | 🔧 doc inconsistent | Parameter list says `follow_type` (values `FOLLOWER NON_FOLLOWER UNKNOWN`), the `views` row and banner say `follower_type`. Test both spellings in G5 |
| 305 | `timeframe` | 🔧 | Values listed are `last_14_days, last_30_days, last_90_days, prev_month, this_month, this_week`, but "will no longer be supported beginning with v20.0" applies to the first four. Only `this_week` and `this_month` are safe. `timeframe` overrides `since` and `until` |
| 305 | Default lookback 24 h | ✅ | Verbatim |
| 306 | `impressions` deprecated | ✅ | "deprecated for v22.0 and will be deprecated for all versions on April 21, 2025" |
| 306 | Demographics need 100+, top 45 | ✅ | Also `follows_and_unfollows` needs 100 followers |
| 306 | Data kept about 90 days | ✅ account only | Media insights: "stored for up to 2 years" |
| 307 | Breakdowns only with `total_value` | ✅ | Verbatim |
| 312 to 315 | Publishing endpoints and status poll | ✅ | |
| 316 | Reply via `/replies` with `message` | ✅ | |
| 317 | Hide via `?hide=` | ✅ | Owner's own comments "will always be displayed, even if... hide=true" |
| 318 | `?comment_enabled=` | ✅ | "Live video Instagram Media not supported" |
| 319 | `DELETE /<comment_id>` | ✅ | Only the media owner can delete, "even if the user attempting to delete the comment is the comment's author" |
| 320 | `POST /<ig_id>/messages` | ✅ | Or `/me/messages`. Body `recipient:{id:<IGSID>}`, `message:{...}` |
| 342 | Resumable upload is Facebook-Login-only | ✅ | "Only for apps that have implemented Facebook Login for Business" |
| 343 | Image specs | ✅ | Verbatim |
| 343 | `alt_text` images only | 🔧 | "Only supported on a single image or image media in a carousel." So it **is** allowed on carousel image children |
| 344 | Caption limits, not on children, `location_id` not on children | ✅ | Verbatim |
| 345 | Reel specs, Reels not in carousels | ✅ | Spec omits "Audio bitrate: 128kbps" and the 0.01:1 to 10:1 aspect range (9:16 recommended) |
| 346 | Carousel 2 to 10, cropped to first | ✅ | |
| 347 | Container 24 h, five statuses, poll once a minute for 5 minutes | ✅ | Verbatim |
| 348 | `is_ai_generated`, not on children | ✅ | "Setting this parameter on carousel children will result in an error" |
| 350 | `2207042` means cap hit | ✅ | |
| 350 | "Retries and failed attempts count" | ℹ️ inference | No doc says so. `quota_usage` is "The number of times the app user has published an IG Container", which if anything implies successes only. Creation attempts do count toward the separate 400-container limit |
| 353 | Window 24 h | 🔧 | "For most user actions, it lasts 24 hours. When the user action is a person sending their first message after clicking a Click-to-Direct ad, the window may last up to 7 days" |
| 353 | Human Agent 7 days | ✅ | |
| 354 | UTF-8, 1,000 bytes, no groups | ✅ | Verbatim |
| 355 | 20 most recent messages | ✅ | Verbatim |
| 356 | Requests folder 30 days | ✅ | Verbatim |
| 357 | 2 per second | ✅ | Overview page |
| 358 | Disclosure, California and Germany | ✅ | Verbatim |
| 472 to 492 | 21 error rows | ✅ all match | `2207008` text matches exactly. Two notes: `2207001` has an empty user message, and `2207005`'s official solution is anomalous ("Possible permission error... Generate a new container"), so the spec's "JPEG" action is a sensible override, not doc text. "Retry once" actions are spec choices, the doc says "Try again" |
| 493 | Code `10` story metric | 🔧 source | Not on the error-codes page. It is on Media Insights: "(#10) Not enough viewers for the media to show insights" |
| 495 | `190`, `4`, `17`, `32`, `613` not on the page | ✅ | |
| 36, 59, 60 | Mentions means `/tags`, replies need webhook IDs | ✅ | "Set up a script that can parse the Webhooks notifications and identify comment IDs." Note: the Mentions page requires `instagram_business_basic` **and** `instagram_business_manage_comments`, while line 84 puts tags under basic alone |

**New facts for the spec**
- Two Reels metrics **throw** if the reel is not shared to Facebook: `crossposted_views` and `facebook_views`. Never include by default.
- `media_url` can be absent (copyrighted audio, flagged media, reel downloads off). "Treat media_url as an optional field... fall back to permalink or thumbnail_url."
- `GET /<IG_ID>/media` excludes Stories.
- `content_publishing_limit` `since` must be no older than 24 hours. Default field is `quota_usage`.
- Error subcodes not in the table: `2207073` (media type not supported on delete, Facebook-Login-only anyway), `INSTAGRAM_PLATFORM_API__INVALID_LOCATION_ID`, and `2207035` to `2207037` (product tags, out of v1).
- Every Instagram Platform reference page now says "The latest version is: v26.0".

---

## C. MCP protocol and SDK (sections 5, 7, 10, 12, G6, G7)

Checked against the 2026-07-28 specification, the SDK `main` docs and source, and the installed `@modelcontextprotocol/server` 2.3.0 (published 2 October 2026).

| Line | Claim | Verdict | Evidence |
|---|---|---|---|
| 21, 243 | Revision 2026-07-28 is current | ✅ | "The current protocol version is 2026-07-28." |
| 243 | Modern clients send per-request `_meta`, may call `server/discover`; legacy send `initialize` | ✅ | Versioning page. "Servers MUST implement `server/discover`. Clients MAY call it" |
| 243 | Modern-only server fails for legacy clients | ✅ | Compatibility matrix: "Legacy client, Modern server: Fails... Legacy clients have no fall-forward mechanism" |
| 243 | `serveStdio` handles era selection | ✅ live | Default `legacy: 'serve'`. Live run: one process served all three opening styles |
| 138 | No connection session in the modern era | ✅ | "MCP has no protocol-level session"; "an open connection, such as a STDIO process, is not a conversation or session" |
| 238 | Imports | ✅ | `McpServer` from `@modelcontextprotocol/server`, `serveStdio` from `@modelcontextprotocol/server/stdio`. Package exports confirm |
| 239 | Node 20+, ESM, `zod/v4`, `tsx` | ✅ | `engines.node >=20`, `"type": "module"` |
| 240 | `registerTool` config fields | ✅ incomplete | Typings also accept `icons`, `scopeChallenge`, `_meta` |
| 240 | Invalid args return `isError: true` before the handler | ✅ live | `Input validation error: Invalid arguments for tool ig_publish_image: url: ...` |
| 240 | Omit `inputSchema` for no-arg tools | ✅ live | SDK advertises an empty object schema on the wire |
| 241 | stdout is protocol only | ✅ | stdio transport page MUST |
| 242 | Factory per connection, state outside | 🔧 | The factory can run **twice per connection** (probe instance then legacy fallback). The factory receives `{ era }`. It must be cheap and side-effect-free: no keychain probe, no network in `createServer` |
| 244 | Cancellation is `notifications/cancelled` | ✅ live | The handler sees it as **`ctx.mcpReq.signal`**, not `extra.signal` (v1 name). My handler keyed on `extra.signal` saw nothing until I used `mcpReq.signal` |
| 245 / G6 | `_meta` passthrough | ✅ live | `tools/list` returned `"_meta":{"anthropic/requiresUserInteraction":true}` in legacy and modern eras |
| 268 | Tool names: letters, digits, underscore | 🔧 | MCP allows `[A-Za-z0-9_.-]`, 1 to 128 chars, case-sensitive, all SHOULD. The spec's subset is fine, the stated rule is not MCP's |
| 268 | `structuredContent` plus text copy | ✅ | "SHOULD also return the serialized JSON in a TextContent block" |
| 467 | Protocol errors only for malformed requests | ✅ | Tools page error taxonomy. "A tool handler cannot produce a protocol error" (SDK errors doc) |
| 467 | `outputSchema` mismatch | ℹ️ now settled | Live: returns `isError: true` with `Output validation error: Invalid structured content for tool bad_output: n: Invalid input: expected number, received string`. It is a tool error, not a JSON-RPC error. The SDK skips output validation on `isError` results |
| 526 | In-memory `Client` tests | 🔧 | `InMemoryTransport.createLinkedPair()` "connects 2025-era instances only". Modern in-process coverage is `createMcpHandler` + `handler.fetch` (HTTP). `serveStdio` dual-era needs a spawned process |
| 529 / G7 | Dual-era test design | 🔧 | Modern leg cannot run over the in-memory transport. Use `StdioClientTransport` with `versionNegotiation: { mode: 'auto' }`, or drive raw stdio as the appendix script does. SDK client default is legacy |
| 450, 459 | Security obligations attributed to Security Best Practices | ❌ attribution | Tools page: "Servers MUST: Validate all tool inputs; Implement proper access controls; Rate limit tool invocations; Sanitize tool outputs". Audit is a client SHOULD |
| 457 | Stdio only, no listener | ✅ | Security Best Practices: local servers "SHOULD... Use the stdio transport" |
| 625 to 641 | Reference URLs | ✅ all resolve | `basic/transports` is now an overview, the stdout rules live at `basic/transports/stdio`. `real-host.md` covers VS Code, Claude Code and Cursor, not Desktop |

### Live run (SDK 2.3.0, raw stdio, one server script, four client openings)

| Opening | Result |
|---|---|
| A. `initialize` 2025-06-18 | Negotiated 2025-06-18, `instructions` returned, tools listed with `_meta`, invalid args gave `isError`, valid call returned `structuredContent`, cancellation aborted `mcpReq.signal`, factory called once with `era=legacy` |
| B. `server/discover` with envelope | `supportedVersions: ["2026-07-28"]`, capabilities and `instructions` in the discover result, tools listed with `_meta`, calls work, factory called once with `era=modern` |
| C. bare `tools/list` with envelope | Same as B, no discover needed |
| D. `initialize` with unknown 2099-01-01 | Server answered with 2025-11-25 (the SDK's `LATEST_PROTOCOL_VERSION` for the legacy era) |
| Any modern request without the envelope | `-32602 Request is missing the required _meta envelope for protocol revision 2026-07-28` |

Protocol constants in `@modelcontextprotocol/core`: `LATEST_PROTOCOL_VERSION = 2025-11-25` (legacy era), `MODERN_WIRE_REVISION = 2026-07-28`, `DEFAULT_NEGOTIATED_PROTOCOL_VERSION = 2025-03-26`.

**New facts for the spec**
- Say explicitly that output-schema violations surface as `isError`, and validate `structuredContent` yourself so the model gets a useful message.
- Use `ctx.mcpReq.signal` for cancellation in polling loops.
- `mcpReq.log`, `elicitInput` and `requestSampling` are deprecated in 2026-07-28 and throw on a modern request. Log to stderr only, which the spec already does.
- The statelessness text ("Clients SHOULD NOT use an individual task, thread, or conversation as the lifetime boundary for the stdio process") is the citation for the "writes never fall back" rule at line 138.
- The Security Best Practices "State Handle Hijacking" section ("secure, non-deterministic handles... Expiring handles") is the right citation for the confirm-token design.

---

## D. Claude Code and Claude Desktop (sections 7.4 step 4, 8, 9, 12)

Checked against the raw markdown of code.claude.com `mcp`, `permissions`, `env-vars`, `headless`, `hooks`, `sandboxing`, and the MCP "connect local servers" guide.

| Line | Claim | Verdict | Evidence |
|---|---|---|---|
| 327 | `_meta["anthropic/requiresUserInteraction"]: true` prompts on every call | ✅ | "The value must be the JSON boolean `true`; any other value is ignored." |
| 327 | "shows its own prompt with the exact args" | 🔧 | Doc says "shows that tool's permission prompt on every call". It does not promise the args are shown. Soften |
| 327 | Not skippable by `auto`, `acceptEdits`, `bypassPermissions`, allow rules, PreToolUse hooks | ✅ | "even in acceptEdits, auto, and bypassPermissions"; "Allow rules that match the tool don't skip the prompt either"; hooks: "a hook can't skip its approval prompt with allow, with or without updatedInput" |
| 327 | No "don't ask again" | ✅ | Verbatim |
| 327 | `dontAsk` denies | ✅ | "In dontAsk mode, which never prompts, Claude Code denies the call instead." |
| 327 | Headless denies | 🔧 | True for `claude -p` and `--permission-prompt-tool` ("converted to a deny"). **Not** true for Agent SDK hosts: "The Agent SDK's canUseTool callback does receive these calls and can approve them." Remote Control shows the full prompt |
| 268 | Descriptions truncated at 2,048 | ✅ | Also **`instructions`**: "truncates each tool description and each server's instructions at 2,048 characters by default". Override `CLAUDE_CODE_MAX_MCP_DESCRIPTION_LENGTH` (v2.1.280+) |
| 268 | Tool search defers MCP tools | ✅ | "Only tool names and server instructions load at session start" |
| 270 / 41 | Root `anyOf`, `oneOf` flattened | ✅, line 41 incomplete | `allOf` too. Nested combinators pass unchanged. If flattening fails the tool is skipped and logged |
| 368 to 373 | `claude mcp add` syntax, `claude mcp get`, `/mcp` | ✅ | Default scope is local. `/mcp reconnect all` exists (v2.1.284+) |
| 386 to 388 | `.mcp.json` with `"type": "stdio"` and `${CLAUDE_PROJECT_DIR:-.}` | ✅ | The `:-.` default is needed because the variable is set in the server's env, not Claude Code's |
| 394 | Project servers load without asking in `-p` and cloud | ✅ | "In claude -p runs, Agent SDK sessions, and cloud sessions... it loads project-scoped servers without asking." |
| 396 | deny, ask, allow, first match | ✅ | Verbatim |
| 396 | Globs after literal `mcp__instagram__` | 🔧 | Correct for **allow**. Deny and ask rules are broader: `"mcp__*"` is valid there. `mcp__` rules with parentheses in settings are skipped |
| 396 | Bare deny removes the tool from context | ✅ | Also for glob denies |
| 40, 454 | `Read(~/.config/instagram-mcp/**)` valid, best-effort | ✅ | Covers file tools, recognised shell commands and redirections, "not... a Python or Node script that opens files itself" |
| 454 | Sandboxing for OS-level enforcement | 🔧 | "The sandbox covers shell commands only. Claude's file tools, MCP servers, and hooks run outside it." It protects the token from Bash-spawned scripts, not from the server process, which must read the token anyway |
| 413 | 10,000 warn, 25,000 to file | ✅ | `MAX_MCP_OUTPUT_TOKENS` default 25,000. Warning threshold fixed |
| 414 | Over 2 minutes to background | ✅ | Not for subagent calls or `-p` runs unless `CLAUDE_AUTO_BACKGROUND_TASKS=1` |
| 415 | Stdio not auto-reconnected | ✅ | Verbatim |
| 416 | Stdio idle 30 min, `MCP_TIMEOUT` startup | ✅ | `MCP_TIMEOUT` default 30,000 ms. Idle override `CLAUDE_CODE_MCP_TOOL_IDLE_TIMEOUT` |
| 417 | `CLAUDE_PROJECT_DIR` set for the server | ✅ | Verbatim |
| 418 | `MCP_PROTOCOL_NEGOTIATION=legacy` and `auto` | 🔧 | Values are exactly `auto` and `legacy` (v2.1.221+). Stdio probing happens only on v2.1.285+ under staged rollout, or always with `auto`. Unset is not the same as `auto`. Test matrix: unset, `auto`, `legacy`, crossed with `MCP_SDK_GENERATION=v1` and `v2` (v2.1.218+) |
| 419 | Server name characters | ✅ | Verbatim |
| 420 | Plugin naming | ✅ | Non `[A-Za-z0-9_-]` replaced with `_` |
| 423, 435, 438 | Desktop config and log paths | ✅ | MCP local-servers guide, verbatim |
| 440 | Desktop asks per tool call | 🔧 | Guide only says "Claude will request your approval" for an operation. Prompt granularity is undocumented |
| 440 | `requiresUserInteraction` is Claude Code only | 🔧 | Documented only in Claude Code docs, but also honoured by Agent SDK hosts and Remote Control. Desktop behaviour is undocumented |
| 441 | `add-from-claude-desktop` macOS and WSL | ✅ | Verbatim |
| 203 | "`claude -p` in cloud sessions" | 🔧 | Docs treat `-p` runs, Agent SDK sessions and cloud sessions as three kinds. List them |

**New facts for the spec**
- State a minimum Claude Code version. At least v2.1.212 (backgrounding), v2.1.280 (description length override) and v2.1.285 (stdio protocol probing).
- `anthropic/maxResultSizeChars` `_meta` exists (up to 500,000) as an alternative to pagination for one tool.
- Top-level schema property names must be 1 to 64 chars of `[A-Za-z0-9_.-]` and the schema must be valid JSON Schema 2020-12 or the tool is excluded (v2.1.216+).
- `claude mcp list` and `get` ignore committed `.mcp.json` approvals until the folder is trusted interactively (v2.1.196+), so a fresh clone shows "Pending approval".
- The "scheduled runs are read-only" conclusion holds for `claude -p`, not for an Agent SDK scheduler with `canUseTool`.

---

## E. Keychain storage (sections 4.4, 4.7, 9, 12, G9)

Checked against the npm registry, the shipped `@napi-rs/keyring` 2.1.0 typings, the keyring-node README, and a native run on this headless Linux container.

| Line | Claim | Verdict | Evidence |
|---|---|---|---|
| 178 | 2.1.0, MIT, updated 13 September 2026 | ✅ | npm: `2.1.0`, `MIT`, modified `2026-09-13` |
| 178 | Prebuilt for macOS x64 and arm64, Windows x64, ia32, arm64, Linux x64 and arm64 glibc and musl, arm, riscv64 | ✅ | 12 optional platform packages, plus `freebsd-x64` which the spec omits |
| 178 | "Its repo README still shows an older version" | 🔧 | The README shows no version number at all. Drop the sentence |
| 179 | `keytar` archived 15 December 2022, needs `libsecret` | ✅ | "This repository was archived by the owner on Dec 15, 2022." README: "this library uses libsecret so you may need to install it" |
| 179 | `cross-keychain` exists | ✅ | npm 1.1.0 |
| 183 | Value well under 500 bytes | ℹ️ | About 300 bytes with a typical long-lived token. Reasonable |
| 183 | Windows limit "about 2.5 KB from memory" | 🔧 now sourced | `CredentialBlobSize... cannot be larger than CRED_MAX_CREDENTIAL_BLOB_SIZE (5*512) bytes` = 2,560 bytes. Drop the G9 size sub-check |
| 187 | `AsyncEntry` not `Entry`, `AbortSignal` on every call | ✅ | `Entry` methods are synchronous. `AsyncEntry.setPassword`, `getPassword`, `getSecret`, `deleteCredential` all take `signal?: AbortSignal` |
| 188 | Absent resolves `undefined` | ❌ | Resolves **`null`** at runtime for `getPassword()` and `getSecret()`, on `AsyncEntry` and `Entry`. The `.d.ts` says `undefined`. Use `== null` |
| 188 | Rejects when locked or inaccessible | ✅ | `getSecret` docstring: "Rejects if the credential store cannot be read, for example when it is locked or inaccessible." Live: auto store threw `Couldn't access platform storage: AccessDenied` |
| 189 | `deleteCredential()` resolves `false` only if nothing existed | ✅ | "A failed deletion is never reported as `false`, so a `false` result always means the credential is absent from the store." Live: delete of absent entry returned `false` |
| 190 | Probe by round trip | ✅ sensible | Live: round trip on keyutils succeeded, on the auto and pinned Secret Service stores it threw. Class existence proved nothing |
| 191 | Linux auto-selects Secret Service, silently falls back to keyutils | ✅ README | "the binding silently falls back to the kernel keyutils keyring when no Secret Service is available" |
| 191 | Pin `linux: { store: "secret-service" }`, throws instead of falling back | ✅ typings and live | `LinuxStore = 'secret-service' \| 'keyutils'`. "Requiring a store that is unavailable throws instead of falling back." Live: pinned Secret Service threw a DBus error on this container |
| 191 | Kernel keyring loses tokens on reboot | ✅ README | "the keyutils store keeps credentials in kernel memory... will not persist across reboots" |
| 192 to 194 | Migration, refresh store, redaction | ✅ design | No external claim |
| 198 | Platform caveats | 🧪 G9 | Unchanged |
| 203 | Headless has no keychain | ✅ live | This container: no Secret Service, auto store `AccessDenied` |
| 525 | Native smoke under `dbus-run-session` with `gnome-keyring` | ✅ sensible | Matches the DBus error seen here |
| 619 | MCP Inspector secret-storage doc | ✅ exists | Tiered keychain, file, memory. It probes read and enumeration and notes that on Linux "single entries can be served from the kernel keyring (keyutils) without a Secret Service, but enumeration... needs the Secret Service itself". Library not named |
| 621 | tasksquatch PR 9 | ✅ exists | "Replace keytar with @napi-rs/keyring". Review flagged exactly the bugs the spec guards against: "Native errors can be mapped to undefined, making inaccessible stores appear logged out", stale `libsecret` guidance, and a suite that "fully mock[s] the native module without platform smoke coverage" |

One more live observation: in this container the **auto** store did not silently fall back to keyutils, it threw `AccessDenied`. The README's fallback is for "no Secret Service available"; a present but unreachable DBus is a failure. Rule 5 still stands, pin it anyway.

---

## F. Secondary references (lines 643, 645)

| Ref | Status |
|---|---|
| Blotato 2026 guide | Not fetched. Flagged by the spec as secondary for the 50 versus 100 conflict only. The conflict is now confirmed from the primary pages in section B, so the citation can go |
| ikas access-token guide | Not fetched. Used for tester-invite steps. The official Create-an-app page now covers those steps, so the citation can go |

---

## G. Recommended edits for v0.6

1. Line 74: "Latest documented is `v26.0` (29 July 2026). Default `v25.0`, configurable."
2. Lines 78, 172, 503, 564: dashboard tokens are long-lived. `accounts add` stores as-is and refreshes after 24 hours. Make `--exchange` an opt-in for Business Login tokens. `IG_APP_SECRET` optional.
3. Line 67: "Bearer header preferred, query parameter fallback, G1 decides."
4. Line 174: add "the account must be public".
5. Line 563: identity check compares `user_id`.
6. Line 183: "Windows cap is 2,560 bytes (`CRED_MAX_CREDENTIAL_BLOB_SIZE`)". Remove the G9 size sub-check.
7. Line 188: "resolves `null` (typings say `undefined`, test for both)".
8. Line 242: "the factory may run twice per connection and receives `{ era }`. No I/O in it."
9. Line 244: "`ctx.mcpReq.signal`".
10. Line 268: tool names per MCP are `[A-Za-z0-9_.-]`, 1 to 128. Add: `instructions` is also truncated at 2,048.
11. Lines 450, 459: cite the MCP Tools page for the server MUSTs and the Security Best Practices page for stdio-only and state-handle hijacking.
12. Line 467: "An `outputSchema` violation returns `isError: true`. Validate before returning."
13. Lines 526, 529: in-memory client is legacy only. Dual-era test drives a spawned process, see appendix.
14. Line 327: soften "with the exact args", add the Agent SDK exception.
15. Line 418: matrix is `MCP_SDK_GENERATION` × `MCP_PROTOCOL_NEGOTIATION` (unset, `auto`, `legacy`). Add minimum Claude Code version.
16. Line 454: say what sandboxing covers (Bash subprocesses) and does not (the MCP server itself).
17. Line 298: move `follows`, `profile_visits`, `profile_activity` (and `impressions`) to a "Feed only" row. Add a "never by default" row for `crossposted_views` and `facebook_views`.
18. Line 305: `timeframe` is `this_week` or `this_month` only. Only `reach` supports `time_series`. Breakdown spelling (`follow_type` versus `follower_type`) goes into G5.
19. Line 295: `caption` is documented Facebook-Login-only. Keep it in G4 and make the default set degrade without it. Treat `media_url` as optional.
20. Line 343: `alt_text` is allowed on carousel image children.
21. Line 353: window is 24 hours, or up to 7 days after a Click-to-Direct ad.
22. Line 493: cite the Media Insights page for error code `10`.
23. Line 84: tags need `instagram_business_manage_comments` as well as basic, per the Mentions page.
24. Mark lines 97 (N+1) and 350 ("retries count") as ℹ️ inference.
25. Drop the two secondary references (Blotato, ikas). Both facts are now sourced from primary pages.

---

## H. Sourcing caveats

- Meta pages were fetched directly this run. Two reference pages (`/reference/instagram-user` and `/reference/instagram-user/media`) are JavaScript-rendered and returned only navigation markup, so `/me` field semantics come from the Get Started page's field table and the 10,000-item cap from the Facebook-Login media reference the spec cites.
- WebFetch summarises pages through a small model. Quotes marked verbatim were requested as such. The Graph API changelog entry for v26.0 was also confirmed from the raw page.
- SDK source findings are from the `main` branch. The live run used the published `@modelcontextprotocol/server` 2.3.0, `@modelcontextprotocol/client` and `@modelcontextprotocol/core` latest as of 4 October 2026.
- Claude Code behaviours are version-gated and several depend on Anthropic-fetched feature flags. Numbers quoted are from the docs on 4 October 2026.
- Desktop's permission model and its handling of `anthropic/*` meta keys are not documented anywhere that was fetched. G7's Desktop leg and 8.2's "asks per tool call" stay observational.
- github.com HTML is blocked by this session's proxy (403). The keytar archive notice and the presubmit PR were read through WebFetch, and raw.githubusercontent.com was reachable for the README and the Inspector doc.

---

## Appendix: live test scripts

`server.mjs` (one factory, four tools, module-level state):

```js
import { McpServer } from '@modelcontextprotocol/server';
import { serveStdio } from '@modelcontextprotocol/server/stdio';
import { z } from 'zod/v4';

const state = { factoryCalls: 0 };

serveStdio((ctx) => {
  state.factoryCalls++;
  console.error(`[factory] call #${state.factoryCalls} era=${ctx?.era}`);
  const server = new McpServer(
    { name: 'instagram-gauntlet', version: '0.0.1' },
    { capabilities: { tools: {} }, instructions: 'Gauntlet test server.' }
  );
  server.registerTool('ig_publish_image', {
    description: 'Write tool with requiresUserInteraction _meta.',
    inputSchema: z.object({ account: z.string(), url: z.string() }),
    outputSchema: z.object({ account: z.string(), username: z.string() }),
    annotations: { destructiveHint: true, readOnlyHint: false },
    _meta: { 'anthropic/requiresUserInteraction': true }
  }, async ({ account }) => ({
    content: [{ type: 'text', text: `published for ${account}` }],
    structuredContent: { account, username: 'test_user' }
  }));
  server.registerTool('bad_output', { outputSchema: z.object({ n: z.number() }) },
    async () => ({ content: [{ type: 'text', text: 'oops' }], structuredContent: { n: 'nope' } }));
  server.registerTool('slow', { description: 'Waits until cancelled.' }, async (ctx) => {
    const signal = ctx.mcpReq.signal;            // v2 name; v1 was extra.signal
    await new Promise((r) => { const t = setTimeout(r, 10_000); signal.addEventListener('abort', () => { clearTimeout(t); r(); }); });
    return { content: [{ type: 'text', text: signal.aborted ? 'cancelled' : 'done' }] };
  });
  return server;
});
```

Client openings sent over the child's stdin, one JSON line each:

```json
{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"legacy","version":"0"}}}
```

```json
{"jsonrpc":"2.0","id":1,"method":"server/discover","params":{"_meta":{"io.modelcontextprotocol/protocolVersion":"2026-07-28","io.modelcontextprotocol/clientInfo":{"name":"modern","version":"0"},"io.modelcontextprotocol/clientCapabilities":{}}}}
```

```json
{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{"_meta":{"io.modelcontextprotocol/protocolVersion":"2026-07-28","io.modelcontextprotocol/clientInfo":{"name":"modern","version":"0"},"io.modelcontextprotocol/clientCapabilities":{}}}}
```

Keychain probe (`@napi-rs/keyring` 2.1.0, headless Linux, no Secret Service):

| Store | Result |
|---|---|
| auto | `Couldn't access platform storage: AccessDenied` |
| pinned `secret-service` | `Platform failure: DBus error: ... set your DBUS_SESSION_BUS_ADDRESS instead` |
| pinned `keyutils` | set, get `x`, delete `true`, get again `null` |
