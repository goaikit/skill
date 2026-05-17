# AIKIT Skill

A FastSkill-compatible skill that teaches an AI coding agent how to use
**AIKit** — the multi-agent CLI + gateway from
[github.com/goaikit/aikit](https://github.com/goaikit/aikit).

Once installed, the agent can:

- run any supported coding-agent CLI uniformly (`aikit agent run`),
- spin up the multi-turn HTTP API (`aikit serve`) and talk to it over
  SSE or single-shot JSON,
- author, build, and publish `aikit.toml` packages,
- merge MCP server entries via `aikit agent mcp`,
- scaffold Spec-Driven Development projects (`aikit init`).

## Installation

From your project root:

```bash
fastskill add https://github.com/goaikit/skill.git
```

See the AIKit [README](https://github.com/goaikit/aikit/blob/main/README.md)
for full CLI documentation and the
[serve API reference](https://github.com/goaikit/aikit/blob/main/webdocs/serve.mdx)
for the HTTP surface.

See [CONTRIBUTING.md](CONTRIBUTING.md) for development and release
details.

## License

Apache-2.0
