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

PENDING_SECTION_C

---

## D. Sourcing

- comms source read at `3f6df9a` (main) and `d9b6aa9` (the spec branch) in `/home/user/telegram-mcp`.
- SDK facts from the wheels `mcp-2.2.0` and `mcp_types-2.2.0` on PyPI, run inside comms' locked environment (`uv sync --locked`, exit 0).
- No Meta or Claude Code page was re-fetched: those facts carry the first-hand verification in `GAUNTLET-v0.5.md` from earlier the same day.
