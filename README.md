# Bifröst — Reference Implementation

Business Central extensions showing how to add message types to
[Bifröst](https://github.com/OrigoSoftwareSolutions/bc-origo-bifrost-core), how to
retrofit an existing solution, and how to call one - from outside BC or from inside it. No
customer or internal implementation code anywhere in this repo - it exists purely to teach
the patterns.

## Start here

| Your situation | Read |
|---|---|
| **Giving this to a coding agent** | [`AGENTS.md`](AGENTS.md) - one dense file covering both build-from-scratch and retrofit, written for an agent to act on directly |
| **First time here, reading it yourself** | [`QUICKSTART.md`](QUICKSTART.md) - clone, compile, call one message type, end to end |
| **Building a new message type from scratch** | [`EXTENDING.md`](EXTENDING.md) |
| **Writing help text an agent can actually use** | [`WRITING-HELP.md`](WRITING-HELP.md) - the help markdown is the only schema most message types get |
| **You already have an extension and want to add Bifröst support** | [`ADAPTING.md`](ADAPTING.md) |
| **Calling Bifröst from outside Business Central** | [`INTEGRATING.md`](INTEGRATING.md) |
| **Calling a message type from AL code already inside Business Central** | [`CALLING-FROM-AL.md`](CALLING-FROM-AL.md) |

## The three apps

| App | Shows |
|---|---|
| `Bifrost Reference` | Three message types built from nothing (no-fail read, read with an expected error, write with isolation and an unexpected error), the app's `Help.Reference.Get` directory type, and its registration with Foundation's app registry. See `EXTENDING.md`. |
| `Legacy App` | Ordinary business logic with no awareness of Bifröst - the "before" state for the retrofit example. |
| `Legacy App - Bifrost` | Depends on `Legacy App` and Bifröst Foundation, and wraps two of `Legacy App`'s procedures - one needs nothing but a thin wrapper, the other needed a precise refactor first. See `ADAPTING.md`. |

`Legacy App` and `Legacy App - Bifrost` were added in separate commits on purpose -
`git log` on `Legacy App/src/LegacyStockMgt.Codeunit.al` shows the exact retrofit diff
`ADAPTING.md` describes, rather than asking you to trust the prose.

## Object ID ranges

All illustrative only - **never reuse these** for a real extension:

| App | Range |
|---|---|
| `Bifrost Reference` | `90000–90049` |
| `Legacy App` | `90050–90099` |
| `Legacy App - Bifrost` | `90100–90149` |

Register your own: `50000–99999` for a per-tenant extension, or a range from Microsoft via
Partner Center for an AppSource app.

## What this repository deliberately leaves out — for now

It is a sample of message types first. Of the things every dependent app does (see
[Build on Bifröst](https://businesscentralal.github.io/bifrost/en-us/extensibility/)), these
are **not yet** shown here and are being added one at a time:

- a setup page hung off the Bifröst Setup page (one action in the *Apps* group) and a
  registered secret in Foundation's secret store;
- an install codeunit with the take-over shape;
- a test app — one test per message type through `Dispatcher ori`, one asserting the exact
  error text the help promises;
- permission sets (a read/full pair per app).

The affix and namespace are still Origo's (`Origo.Bifrost.Reference`, no object-name affix);
they will change to a partner's so that nothing here can be copied into your app by accident.
Until then: **use your own affix, namespace and object range** — never these.

## Building it

CI (`.github/workflows/CICD.yaml`, `PullRequestHandler.yaml`) uses the standard
[Microsoft AL-Go for GitHub](https://github.com/microsoft/AL-Go) actions and compiles all
three projects on every push and PR. Because Bifröst Foundation's source is a private
repository, CI needs a `GH_TOKEN_DEPS` secret (read access to
`OrigoSoftwareSolutions/bc-origo-bifrost-core`) configured on this repo — see
`.AL-Go/settings.json`. **A fork outside Origo cannot supply that secret**, so for a partner
the supported route is a local build: open a project folder in VS Code with the AL extension,
download symbols from a Business Central 28 environment that has Bifrost Foundation installed,
and build (`Ctrl+Shift+B`). Foundation's symbols will be published where a partner can fetch
them without a secret; until then, ask Origo.

## Extension model

- `interface "Msg Interface ori"` — implement this; six methods, the *interface itself*
  carries no `Access = Internal` (an interface's own methods are the public contract). Implementing
  codeunits commonly are `Access = Internal` — see `RefNoteAddImpl.Codeunit.al` — since the interface,
  not the codeunit directly, is how a message type is invoked.
- `enum "Message Type ori"` — `Extensible = true`; add your value via an enum extension.
- `table "Message Argument ori"` — the request/response carrier every implementation reads from and writes to.
- `codeunit "Dispatcher ori"` — the public, in-process entry point; see `CALLING-FROM-AL.md`.

This is the same pattern Origo's own dependent apps (Storage, Iceland, Arionbanki,
Landsbankinn, DocEx) use — nothing about the extension model is special-cased for Origo.

## What NOT to copy

If you've seen Origo's own Bifröst extensions, copy the *pattern* shown here, not their
*content* — those call real external systems (banking APIs, government registries, document
exchange vendors) under commercial agreements that don't transfer to you. Everything in this
repo is deliberately synthetic, with zero external dependency.

