---
name: aikit
description: Multi-agent template package manager and CLI (aikit init, install/list/update/remove, package init/build/publish, release, aikit agent run, aikit serve, check). For Rust/Python integrations use aikit-sdk and aikit-py as the programmatic gateway to the same catalog, deploy, detection, and run/event APIs. Large CLI prompts should use stdin (omit -p) to avoid Linux ARG_MAX.
license: Apache-2.0
---

# AIKIT

**Multi-agent template package manager, CLI, and HTTP gateway:** install
and publish `aikit.toml` packages from GitHub or a local directory, map
templates into many assistant layouts, scaffold with `aikit init`, run
supported agent CLIs with `aikit agent run`, and expose the same runtime
over HTTP with `aikit serve`. The catalog covers **18** assistants;
`aikit agent run` supports `codex`, `claude`, `gemini`, `opencode`,
`cursor`, `pi`, `aikit`. **aikit-sdk** (Rust) and **aikit-py** (Python) mirror
the same gateway for automation (deploy, probe CLIs, buffered run, event
stream / NDJSON shape).

## When to use

- Initializing a project for an AI assistant (Cursor, Claude, etc.).
- Installing, listing, updating, or removing packages.
- Creating a new package (`aikit.toml`, templates).
- Building and publishing packages to GitHub.
- Running coding agents directly from the CLI (with optional streaming
  or machine-readable NDJSON output).
- Serving an agent as a multi-turn HTTP API for other tools.
- Checking git, VS Code, and AI agent CLI availability.

## Installation

```bash
# Linux (GNU)
curl -L https://github.com/goaikit/aikit/releases/latest/download/aikit-x86_64-unknown-linux-gnu.tar.gz | tar xz
sudo mv aikit /usr/local/bin/

# Homebrew (Linux)
brew install goaikit/cli/aikit

# Scoop (Windows)
scoop bucket add goaikit https://github.com/goaikit/scoop-bucket
scoop install aikit
```

Verify: `aikit version`

## Commands

| Command | Description |
|---------|-------------|
| `aikit init [name]` | Initialize a Spec-Driven Development project with AI assistant templates |
| `aikit install <source>` | Install packages from GitHub (`owner/repo`) or local directory |
| `aikit list` | Show installed packages (optional: `--author`, `--detailed`) |
| `aikit update <pkg>` | Update a package to latest (optional: `--breaking`) |
| `aikit remove <pkg>` | Uninstall a package (optional: `--force`) |
| `aikit agent run` | Run a coding agent with a prompt. Common subagent-invocation knobs (spec 013): `-a/--agent`, `-m/--model`, `-p/--prompt`, `--sandbox`, `--auto-approve`, `-C/--cd`, `--add-dir`, `-o/--output-result`, `--output-schema`, `--capabilities`, `--bare`, `--ephemeral`, `--skip-git-repo-check`, `--yolo`, `--stream`, `--events` |
| `aikit agent list` | List project-scoped agent definitions |
| `aikit agent check` | Probe runnable agent CLIs only |
| `aikit agent mcp list` / `add` | Inspect or merge MCP server entries (Cursor, Claude, Gemini, VS Code Copilot, OpenCode, Codex) |
| `aikit serve` | HTTP server for multi-turn agent sessions (SSE or single-shot JSON via `Accept`) |
| `aikit check` | Check git, VS Code, and AI agent CLIs availability |
| `aikit package init <name>` | Create a new package with `aikit.toml` |
| `aikit package validate` | Validate package structure and that templates exist (install-ready) |
| `aikit package build` | Build distributable package (`dist/` or `.genreleases/`) |
| `aikit package publish <owner/repo>` | Publish package to GitHub (release + assets) |
| `aikit release <version>` | Create GitHub release from `.genreleases/` (e.g. `v1.0.0`) |
| `aikit spec` | Export the CLI command surface as JSON, YAML, or Markdown |
| `aikit version` | Show version |

> **Deprecated aliases:** `aikit run`, `aikit agents`, and the top-level
> `aikit mcp ...` still work but print a warning. Always prefer the
> `aikit agent ...` namespace.

## Quick start: use an existing project

```bash
# Initialize current directory for Cursor
aikit init --here --ai cursor

# Or create a new project for Claude
aikit init my-project --ai claude

# Install a package (GitHub owner/repo or local path)
aikit install username/package-name
aikit list
aikit check
```

## Quick start: create and publish a package

```bash
# 1. Create package
aikit package init my-tools --description "My AI commands"

# 2. Edit aikit.toml and add files under templates/
cd my-tools

# 3. Build
aikit package build

# 4. Publish to GitHub (creates release and uploads)
aikit package publish username/my-tools
# Or: push repo first, then aikit release v1.0.0

# 5. Install elsewhere (e.g. for Cursor)
cd /path/to/your-project
aikit install username/my-tools --ai cursor
```

## Configuration

- **GitHub auth**: Set `GITHUB_TOKEN` or `GH_TOKEN` in `.env` or use
  `--token` / `--github-token` on `install`, `init`, or `release`.
- **Package manifest**: Each package has an `aikit.toml` (name, version,
  description). Required for local installs and publish.
- **Server auth**: `aikit serve --api-key <key>` or
  `AIKIT_SERVE_API_KEY`.
- **Logging**: `RUST_LOG` (e.g.
  `RUST_LOG=aikit::serve::run=debug aikit serve`).

Example `.env`:

```bash
GITHUB_TOKEN=your_github_token_here
```

## Run coding agents (`aikit agent run`)

Runnable agents: `codex`, `claude`, `gemini`, `opencode`, `cursor`, `pi`,
`aikit`, `auto`.

```bash
# Run with inline prompt
aikit agent run --agent claude -p "Refactor this function"

# Read prompt from stdin
echo "Add tests" | aikit agent run --agent opencode

# Emit NDJSON events to stdout (one JSON object per line)
aikit agent run --agent claude --events -p "Summarize the project"

# Combine --events and --stream for streaming-aware JSON output
aikit agent run --agent claude --events --stream -p "Refactor this module"
```

### Long prompts (avoid `Argument list too long`)

On Linux, the kernel limits the **total size of the process argument
list and environment** (`getconf ARG_MAX`, often on the order of 128 KiB
to 2 MiB). A very large string passed as a single shell argument to
`aikit` can exceed that limit; the shell then fails before `aikit`
starts, often with `Argument list too long`.

**Do this for long prompts:** omit `-p` and pass the prompt on **stdin**
(aikit reads the full prompt from stdin when `--prompt` is not set).

```bash
# File redirect
aikit agent run -a claude < prompt.txt

# Pipe
cat build.log | aikit agent run -a aikit

# Here-doc (no word-splitting of the body)
aikit agent run -a claude <<'EOF'
Paste or generate arbitrarily long text here.
EOF
```

**Avoid** for large content: inline quotes with megabytes of text, and
`-p "$(cat huge.txt)"` (the shell still expands to one massive `-p`
argument and can hit the same limit).

**NDJSON** (one object per stdout line when using `--events`): `payload`
is one of `json_line`, `raw_line`, `raw_bytes`, `token_usage_line`,
`stream_message`, `quota_exceeded`, or one of the `aikit_*` variants
emitted by the built-in agent. Example:

```json
{"agent_key":"claude","seq":1,"stream":"stdout","payload":{"stream_message":{"text":"Hello","phase":"final","role":"assistant","kind":"message"}}}
{"agent_key":"claude","seq":2,"stream":"stdout","payload":{"token_usage_line":{"usage":{"input_tokens":100,"output_tokens":50}}}}
```

**Key options:**

| Flag | Short | Description | Default |
|------|-------|-------------|---------|
| `--agent` | `-a` | Runnable agent key | **Required** |
| `--model` | `-m` | Model passed to the agent | Agent default |
| `--prompt` | `-p` | Prompt | Reads from stdin if omitted |
| `--sandbox` | | Filesystem-trust policy: `read-only` \| `bounded-write` \| `unrestricted` (spec 013 D1) | Backend default |
| `--auto-approve` | | Auto-approve every tool call without prompting (D1) | `false` |
| `--cd` | `-C` | Working root for the agent (D2); bounds `bounded-write` | Current dir |
| `--add-dir` | | Extra writable root; repeatable (D2) | — |
| `--output-result` | `-o` | Write the agent's final message to `<file>` (D3) | — |
| `--output-schema` | | JSON Schema file the final output must satisfy (D4) | — |
| `--capabilities` | | Print the resolved spec-013 capability matrix for `--agent` and exit (D5) | `false` |
| `--bare` | | Skip user config/hooks/MCP for reproducible runs (D6) | `false` |
| `--ephemeral` | | Do not persist the session (D6) | `false` |
| `--skip-git-repo-check` | | Skip the headless git-repo guard (D6) | `false` |
| `--yolo` | | Auto-confirm, skip checks. Macro for `--sandbox unrestricted --auto-approve` | `false` |
| `--stream` | | Agent-native streaming flags | `false` |
| `--events` | | NDJSON event stream to stdout | `false` |
| `--progress` | | Live human-readable progress on stderr (conflicts with `--events`) | `false` |
| `--resume` | `-r` | Resume the session with the given session ID | |
| `--resume-last` | | Resume the most recent session for the current directory | `false` |
| `--dry-run` | | Validate inputs (and print the resolved matrix) but don't execute the agent | `false` |
| `--debug` | | Verbose diagnostics (global `aikit` flag) | `false` |

`--stream` and `--events` are independent: `--stream` tunes agent argv;
`--events` switches CLI output to NDJSON. Combined use is valid and
recommended for tooling integration.

`aikit agent run` does not read `CODING_AGENT` or `CODING_AGENT_MODEL`;
always pass `-a` (and `-m` only when you want to override the agent's
model).

Run `aikit agent run --help` for the authoritative option reference.

### Trust, approval, and capability negotiation (spec 013)

`--sandbox` is the filesystem **trust boundary** (ADR-0012) — graduated and
vendor-neutral:

- `read-only` — no writes.
- `bounded-write` — writes confined to `-C/--cd` plus any `--add-dir`.
- `unrestricted` — no boundary.

It is separate from `--auto-approve` (whether each tool call prompts). codex
couples approval into its sandbox; the other backends model the two axes
independently.

Each backend declares, per knob, how it can honor a request. Legend:
**OS** = enforced via OS primitives; **App** = cooperative (app-level);
**Emu** = emulated by aikit; **—** = unsupported.

| Knob | codex | claude | gemini | opencode | cursor | pi | aikit |
|---|---|---|---|---|---|---|---|
| `--sandbox` | OS | App | OS | App | — | — | App |
| `--auto-approve` | App | App | App | App | — | — | App |
| `-C/--cd` | OS | OS | OS | OS | OS | OS | OS |
| `--add-dir` | OS | — | OS | — | — | — | App |
| `--output-schema` | OS | OS | Emu | Emu | Emu | Emu | Emu |
| `--bare` | OS | OS | — | — | — | OS | — |
| `--ephemeral` | OS | — | — | — | — | OS | — |

**Fail-closed:** a security knob (`--sandbox`, `--add-dir`) a backend cannot
honor is never silently downgraded — the run exits `3` before spawning.
App-level fidelity is reported honestly, never relabeled OS-enforced; aikit
does not synthesize its own OS sandbox where a backend lacks one.

Pre-flight a backend with `--capabilities` (or `--dry-run`) before committing
work:

```bash
aikit agent run -a cursor --capabilities
aikit agent run -a pi --sandbox read-only --dry-run   # exits 3 — pi has no sandbox
```

### Exit codes (spec 013 D6)

Uniform across backends:

| Code | Meaning |
|---|---|
| `0` | success (terminal `Result` emitted) |
| `2` | invalid invocation |
| `3` | pre-flight failure (unsupported security knob / backend missing / auth) |
| `10` | runtime sandbox violation |
| `124` | timeout |
| `130` / `137` / `143` | cancelled (SIGINT / SIGKILL / SIGTERM) |
| `1` / other | backend failure (pass-through of the agent's own non-zero exit) |

### Result capture (D3 / D4)

- `-o/--output-result <file>` writes the agent's final assistant message to
  `<file>` — derived from the terminal canonical event, so it works even for
  backends whose native CLI has no such flag.
- `--output-schema <file>` constrains the final output to a JSON Schema
  (native on codex/claude; emulated elsewhere).
- In `--events` mode a terminal `Result` event
  (`{"type":"result","text":…,"structured":…,"session_id":…}`) is emitted
  once, so streaming callers get the same handle without a temp file.

## Serve agents over HTTP (`aikit serve`)

`aikit serve` exposes the same agent runtime via a small REST + SSE
surface. Domain endpoints are versioned under `/api/v1/`. Sessions are
created implicitly on the first POST `/api/v1/messages` that omits
`session_id`; subsequent calls resume by passing the returned id.

```bash
# Defaults: 127.0.0.1:8787, 300s timeout, 10 concurrent runs
aikit serve

# Public bind, longer timeout, require an API key
aikit serve --host 0.0.0.0 --port 8787 \
  --run-timeout-secs 600 --max-sessions 20 \
  --api-key "$(openssl rand -hex 32)"
```

**Endpoints:** `GET /healthz` (liveness), `GET /readyz` (readiness),
`GET /api/v1/agents`, `POST /api/v1/messages`, `GET /api/v1/sessions`,
`GET /api/v1/sessions/{id}`, `DELETE /api/v1/sessions/{id}`, plus the
bidirectional `POST /api/v1/live-sessions` (+ `GET`, `DELETE`, and
`POST …/{id}/control`) for `claude`, `codex`, and `pi`. The health
endpoints live at the root; `GET /api/` redirects `308` to `/api/v1`.
`GET /api/v1/agents` reports each agent's `available` (binary on PATH) and
`auth` (`ok` / `unauthenticated` / `unknown`) — `available` does **not**
imply authenticated; the `auth` probe checks credential presence, not
that a run will succeed.

**SSE frames** on a streaming `POST /api/v1/messages`. The first frame is
`event: session` (the new id); then, as the turn runs, any of:
`event: text` (assistant content deltas), `event: reasoning` (model
thinking), `event: tool_use` / `event: tool_result` (live tool activity),
`event: token_usage` (per-step input/output/cache tokens),
`event: step_finish`, `event: subagent_spawn` / `event: subagent_result`,
and `event: context_compressed`. The stream ends with `event: done`
(`{"exit_code":N}`); on failure an `event: error` (`{"code","message"}`)
is emitted just before it. Use these to watch what the agent is doing
within a turn.

**Frame fidelity is agent-dependent.** The built-in `aikit` backend emits
the richest stream: `session` first, `tool_use`/`tool_result` for every
tool call, token-by-token `text` deltas, plus `reasoning`, `token_usage`,
`step_finish`, and sub-agent/compression frames. External CLI agents
(`claude`, `codex`, `gemini`, `cursor`) emit only what their CLI surfaces —
commonly a single final `text` frame and a `token_usage` frame, often
without `tool_use`/`tool_result` (tool calls can run silently); `pi` (driven
over RPC) additionally emits structured `tool_use`/`tool_result` and
`reasoning` frames. A
`session_id` is returned when the backend exposes one: the built-in
`aikit` agent always does, and CLI backends do when their output carries a
session/turn id — but some CLIs (or versions) return `null`, so check the
response before relying on resume. The sync JSON body mirrors the stream:
it always carries `content` and `exit_code`, plus an aggregated `usage`
object when reported, `session_id` when known, and `error` (code
`agent_error`, or `unauthenticated` for auth failures) on a non-zero exit.

**Bidirectional live sessions** (`/api/v1/live-sessions`) are long-lived,
multi-turn agent processes for `claude`, `codex`, and `pi`. Open one with
`POST /api/v1/live-sessions` (the session streams as SSE) and drive it through
`POST /api/v1/live-sessions/{id}/control`, whose `action` is one of
`send_turn` (a follow-up `prompt`; pass `text`), `interrupt`, `set_model`
(pass `model`), `get_context_usage`, or `disconnect`. `pi`'s control surface is
complete: `set_model` takes `provider/id` (split into Pi's `provider` +
`modelId`; Pi rejects a bare id), and `get_context_usage` returns Pi's
`get_session_stats` payload — tokens, cost, and `contextUsage` (context-window
fill). The same ops are available programmatically on `PiControlHandle` (plus
pi-specific `steer` / `follow_up`).

**Choosing a response shape on `/api/v1/messages`** — content
negotiation via `Accept`:

```bash
# SSE (incremental events; default if Accept is missing or */*)
curl -sN -X POST http://127.0.0.1:8787/api/v1/messages \
  -H 'Accept: text/event-stream' \
  -H 'Content-Type: application/json' \
  -d '{"agent":"aikit","content":"Say hello."}'

# Single JSON body (runs to completion, returns assistant text + session_id)
curl -s -X POST http://127.0.0.1:8787/api/v1/messages \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{"agent":"aikit","content":"Say hello."}' | jq .
# → {"session_id":"…","content":"Hello, world!","exit_code":0}
```

Request body fields: `agent` (required), `content` (required),
`session_id`, `model`, `yolo`. Unknown fields are rejected (`422`).

Resume by quoting the returned `session_id` in the next request body.
Any other explicit `Accept` returns `406 Not Acceptable`.

**Logs and failure diagnosis:** the server installs a `tracing`
subscriber on stderr that honours `RUST_LOG`. When an agent exits
non-zero with no recognised output, the sync JSON body surfaces the
captured stderr tail as
`error: { code: "agent_error", message: <stderr tail> }`.

Full reference:
[github.com/goaikit/aikit/blob/main/webdocs/serve.mdx](https://github.com/goaikit/aikit/blob/main/webdocs/serve.mdx).

## Getting help

- `aikit <command> --help`
- [GitHub](https://github.com/goaikit/aikit)
