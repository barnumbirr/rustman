# Rustman

A free and open source API client built in Rust with iced. Native, no webview.

I got tired of API clients that want an account, eat hundreds of megabytes, and ship a whole browser just to send an HTTP request. So I built my own. It is small, it is fast, and nothing you do leaves your machine.

**[Download](https://github.com/rust-man-org/rustman/releases/latest)** · **[Website](https://rust-man-org.github.io/rustman/)** · **[Discord](https://discord.gg/vfa8rKYzKG)**

---

## Free, and staying that way

Rustman is free and open source and it is going to stay that way.

- No paid plans, no pro tier, no subscriptions, ever.
- No account. You just open it and use it.
- No telemetry, no analytics, nothing phones home.
- Your collections, history, and environments live on your machine and nowhere else.

The app makes zero automatic network calls. The only update traffic that
can ever happen is the GitHub releases check, which is off by default —
turn it on in Settings → Updates ("Check for updates on launch"), or use
the manual Check button any time. Everything else is strictly on-demand:
your requests, plus anything you explicitly sync or download.

This is not a startup and I am not trying to sell you anything. It is a tool I wanted for myself, so I built it and put it out for everyone.

---

## Why Rustman

| | Rustman | Postman | Bruno |
|---|---|---|---|
| Price | Free forever | Freemium / $14mo | Free |
| Install size | ~30 MB | ~350 MB | ~100 MB |
| No account | Yes | No | Yes |
| Zero telemetry | Yes | No | Yes |
| Native GUI (no webview) | Yes | No | No |
| Native HTTP (no CORS) | Yes | No | No |
| WebSocket support | Yes | Yes | No |
| Git for collections | Yes (local + remote) | No | Yes |
| Command palette | Yes | Yes | Yes |
| Pre/post scripts | Yes (own Rust language, not JS) | Yes | Yes |
| Large response viewer | Yes | No | No |

## Features

- **Pure native GUI.** Built with [iced](https://github.com/iced-rs/iced). No webview, no Electron, no JavaScript runtime.
- **Portable Linux build.** No WebKitGTK, GTK or pdfium dependency, so one binary runs on Debian, Arch, NixOS, immutable distros and minimal systems. HTML responses render as readable text.
- **Native Rust HTTP engine.** Requests go through reqwest, so no browser CORS limits and no response size caps.
- **WebSocket.** Type a ws:// or wss:// URL and the whole panel flips into WebSocket mode. Connect, then send and receive in real time on a live feed.
- **Code editor for body and response.** A syntax highlighted, line numbered editor (the rust-man-org fork of iced-code-editor, backed by syntect) for the request body and the response, with full text selection.
- **File upload.** multipart/form-data with a real file picker. The file's bytes go to the server with a Content-Type guessed from the extension.
- **All common auth types.** Bearer, Basic, API Key (header or query), Cookie, and JWT (HS256).
- **Scripting.** Pre-request and test scripts run on a custom Rust scripting language — see below.
- **Environment variables.** `{{variable}}` substitution from the active environment, applied to the URL, headers, query params, and the JSON, Text or GraphQL body.
- **Collections and history.** Organize requests and replay them from history. Local SQLite is the source of truth.
- **Git for collections.** A built in Source Control panel. Commit collections (they are stored as JSON), browse the log, restore any commit back into the app, make and switch branches, see the working diff, and juggle several repos. Clone, fetch, pull, and push go through your system git, so they use the SSH keys and logins you already have. SQLite stays the source of truth and git sits on top.
- **Import.** cURL commands (native Rust tokenizer), Postman v2 and v2.1 collections, and OpenAPI specs. The OpenAPI parser reads JSON or YAML, but the file picker only offers `.json`, so a `.yaml` spec cannot be chosen through the dialog yet.
- **Export.** Turn any request into a cURL command, or export a collection as Postman v2.1 JSON.
- **Command palette.** Keyboard driven access to everything.
- **Multiple request tabs.** Work on a few requests at once.
- **Session persistence.** Your open tabs come back next launch.
- **Self update.** A built in updater (self-replace) grabs and swaps in new releases.
- **Global default timeout.** One Settings-panel value applied to every request, persisted across restarts.

## Scripting

![Rustman Scripting](docs/rustman-scripting.png)

Pre-request and test scripts run on [**rustman-engine**](https://github.com/rust-man-org/rustman-engine), a small scripting language built from scratch in Rust for this app — not JavaScript, no `pm.*` API, no embedded runtime. A test script's assertions land in a **Tests** tab with a selectable, copyable **Console** for anything you `print(...)`. A **Global Scripts** pair (Settings panel) runs before every request's own script and can be overridden by it, so common setup doesn't need copy-pasting into every request.

You get `env`/`set_env`, `header`/`headers`/`set_header`, `cookie`, `body`/`set_body`, `url`, `test`, `print`, `contains`, base64, `jwt_decode`, `json_parse`/`json_stringify`, and AES-256-GCM. `contains` covers the keyword checks you actually reach for:

```
test("no stack trace leaked", !contains(response.text(), "panicked at"))
test("user is an admin", contains(response.json().roles, "admin"))
```

It is a substring test on a string, a membership test on an array, and a key test on an object, and `haystack.contains(needle)` is the same function spelled the other way.

Full grammar, built-ins, and a copy-pasteable **LLM prompt** that writes scripts for you: [rust-man-org.github.io/rustman/scripting.html](https://rust-man-org.github.io/rustman/scripting.html).

## What works and what does not

I would rather be straight with you than oversell. A few things are half done right now.

- **Environment variables use the active environment only.** Substitution pulls from the one active environment and hits the URL, headers, query params, and the JSON, Text or GraphQL body. It does not touch auth fields (tokens, keys, passwords) or form-data fields. There are no collection, global, or dynamic (`{{$guid}}`) variables, and it resolves in a single non-recursive pass. With no environment active, `{{var}}` goes out as written.
- **Git commits are manual, and the store is two way.** No commit on save yet, but the git store does read back into the app. Restore loads a commit's collections over your current state after it asks you. Clone, fetch, pull, and push shell out to your system git. SQLite is still the source of truth.

## Download

Grab the latest binary from the [releases page](https://github.com/rust-man-org/rustman/releases/latest).

| Platform | File |
|---|---|
| macOS (Apple Silicon + Intel) | `Rustman-macos-universal.dmg` |
| Windows x86_64 | `rustman-windows-x86_64.zip` |
| Linux x86_64 | `rustman-linux-x86_64.tar.gz` |
| Linux arm64 | `rustman-linux-aarch64.tar.gz` |

The Linux build has no WebKitGTK, GTK or PDF runtime dependency, so the same
binary runs on Debian, Ubuntu, Arch, NixOS, immutable distros and minimal
systems, on both x86_64 and arm64. HTML responses are shown as readable text
rather than a rendered page, and PDFs show a binary-response summary.

## Build from source

You will need a few things first.

- A recent [Rust toolchain](https://www.rust-lang.org/tools/install), 1.85 or newer. Rustman is edition 2024, as is the iced-code-editor fork it builds against, so older toolchains will not build it. The latest stable Rust works fine.
- A C toolchain and CMake. git2 builds with vendored-libgit2 and vendored-openssl, so libgit2 and OpenSSL compile from source, and rusqlite bundles SQLite. You need a C compiler and cmake on the machine.
- A few system libraries on Linux, listed below.

Two dependencies (the scripting engine and the editor fork) come from
`rust-man-org` repos on GitHub and are fetched by cargo automatically —
there is nothing extra to clone.

```sh
git clone https://github.com/rust-man-org/rustman
cd rustman
cargo build --release
```

The binary ends up at `target/release/rustman`. The longer guide lives in [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md).

Package managers should build with `--no-default-features`:

```sh
cargo build --release --no-default-features
```

That compiles out the in-app updater entirely — no launch check, no
Settings → Updates card, no update banner, none of the crates that swap the
running binary — so the package manager stays the only thing that touches the
installed executable. The default build keeps the updater.

There is no WebKitGTK, GTK or pdfium build dependency: HTML responses are
rendered as readable text.

### Linux system dependencies

```sh
sudo apt-get install -y \
  libxkbcommon-dev libxi-dev libx11-dev \
  libxcb1-dev libxcb-xkb-dev libdbus-1-dev \
  pkg-config cmake build-essential
```

macOS and Windows want the platform C toolchain (Xcode command line tools or MSVC build tools) plus CMake for the vendored native bits, on top of Rust.

### Development

```sh
cargo run
```

## Tech stack

- **GUI.** [iced](https://github.com/iced-rs/iced) 0.14, pure Rust, drawn with wgpu.
- **HTTP.** reqwest with rustls and multipart.
- **WebSocket.** tokio-tungstenite.
- **Storage.** rusqlite (bundled SQLite), the source of truth.
- **Collection version control.** git2 (vendored libgit2 and OpenSSL) for local commits, plus your system git for remotes.
- **Code editor and highlighting.** Rustman's fork of [iced-code-editor-rustman](https://github.com/rust-man-org/iced-code-editor-rustman) (upstream by LuDog71), pinned to a tag and backed by syntect.
- **Self update.** self-replace.
- **Scripting.** [rustman-engine](https://github.com/rust-man-org/rustman-engine), a small custom scripting language (own crate, not JS/QuickJS) for pre-request/test scripts.

## Documentation

- [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) walks through building from source, the toolchain, the module layout, and how persistence works.
- [rust-man-org.github.io/rustman/scripting.html](https://rust-man-org.github.io/rustman/scripting.html) is the full scripting language reference, with worked examples and an LLM prompt you can copy.
- [rust-man-org.github.io/rustman/#roadmap](https://rust-man-org.github.io/rustman/#roadmap) is what I am building next.

## Thanks

Two things Rustman relies on live in their own small repos under [rust-man-org](https://github.com/rust-man-org), pinned by tag and fetched by cargo automatically.

- **[iced-code-editor](https://github.com/LuDog71FR/iced-code-editor)** by **LuDog71** is the syntax highlighted, line numbered editor behind the request body and the response viewer. Rustman uses its own [iced-code-editor-rustman](https://github.com/rust-man-org/iced-code-editor-rustman) fork of it — MIT licensed, pinned so it can be tweaked without waiting on a release. Thank you for building it.
- **[rustman-engine](https://github.com/rust-man-org/rustman-engine)** is the scripting language behind pre-request/test scripts — not a third-party crate, this one's written for Rustman itself and lives in its own repo.

And the projects this whole thing stands on, [iced](https://github.com/iced-rs/iced), [reqwest](https://github.com/seanmonstar/reqwest), [tokio](https://github.com/tokio-rs/tokio), [syntect](https://github.com/trishume/syntect), [rusqlite](https://github.com/rusqlite/rusqlite), and [git2-rs](https://github.com/rust-lang/git2-rs).

## Come say hi

Got a question, a bug, or an idea? **[Join the Discord](https://discord.gg/vfa8rKYzKG)**.

## License

MIT. See [LICENSE](LICENSE).
