---
name: aikit
description: Universal package manager for AI agent extensions. Use when initializing projects with aikit init, installing or managing packages with aikit install/list/update/remove, creating and publishing packages with aikit package init/build/publish, or checking agent availability with aikit check.
license: Apache-2.0
---

# AIKIT

AIKIT is a universal package manager for AI agent extensions. Create, share, and install reusable commands and templates across 17+ AI assistants (Claude, Cursor, GitHub Copilot, Gemini, etc.). Use this skill when working with aikit from the CLI or creating/publishing packages.

## When to use

- Initializing a project for an AI assistant (Cursor, Claude, etc.).
- Installing, listing, updating, or removing packages.
- Creating a new package (aikit.toml, templates).
- Building and publishing packages to GitHub.
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

# Install a community package
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

## Getting help

- `aikit <command> --help`
- [GitHub](https://github.com/goaikit/aikit)
