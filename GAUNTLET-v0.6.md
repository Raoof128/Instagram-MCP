# Gauntlet report: SPEC v0.6 (Instagram for comms, proposed amendment A49)

**Run:** 4 October 2026, against `docs/instagram-spec-v0.6.md` at `d9b6aa9` in `Raoof128/telegram-mcp` and its mirror `SPEC.md` at `52759c5` here.
**What this gauntlet covers.** The Meta, MCP and Claude Code facts in v0.6 were verified first-hand earlier today (`GAUNTLET-v0.5.md`), so they are checked here only for faithful transcription. The new material is what v0.6 claims about **comms itself** (amendments, modules, enums, tests, settings), its **internal consistency**, and the **Python SDK** claims. The SDK claims were driven live.

**Legend:** ✅ confirmed, ❌ wrong, 🔧 tighten, 🛠️ a code change the spec implies but does not state, 🧪 stays a gate.

---

## 0. Verdict

PENDING_VERDICT

---

## A. Python SDK and the stdio proxy (sections 0, 11, 13; gates GI-6, GI-7)

Live run on `mcp` 2.2.0 and `mcp-types` 2.2.0 from comms' own `uv` environment. The test server is a copy of `build_server` in `src/comms/mcp/stdio_proxy.py` (lowlevel `Server` with `on_list_tools` returning `ListToolsResult.model_validate(payload)`, served by `stdio_server()`), with a `tools/list` payload whose first tool carries `"_meta": {"anthropic/requiresUserInteraction": true}`.

| Claim | Verdict | Evidence |
|---|---|---|
| `mcp-types` 2.2.0 is dual-era | ✅ | `KNOWN_PROTOCOL_VERSIONS` ends `2025-11-25, 2026-07-28`; `MODERN_PROTOCOL_VERSIONS = ("2026-07-28",)`; `DEFAULT_NEGOTIATED_VERSION = "2025-03-26"` |
| The proxy speaks 2026-07-28 to the daemon | ✅ | `PROTOCOL_VERSION = "2026-07-28"` and the envelope keys in `http_post` |
| The proxy's host side handles both eras | ✅ live | A legacy `initialize` (2025-06-18) negotiated `2025-06-18`; a modern `server/discover` returned `supportedVersions: ["2026-07-28"]`; a bare modern `tools/list` with the envelope was served. The lowlevel `Server` has a default `server/discover` handler (`_handle_discover`) and reserves `initialize` for the runner |
| `_meta` on a tool reaches the host through the proxy | ✅ live | `Tool.meta` is `Field(alias="_meta")`; `ListToolsResult.model_validate` accepted it and all three eras emitted `"_meta": {"anthropic/requiresUserInteraction": true}` on the wire |
| `tools/call` passes through unchanged | ✅ live | `{"content":[...],"isError":false}` in both eras; the modern era adds `resultType` and the `serverInfo` envelope |
| "Nothing Instagram-specific is added to the proxy" | ✅ | The proxy forwards `tools/list` and `tools/call` verbatim; a new `_meta` key needs no proxy change |

**Consequence for the gates.** GI-7's SDK half is closed (the proxy itself is dual-era), and GI-6's SDK half is closed (the flag reaches the host). What remains in both is only "Claude Code and Desktop actually connect and honour it", which is a smoke test, not an unknown. Section 13 should say so.

---

## B. Claims about comms (sections 0 to 5, 8 to 13)

PENDING_SECTION_B

---

## C. Transcription from the v0.5 gauntlet and internal consistency

The mirror `SPEC.md` is byte-identical to the comms copy apart from its two-line banner.

### C.1 The 25 recommended edits from `GAUNTLET-v0.5.md` section G

| Status | Items |
|---|---|
| Done | 1, 2, 4, 5, 10, 11 (by removal), 14, 15, 17, 18, 19, 20, 21 (in section 7), 22, 23, 24, 25 |
| Not applicable, the TypeScript stack is retired | 6, 7, 8 (principle kept: no I/O at boot), 13, 16 |
| **Partial** | 3: GI-1 is now a hard gate with no query-param fallback (deliberate, A26). But the fallback text "a POST-body token for reads" is incoherent for GET endpoints, and the **refresh call** `GET /refresh_access_token?grant_type=ig_refresh_token&access_token=...` is the one documented form that puts the token in the URL. GI-1 must name it, or section 4.4 needs a 🧪 |
| **Mis-transcribed** | 9: section 6 says "honouring cancellation (`ctx.mcpReq.signal` on the proxy side)". That is the **TypeScript** SDK 2.3.0 handler name. v0.6 runs the Python `mcp` 2.2.0, where it does not exist. Replace with the Python SDK's cancellation mechanism and mark 🧪 |
| **Missing** | 12: the rule "validate `structuredContent` before returning" (an output-schema violation surfaces as `isError`) is not carried anywhere |

New facts from the v0.5 gauntlet still missing: `content_publishing_limit` `since` must be no older than 24 hours (default field `quota_usage`); `INSTAGRAM_PLATFORM_API__INVALID_LOCATION_ID` is absent from `IG_CODES` although `location_id` is an accepted argument (it would land in `OUTCOME_UNKNOWN`); Claude Code excludes a tool whose schema is not valid JSON Schema 2020-12 (v2.1.216+); the Reel aspect range 0.01:1 to 10:1.

No ❌ fact was re-introduced. Three places put ✅ on something still open: the `conversation_messages` row (GI-8 still tests the shapes), "neither works on live video" for hide and delete (the gauntlet records the live-video restriction only for `comment_enabled`), and error code `0` in the token row (unsourced; the gauntlet lists `190` only). The token shape `[A-Za-z0-9_.\-]{20,512}` is copied from comms' WhatsApp client, not from Meta, and should say so.

### C.2 Internal inconsistencies

| # | Where | Problem | Fix |
|---|---|---|---|
| I-1 | 8 vs 5.3, 13 | **Contradiction.** The error table says `2207003` "one retry, same container", `2207032` "one retry, then new", `2207008` "1 to 2 retries, then new container", while the footer, 5.3 and 13 say CREATE is resolve-only and never creates a second container for one `req_` | Make the Kind column step-scoped: retries only on `MEDIA_CONTAINER_STATUS` (a read) and on `MEDIA_PUBLISH` of the **same** `creation_id`. Every "then new container" becomes `FAILED CONTAINER_FAILED`; a new container needs a new `request_id` |
| I-2 | 6 vs 11 | "Poll once a minute for at most 5 minutes" exceeds Claude Code's 2-minute backgrounding, yet 11 says a Reel ends `IN_FLIGHT` "well before that" | State the in-request poll budget (for example 90 s) that returns `IN_FLIGHT`; keep "once a minute for 5 minutes" as Meta's advice, not the executor's budget |
| I-3 | 3 vs 5 | The module layout has no home for `tag_list`, `publish_preview`, `publish_quota`, `media_list`, `media_get`, `profile_get`, `whoami`, nor for the 23 `ToolSpec` entries | Add `media.py`, put tags beside comments, put preview, quota and the ledger in `publish.py`, and name the catalog module (`src/comms/mcp/tools/instagram.py`) |
| I-4 | 5.1 vs 4.1, 9 | `conversation_messages` takes a conversation id, which is an identity with no ref | The tool takes an `rcp_`; comms resolves the conversation internally |
| I-5 | 5.2 vs 4.1 | `message_send` names no recipient argument; `dst_` is introduced and never used | Recipient is `rcp_`; drop `dst_` |
| I-6 | 0 vs 5.2, D-I3 | Section 0 says the flag is on "public-posting" tools; seven are flagged including `comment_delete` and `message_send` | "public-posting and irreversible", as D-I3 says |
| I-7 | 5.1 vs 15 Q3 | `preview_digest` is returned in 5.1 but still an open question | Mark "(proposed, Q3)" or resolve Q3 |
| I-8 | 13 vs body | GI-2, GI-7, GI-8 are referenced only in 13 and 14 | Add 🧪 GI-2 to 4.4, GI-7 to the dual-era sentence, GI-8 to the `conversation_messages` row in place of ✅ |
| I-9 | 7 vs GI-4 | GI-4 tests `media_product_type`, which 7 says never to request | Drop it and `is_comment_enabled` (no restriction) from GI-4 |
| I-10 | D-I8 vs 7, 5.2 | D-I8 says 24 hours; 7 says 24 hours or 7 days after a Click-to-Direct ad | Live check enforces 24 h and reports the 7-day case as `WINDOW_CLOSED`, because without webhooks comms cannot see the ad origin. Mark ℹ️ |
| I-11 | 5.3 vs 6 | Only `publish_reel` "may return `IN_FLIGHT`"; carousel child types are unstated | State child types; if video children are allowed, `publish_carousel` may end `IN_FLIGHT` too |
| I-12 | 13 vs 14 | GI-8's write side ("a send inside the window") is assigned to no milestone | Add "GI-8 (write side)" to MI-3 |
| I-13 | 13 | GI-7 says "`legacy` and `auto`", the host matrix also has "unset" | Add "unset" |
| I-14 | 3 vs 8 | `GraphTransportError("not_sent")` is never mapped in 8 | Add `not_sent` → `FAILED_TRANSIENT` (as `classify.py` does for WhatsApp) |
| I-15 | 5 vs 5.1 | "Every tool takes `account`" but `account_list` lists all aliases | Exempt it |
| I-16 | References | Dropped: MCP *Versioning and Compatibility* (source of the era claims) and *Transports*. Listed but uncited: *Security Best Practices*, *Permissions Reference* | Restore Versioning; drop or cite the other two |

Passed: 23 tools (14 reads, 9 writes); every write has a semantics entry; every capability in the tool tables is in 5.3; the nine ask-list tools match section 11; every D-I resolves; the 21 error rows match the gauntlet; `IN_FLIGHT` is consistent across D-I7, 6 and 5.3 apart from I-2 and I-11.

### C.3 Hygiene

- Unmarked assertions among marked neighbours: the publish, reply and send endpoint cells in 5.2 (all gauntleted ✅), the `whoami` row, the whole of section 10 (webhook field names, `subscribed_apps`, `entry[].messaging[]`: Meta facts with no gauntlet row, mark ℹ️ or defer to a v2 gate), and "`mcp-types` 2.2.0 is dual-era" in 13 (now verified in section A of this report).
- Undefined before use: "standing", `req_` (not in 4.1's prefix list), the schema DSL in 5.2, "the three other actors", and A49 and R-IG0 in the header before D-I12.
- Casing: `<ig_id>` in 5 versus `<IG_ID>` in 2 and 10.
- Harvard: split the PyPI entry into two works; point the GAUNTLET reference at the file and commit `cdae96e`; pick one date convention (the gauntlet has page stamps for every Meta page); order by author.
- Repetition worth trimming: the `/me` call (four places), "never fetched" (four), N+1 (four), the preview description (two).

---

## D. Sourcing

- comms source read at `3f6df9a` (main) and `d9b6aa9` (the spec branch) in `/home/user/telegram-mcp`.
- SDK facts from the wheels `mcp-2.2.0` and `mcp_types-2.2.0` on PyPI, run inside comms' locked environment (`uv sync --locked`, exit 0).
- No Meta or Claude Code page was re-fetched: those facts carry the first-hand verification in `GAUNTLET-v0.5.md` from earlier the same day.
