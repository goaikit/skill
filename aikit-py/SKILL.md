---
name: aikit-py
description: Python gateway to aikit-sdk — catalog, deploy, agent detection, run_agent (buffered), run_agent_events_py (per-event callbacks, same schema as aikit agent run --events). Use with pip/poetry/uv.
license: Apache-2.0
---

# aikit-py Skill

`aikit-py` exposes the same **programmatic gateway** as **aikit-sdk** from Python: catalog and paths, deploy commands/skills/subagents, probe installed runnable CLIs, **`run_agent`** for buffered stdout/stderr, and **`run_agent_events_py`** for incremental events (payload keys match `aikit agent run --events` NDJSON, including `token_usage_line` when emitted).

## When to use

Use `aikit-py` when you want to drive aikit from Python scripts, tools, or automation. This includes:
- Installing `aikit-py` via `pip`, `poetry`, or `uv`.
- Listing or resolving agent configurations.
- Programmatically deploying commands, skills, or subagents.
- Running coding agents: **`run_agent`** (wait for completion) or **`run_agent_events_py(..., on_event, ...)`** (callback per event; pass `stream=True` to align agent argv with CLI `--stream`).

For subprocess-only workflows you can still shell out to `aikit agent run --events`. See the [aikit skill](../aikit/SKILL.md) for CLI flags (including **long prompts via stdin** to avoid Linux `ARG_MAX` when you omit `-p`) and the [aikit-py README](https://github.com/goaikit/aikit/blob/main/aikit-py/README.md) for API details.

Run `aikit agent run --help` for the full CLI option list.

## Installation

Install `aikit-py` using your preferred Python package manager:

### pip

```bash
pip install aikit-py
```

### poetry

```bash
poetry add aikit-py
```

### uv

```bash
uv add aikit-py
```

## Consume

Once installed, you can import and use `aikit_py`:

```python
import aikit_py

# List all available agents
agents = aikit_py.all_agents()
for agent_config in agents:
    print(f"Agent: {agent_config.name} (Key: {agent_config.key()})")

# Get a specific agent
claude_agent = aikit_py.agent("claude")
if claude_agent:
    print(f"Claude's commands directory: {claude_agent.commands_dir}")

# Validate an agent key
try:
    aikit_py.validate_agent_key("cursor-agent")
    print("'cursor-agent' is valid.")
except aikit_py.PyDeployError as e:
    print(f"Validation error: {e.message}")
```

## Add a new skill

You can deploy new skills programmatically to agents that support them.

```python
import aikit_py
import tempfile
import os

with tempfile.TemporaryDirectory() as project_root:
    agent_key = "cursor-agent" # Ensure this agent supports skills
    skill_name = "my-automation-skill"
    skill_md_content = """---
name: My Automation Skill
description: A skill deployed for automation tasks.
license: Apache-2.0
---
This skill performs automated tasks using Python.
"""
    # Optional scripts for the skill. Content must be bytes.
    optional_scripts = [
        ("setup.sh", b"#!/bin/sh
echo 'Setting up automation...'"),
        ("run.py", b"#!/usr/bin/env python
print('Running automation logic!')")
    ]

    try:
        deployed_skill_path = aikit_py.deploy_skill(
            agent_key,
            project_root,
            skill_name,
            skill_md_content,
            optional_scripts
        )
        print(f"Skill '{skill_name}' deployed to: {deployed_skill_path}")

        # You can verify the files exist
        assert os.path.exists(deployed_skill_path)
        assert os.path.exists(os.path.join(os.path.dirname(deployed_skill_path), "scripts", "run.py"))

    except aikit_py.PyDeployError as e:
        print(f"Failed to deploy skill: {e.message} (Kind: {e.kind})")

    # Deploying a command or subagent is similar:
    # aikit_py.deploy_command(agent_key, project_root, "my-command", "# Command content")
    # aikit_py.deploy_subagent(agent_key, project_root, "my-subagent", "# Subagent content")
```
