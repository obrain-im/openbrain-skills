---
name: open-brain
description: Work with Open Brain, an agent registry, local daemon, and service discovery platform. Use when Codex needs to initialize an OpenBrain workspace, log in or inspect the CLI daemon, register OpenCode/A2A agents, expose REST/OpenAPI services through an agent, maintain Open Brain CLI config, schedule daemon-managed agent runtime reloads, or debug Open Brain gateway connectivity.
---

# Open Brain

## Overview

Use this skill to connect an agent-maintained workspace to Open Brain. Open Brain lets local agents register with a gateway, keep a daemon-backed websocket connector alive, expose A2A agent cards, and attach REST/OpenAPI services that the gateway can call or convert into GraphQL.

## Core Workflow

1. Inspect the current repo before changing anything:
   - Find existing REST services, OpenAPI specs, ports, and startup commands.

2. Use the Open Brain CLI when the user asks to initialize a workspace, log in, register an agent, expose a service, schedule a daemon-managed runtime reload, or inspect gateway status.
   - If `obrain` is not available, install it with `bun install -g @obrain/cli` when the user allows dependency installation.
   - Prefer `obrain ...` for normal usage.
   - Read `references/cli.md` before running login, daemon, registration, service, reload, or debug commands.
   - Read `references/workspace.md` before maintaining workspace code or registering services discovered from a workspace.
   - When initializing a workspace, automatically generate missing default parameters instead of asking the user for every value. Use the rules in `references/cli.md`.

3. Keep workspace service registration explicit:
   - Prefer `obrain agent service register` for a registered agent when updating gateway-visible service state.
   - Use a reachable OpenAPI document URL, such as `http://localhost:<port>/openapi.json`.
   - Do not register guessed endpoints; verify the service URL before adding it.

4. Register agents or reload daemon-managed runtimes after discovery changes:
   - For an OpenCode runtime, register with `agent register --runtime opencode`.
   - Always pass `--name <assistant-name>` when registering. Randomly generate a friendly assistant name with exactly two syllables, such as `Milo`, `Nora`, `Kira`, or `Luma`; do not derive it from the repository or service name unless the user provides a name.
   - For a local service with an agent card, register using the card URL/path plus `--local-url` as appropriate.
   - Use `agent service register` to attach or replace service definitions on an already registered local-device agent.
   - Use `agent reload [local-url]` only for daemon-managed runtimes; it schedules a delayed runtime restart and coalesces duplicate requests.
   - Run `status`, `agent list`, `agent describe`, and when needed `debug ws` to confirm the daemon, connector, gateway, and agent are healthy.

## Workspace Initialization Defaults

When the user asks to initialize an Open Brain workspace and does not provide every CLI option, generate defaults before running `obrain init workspace`:

- `--name`: derive from the current repository or directory name, normalize to lowercase kebab-case, remove invalid path separators, and fall back to `openbrain-workspace` if no stable name is available.
- `--repo`: use the current Git remote URL when it exists and points to a non-local repository. Omit `--repo` when no suitable remote is configured so the CLI uses its built-in default repository.

Tell the user which defaults were generated before running the command. If a generated value conflicts with an existing `~/.openbrain/workspace/<name>` directory, append a short numeric suffix such as `-2` rather than overwriting it.

## Service Discovery

When asked to register REST services from a workspace:

- Search for OpenAPI files (`openapi.json`, `openapi.yaml`, `swagger.json`, `swagger.yaml`) and route definitions.
- Identify service base URLs from dev server config, Docker Compose, env files, package scripts, or README instructions.
- If no OpenAPI spec exists, recommend adding one through the app's existing framework or docs generator before registering with Open Brain.
- Avoid inventing service endpoints; use discovered ports and specs from the workspace.

## Config

Open Brain CLI stores login and daemon state in:

```text
~/.openbrain/gateway.json
```

Do not edit global Open Brain user state unless the task requires it.

## References

- Read `references/cli.md` for current CLI commands, daemon behavior, config fields, agent-card URL handling, service registration, and GraphQL gateway behavior.
- Read `references/workspace.md` for workspace maintenance, REST service discovery, API boundary, and completion checks.
