# flab

A Frida-based analysis workspace for building reusable, game-specific "Instruments" (hooks, memory probes, and controls) against processes you own, with a human TUI and machine interfaces (CLI, NDJSON, ACP, MCP) sharing one engine.

**Status:** Prototype — actively developed personal tooling; interfaces and the Instrument API still change between commits.

## Features

- Guided Ink TUI with a three-step flow: **Connect → Analyze → Instrument**
- Device selection for local, USB/mobile, remote, and explicit `frida-server` endpoints
- Reusable agent library (`agent/lib/`) for hooks, memory search/watch, Stalker, exception handling, symbolication, and Mono / IL2CPP / Unreal / Java / ObjC runtimes
- Per-game targets declared once with `defineInstrument()`; flab generates RPC handlers and `__describe()` metadata for every interface
- **Record**: bounded capture of function entry/exit timing, arguments, returns, and backtraces under `artifacts/records/`
- Agent access: `flab agent --json` (control API), `flab acp` (ACP gateway), `flab mcp` (MCP server), and a per-session NDJSON protocol (`flab.ndjson.v1`)
- `flab depcheck` enforces that targets import reusable code only from `agent/lib/`
- Target lifecycle commands: scaffold, register, rename, build, package, delete

## How it works

- `src/core/` is the single engine: session lifecycle (`session.ts`, `lifecycle.ts`), agent compilation (`compile.ts`), device/process discovery (`devices.ts`, `processes.ts`), Instrument and project management (`instruments.ts`, `projects.ts`), Record storage (`records.ts`), and the ACP/MCP/NDJSON surfaces (`acp.ts`, `mcp.ts`, `protocol.ts`, `control.ts`).
- `src/tui/` (Ink + React) and `src/cli/` are thin frontends over `src/core/index.ts`; they do not talk to Frida directly.
- The Frida agent lives in `agent/`. `agent/lib/` is side-effect-free at import time and is compiled with `frida-compile` together with a target entry from `agent/targets/<game>/index.ts`.
- Targets are registered in `frida-labs.json` (process name, attach/spawn mode, entry file, optional device).
- Running bare `flab` on a TTY opens the TUI; set `FLAB_NO_TUI=1` to force plain CLI dispatch.

## Tech stack

- Runtime/Language: Bun, TypeScript
- Framework: Ink 7 + React 19 (TUI), `@modelcontextprotocol/server` (MCP)
- Instrumentation: `frida` 17, `frida-compile`, `frida-java-bridge`, `frida-objc-bridge`, `frida-il2cpp-bridge`
- Tooling: `tsc` typecheck, `bun test`, `bun build --compile` for a standalone binary

## Getting started

### Requirements

- Bun
- A Frida-reachable target: a local process, or a device running `frida-server` (USB/remote)
- Only use flab on processes and devices you own or are authorized to test

### Install

```sh
bun install
```

### Run

```sh
bun run flab                          # guided TUI
bun run flab -- --help                # grouped CLI overview
bun run flab -- devices               # list reachable Frida devices
bun run flab -- processes Terraria    # search a running process (add --device usb|remote|<id> or --host host:port)
bun run flab -- probe --pid 1234      # generic quick analysis
bun run flab -- run terraria          # saved target with its control dashboard
bun run flab -- run terraria --console
bun run flab -- new terraria "Terraria.exe"
bun run flab -- targets
bun run flab -- build terraria
bun run flab -- package terraria
bun run flab -- doctor
```

Agent interfaces:

```sh
bun run flab -- agent --json            # control API over NDJSON
bun run flab -- acp                     # ACP gateway (auto-detects an upstream coding agent)
bun run flab -- mcp                     # MCP tool server (flab_control, flab_events)
bun run flab -- capabilities --json     # live-session protocol contract
```

Development:

```sh
bun run typecheck
bun test
bun run test:runtime        # native Frida runtime checks
bun run depcheck
bun run build-bin           # produces ./flab
```

Detailed guides: [Agent Control API](docs/agent-control-api.md), [AI harness game-mod guide](docs/agent-game-mod-guide.md), [analysis playbooks](docs/analysis-playbooks.md), [Android live-verification report](docs/android-live-verification.md), [Windows Steam E2E report](docs/windows-steam-e2e-report.md).

## Project structure

```text
agent/lib/            reusable Frida agent library (hook, mem, watch, stalker, mono/, ue/, il2cpp, java, objc, recording, instrument)
agent/targets/<game>/ game-specific Instruments (index.ts + README.md) and a generic _probe
src/core/             session, device, Instrument, Record, protocol, ACP/MCP engine
src/tui/              Ink TUI: workbench, explorer, repl, observe, instrument, record panels
src/cli/              argument parsing and command dispatch over the core
tests/                bun unit/protocol/TUI tests, runtime-attach and Windows E2E harnesses
docs/                 protocol and workflow guides
artifacts/            QA transcripts and Record output (runtime output is git-ignored)
frida-labs.json       target registry
.mcp.json, .codex/, .claude/, .agents/, .opencode/, .gjc/  harness-specific entry points for the build-game-mod skill
```

## Configuration

- `FLAB_NO_TUI` — when set, bare `flab` never opens the TUI (for CI / automation that owns a TTY).
- `FLAB_WORKSPACE` — workspace root used when flab is launched as an ACP/MCP subprocess.
- `frida-labs.json` — target registry (process name, mode, entry, optional device id).

## Safety boundary

Use flab only on a process and device you own or are authorized to test, in owned-offline mode or an isolated private test lab. Do not bypass anti-cheat, enter public matchmaking or production economies, or affect another user's process, account, or data.

## License

No license file yet.
