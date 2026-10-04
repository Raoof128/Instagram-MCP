# Security policy

The Instagram actor holds account tokens and reads other people's comments and DMs. A
vulnerability here can expose private messages or let a model act on an account it should not.
Please treat any finding as a privacy incident.

## Reporting a vulnerability

**Please do not open a public issue for a security problem.**

Report privately through
[GitHub Security Advisories](https://github.com/Raoof128/Instagram-MCP/security/advisories/new).
If the problem is in the implementation rather than the design, you may report it on the comms
repository's advisories instead:
[Raoof128/telegram-mcp](https://github.com/Raoof128/telegram-mcp/security/advisories/new).

If neither is available to you, open a public issue that says only "security report, requesting
private contact". A private channel will be arranged.

Please include:

- the boundary you believe is broken, in terms of the rules below;
- a minimal reproduction (a failing test is ideal);
- what an attacker gains, concretely;
- any limits on exploitability, such as local access or a prompt-injected model.

**Expected response:** an acknowledgement within 7 days and an assessment within 14. A fix for
a confirmed issue lands with a regression test, and you are credited unless you prefer not to be.

**Never include a real access token in a report,** even an expired one. Describe it or redact it.

## Scope

### In scope

- Any path by which an access token leaves the daemon's secret store: a URL, a log, an error
  string, an MCP response, an audit record or a file.
- Any way to reach a host other than `graph.instagram.com` through the Graph client, or to make
  comms fetch a model-supplied URL.
- Any way a caption, comment or DM becomes authority: choosing a tool, picking an account,
  widening a read, or reaching a write.
- A write on an account other than the one named, or a write past that account's `writes` or
  `dms` ceiling.
- A ref from one account that resolves inside another.
- A repeated `request_id` that repeats an effect, or a recorded outcome that misreports what Meta
  did.
- A provider id or token appearing in a tool result, the audit chain or a log.
- A DM sent outside the 24-hour window.

### Out of scope

- Compromise of the owner's own machine or user account.
- Vulnerabilities in Instagram, the Graph API, the MCP host or a model provider. Please report
  those to the vendor.
- Data the owner deliberately lets the model read: allowing the read tools discloses comments and
  DMs to the model provider by design.
- Denial of service against your own local daemon.

## The rules a report should aim at

| Rule | Broken if you can... |
|---|---|
| Token containment | read a token from anywhere but the daemon's secret store |
| Pinned origin | make the Graph client talk to any other host |
| Untrusted content | make retrieved text select a tool, an account or a write |
| Named account | write to an account the call did not name, or past its ceiling |
| Ref isolation | use one account's ref under another account |
| Replay safety | make one `request_id` cause two effects |
| Honest outcomes | get `SUCCEEDED` for an effect that did not happen, or the reverse |

## Supported versions

The design is a proposed amendment (A49) and the implementation is pre-release. Only the latest
commit of `comms-instagram-spec` in comms receives fixes.

## Handling your own tokens

- Add tokens only at the hidden prompt: `comms transport instagram account add <alias>`. Never
  pass a token as an argument or an environment variable, or keep it in a file.
- Generate each token with only the scopes that account's policy needs.
- Start accounts with `writes: false` and `dms: false`, and turn them on deliberately.
- If a token may have leaked, remove the account
  (`comms transport instagram account remove <alias>`) and revoke the token in the Instagram app.
