---
name: aikit
description: Multi-agent template package manager and CLI (aikit init, install/list/update/remove, package init/build/publish, release, aikit run, check). For Rust/Python integrations use aikit-sdk and aikit-py as the programmatic gateway to the same catalog, deploy, detection, and run/event APIs.
license: Apache-2.0
---

# AIKIT

**Multi-agent template package manager and CLI:** install and publish `aikit.toml` packages from GitHub or a local directory, map templates into many assistant layouts, scaffold with `aikit init`, run supported agent CLIs with `aikit run`. The catalog covers **18** assistants; `aikit run` supports `codex`, `claude`, `gemini`, `opencode`, `agent`. **aikit-sdk** (Rust) and **aikit-py** (Python) mirror the same gateway for automation (deploy, probe CLIs, buffered run, event stream / NDJSON shape).

## When to use

- Initializing a project for an AI assistant (Cursor, Claude, etc.).
- Installing, listing, updating, or removing packages.
- Creating a new package (aikit.toml, templates).
- Building and publishing packages to GitHub.
- Running coding agents directly from the CLI (with optional streaming or machine-readable NDJSON output).
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
| `aikit install <source>` | Install packages from GitHub (owner/repo) or local directory |
| `aikit list` | Show installed packages (optional: `--author`, `--detailed`) |
| `aikit update <pkg>` | Update a package to latest (optional: `--breaking`) |
| `aikit remove <pkg>` | Uninstall a package (optional: `--force`) |
| `aikit run` | Run a coding agent with a prompt (`-a/--agent`, `-m/--model`, `-p/--prompt`, `--yolo`, `--stream`, `--events`) |
| `aikit check` | Check git, VS Code, and AI agent CLIs availability |
| `aikit version` | Show version |
| `aikit package init <name>` | Create a new package with aikit.toml |
| `aikit package build` | Build distributable package (dist/ or .genreleases/) |
| `aikit package publish <owner/repo>` | Publish package to GitHub (release and assets) |
| `aikit release <version>` | Create GitHub release from .genreleases/ (e.g. v1.0.0) |

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

- **GitHub auth**: Set `GITHUB_TOKEN` or `GH_TOKEN` in `.env` or use `--token` / `--github-token` on install, init, or release.
- **Package manifest**: Each package has an `aikit.toml` (name, version, description). Required for local installs and publish.

Example `.env`:

```bash
GITHUB_TOKEN=your_github_token_here
```

## Run coding agents

The `aikit run` command executes AI coding agents directly. Runnable agents: `codex`, `claude`, `gemini`, `opencode`, `agent`.

```bash
# Run with inline prompt
aikit run --agent claude -p "Refactor this function"

# Read prompt from stdin
echo "Add tests" | aikit run --agent opencode

# Emit NDJSON events to stdout (one JSON object per line)
aikit run --agent claude --events -p "Summarize the project"

# Combine --events and --stream for streaming-aware JSON output
aikit run --agent claude --events --stream -p "Refactor this module"
```

**NDJSON** (one object per stdout line when using `--events`): `payload` is one of `json_line`, `raw_line`, `raw_bytes`, or `token_usage_line` (normalized usage after a line that contained usage). Example:

```json
{"agent_key":"claude","seq":1,"stream":"stdout","payload":{"json_line":{"type":"progress","message":"Starting..."}}}
{"agent_key":"claude","seq":2,"stream":"stdout","payload":{"raw_line":"Processing file..."}}
```

**Key options:**

| Flag | Short | Description | Default |
|------|-------|-------------|---------|
| `--agent` | `-a` | Runnable agent key | **Required** |
| `--model` | `-m` | Model passed to the agent | Optional; if omitted, aikit does not inject a model (agent decides) |
| `--prompt` | `-p` | Prompt | Reads from stdin if omitted |
| `--yolo` | | Auto-confirm, skip checks | `false` |
| `--stream` | | Agent-native streaming flags | `false` |
| `--events` | | NDJSON event stream to stdout | `false` |
| `--debug` | | Verbose diagnostics (global `aikit` flag; see `aikit run --help`) | `false` |

`--stream` and `--events` are independent: `--stream` tunes agent argv; `--events` switches CLI output to NDJSON. Combined use is valid and recommended for tooling integration.

`aikit run` does not read `CODING_AGENT` or `CODING_AGENT_MODEL`; always pass `-a` (and `-m` only when you want to override the agent's model).

Run `aikit run --help` for the authoritative option reference.

## Getting help

- `aikit <command> --help`
- [GitHub](https://github.com/goaikit/aikit)
