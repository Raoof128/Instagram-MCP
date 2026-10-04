# Contributing

Thanks for your interest. This repository holds the design of the Instagram actor; the code lives
in [comms](https://github.com/Raoof128/telegram-mcp). Where a change belongs decides how to make
it.

## Where a change goes

| You want to... | Go to |
|---|---|
| Fix a bug or add a feature in the code | comms, branched from `main`, under `src/comms/transports/instagram/` |
| Correct a fact about Meta, MCP or Claude Code | this repository: an issue citing the primary source |
| Propose a change to the design | this repository: an issue first, then a pull request to `docs/SPEC.md` |
| Report a vulnerability | privately, as described in [`SECURITY.md`](SECURITY.md) |

## Proposing a design change

1. **Open an issue first** and describe the problem before the solution, so the approach can be
   agreed.
2. **Cite a primary source** for every external claim: Meta's developer documentation, the MCP
   specification, or the Claude Code documentation. Use Harvard (Australian) style, as the
   specification's reference list does.
3. **Edit the canonical copy.** `docs/SPEC.md` mirrors `docs/instagram-spec-v0.6.md` in comms.
   A change lands there first, as a ruling in comms' rulings register, and the mirror follows.
4. **Keep the legend honest.** Mark each claim ✅ (verified), 🔧 (corrected), 🧪 (needs a live
   test) or ℹ️ (inference), as the specification does.

## Changing the code

The comms working agreement applies in full: its `CLAUDE.md` lists the gate. In short:

- **Write the failing test first** and watch it fail.
- **Run the whole gate before you push:** the suite, the end-to-end smoke, the formal models,
  `ruff`, `mypy` and the build.
- **Record any deviation from the plan** as a ruling (R-IG*n*) in
  `docs/verification/comms-v0.3-rulings.md`.
- **Append a dated entry** to comms' `AGENT.md` and `CHANGELOG.md`.

## Never commit secrets

Never commit a real access token, an app secret, a session file, a database or a key, and never
paste one into an issue or a pull request. Test tokens must be obviously fake, such as
`IGAAcanaryTOKEN...`. If you commit a secret by mistake, revoke it first, then tell the
maintainer privately.

## Style

- Plain Australian English, short sentences, and one idea per sentence.
- Name the decision (D-I*n*), ruling (R-IG*n*) or gate (GI-*n*) a change relies on.

## Code of conduct

This project follows the [Contributor Covenant](CODE_OF_CONDUCT.md). Be kind and assume good
faith.
