---
url: https://github.com/vercel-labs/screen-relay
date_fetched: 2026-11-20
---

# screen-relay

Desktop streaming and control for humans and AI agents: watch a desktop in a browser, and click, type, and take screenshots from the shell.

<p>
  <a href="https://vercel.com/labs#active-experiments"><img alt="Vercel Labs Experiment" src="https://img.shields.io/badge/LABS-EXPERIMENT-0a0a0a.svg?style=for-the-badge&amp;logo=Vercel&amp;labelColor=000000" height="28"></a>
  <a href="https://www.npmjs.com/package/screen-relay"><img alt="npm version: screen-relay" src="https://img.shields.io/npm/v/screen-relay.svg?style=for-the-badge&amp;labelColor=000000" height="28"></a>
  <a href="LICENSE"><img alt="License: Apache-2.0" src="https://img.shields.io/badge/LICENSE-APACHE--2.0-97ca00.svg?style=for-the-badge&amp;labelColor=000000" height="28"></a>
</p>

screen-relay starts a desktop on a Linux VM, sandbox, or container, or attaches to the display of a Mac or the signed-in desktop of a Windows PC, and streams it to a browser as H.264. An agent controls the same desktop one command at a time: it takes a screenshot, clicks or types, and takes another screenshot to check the result. The `screen-relay` CLI, a Node SDK, and an HTTP and WebSocket API all reach the same session.

## Install

```bash
npm install -g screen-relay
screen-relay install            # system packages and the engine build (first time only)
```

- **Requirements:** Node 22 or newer. On Linux: Debian, Ubuntu, Fedora, Arch Linux, or Alpine. On macOS: Homebrew and the Xcode Command Line Tools. On Windows: Windows 10 or later on x64, with a user signed in at the PC's screen.
- **Scripts and agents:** `screen-relay install --yes` skips the confirmation. At a terminal, `install` asks before it installs packages.
- **Wayland:** `screen-relay install --backend wayland`. The default backend on Linux is X11.
- **Client only:** the npm install alone is enough to control a session on another machine.

Requirements, backends, and Windows are covered in [Getting started](https://screen-relay.dev/docs/getting-started).

## Quick start

```bash
screen-relay up -d                    # start a display, a desktop, and the engine
screen-relay run -- xfce4-terminal    # open an app on the desktop
screen-relay screenshot screen.png    # see the screen
screen-relay click 0.5 0.5            # coordinates are fractions of the screen
screen-relay type "echo hello"
screen-relay key Enter
screen-relay down
```

- `up` prints a viewer link. Open it in a browser with H.264 WebCodecs: current Chrome, Edge, or Safari.
- The engine listens on `127.0.0.1:8080`. From another machine, forward the port with `ssh -N -L 8080:127.0.0.1:8080 user@your-vm`, or see [Remote access](https://screen-relay.dev/docs/remote-access).
- On a machine that is not set up yet, `up` runs the same setup as `install` first.
- A successful input command means the engine applied the input, not that the app reacted. Take a screenshot to check.

The full walkthrough is in [Getting started](https://screen-relay.dev/docs/getting-started).

## How it works

`screen-relay up` starts a session: a virtual display, a desktop, and the engine. On Linux the display is X11 with XFCE or headless Wayland with sway. On a Mac the engine attaches to the display you are logged in to. On Windows it attaches to the desktop of the signed-in user. The engine captures the screen, encodes it as H.264, and serves the viewer, the stream, and the input API on one port. It lowers the bitrate on a slow link. See [How streaming works](https://screen-relay.dev/docs/how-streaming-works).

A session listens on loopback with no token by default. Starting it with `--public` or on another address creates a token that every remote request must send. There is one input owner at a time. A viewer that opens when input is idle gets control. If another viewer or an SDK session owns input, the new viewer opens view-only. Choose **Take control** to transfer input to your viewer, or **Release control** to keep watching without owning input. Screenshots still work while another controller holds input.

## For AI agents

The CLI ships skills that match its version:

```bash
screen-relay skills get core           # the control loop, coordinates, type versus key, and errors
screen-relay skills get core --full    # adds every reference: CLI, config, SDK, macOS, Wayland, troubleshooting
screen-relay skills list               # specialized skills, such as vercel-sandbox
```

Or add this to your `AGENTS.md`:

```markdown
Use `screen-relay` to see and control a desktop. Before the first use, run
`screen-relay skills get core` and follow it.
```

Every command prints JSON with `--json`. An error carries a `reason` and, when there is one, the next command to run.

## Commands

Run `screen-relay --help` for the full list, or `screen-relay <command> --help` for one command.

| Group | Commands | Docs |
|---|---|---|
| Session | `up`, `down`, `status`, `logs`, `run` | [CLI reference](https://screen-relay.dev/docs/cli) |
| Control | `screenshot`, `click`, `move`, `scroll`, `key`, `type`, `input` | [Client commands](https://screen-relay.dev/docs/cli#client-commands) |
| Setup | `install`, `doctor` | [Getting started](https://screen-relay.dev/docs/getting-started), [Troubleshooting](https://screen-relay.dev/docs/troubleshooting) |
| Agents | `skills`, `help` | [Agent skills](https://screen-relay.dev/docs/agent-skills) |

Flags, environment variables, and `config.yaml` are documented in [Configuration](https://screen-relay.dev/docs/configuration).

## Platforms

The engine runs on Linux (x64 and ARM64, X11 and Wayland), on macOS 12.3 or later (Apple silicon and Intel), and on Windows 10 or later (x64). `screen-relay install` builds it on your machine. See [Linux](https://screen-relay.dev/docs/linux), [macOS](https://screen-relay.dev/docs/macos), and [Windows](https://screen-relay.dev/docs/windows).

## Development

Building from source requires Node.js 22 or later, a C++17 compiler, `make`, and the libraries that `screen-relay install` installs:

```bash
npm ci
npm run build                   # the CLI, the SDK, and the viewer
make                            # the engine, for the default backend
npm test
```

Run the engine tests with `make test`, which also needs the FFmpeg development libraries.

[CONTRIBUTING.md](CONTRIBUTING.md) covers the source layout, the engine, and the tests. [AGENTS.md](AGENTS.md) covers the writing rules and the release process.

## Documentation

- [screen-relay.dev](https://screen-relay.dev): introduction, guides, and reference
- [CLI reference](https://screen-relay.dev/docs/cli) and [Configuration](https://screen-relay.dev/docs/configuration)
- [Node SDK](https://screen-relay.dev/docs/sdk) and [HTTP API](https://screen-relay.dev/docs/protocol)
- [Remote access](https://screen-relay.dev/docs/remote-access): tokens, TLS, and tunnels
- [Docker](https://screen-relay.dev/docs/docker) and [Vercel Sandbox](https://screen-relay.dev/docs/vercel-sandbox)
- [Changelog](CHANGELOG.md)

## License

Apache-2.0. The engine links x264, which is licensed under GPL-2.0-or-later, from your system when `screen-relay install` builds it. See [THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md).
