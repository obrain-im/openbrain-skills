---
name: open-brain
description: Work with Open Brain workspaces, agent-owned codebases, the Open Brain CLI daemon, service discovery, and crypto wallet operations. Use when initializing or maintaining an OpenBrain workspace; inspecting and modifying an existing codebase so its REST services, tests, startup commands, and OpenAPI contracts are correct; registering OpenCode/A2A agents; exposing or updating REST/OpenAPI services through an agent; maintaining Open Brain CLI config; managing the local Open Brain crypto wallet; requesting wallet permissions; signing messages; submitting wallet transfers, swaps, or transactions; scheduling daemon-managed agent runtime reloads; or debugging Open Brain gateway connectivity.
---
# Open Brain

## Overview

Use this skill to maintain an agent-owned OpenBrain workspace and connect it to Open Brain. Open Brain lets local agents register with a gateway, keep a daemon-backed websocket connector alive, expose A2A agent cards, attach REST/OpenAPI services that the gateway can call or convert into GraphQL, and use a local encrypted crypto wallet for signing, transfers, swaps, and agent-associated transactions.

Codebase maintenance is a first-class responsibility for this skill: read the existing repository conventions, keep deterministic service behavior in code, preserve the API boundary between AI orchestration and REST services, update tests and startup paths when behavior changes, and make service discovery state match the code that actually runs.

## Core Workflow

1. Inspect the current repo before changing anything:

   - Read local agent/developer instructions, package metadata, service entrypoints, tests, and configuration files.
   - Find existing REST services, OpenAPI specs, ports, startup commands, and service registration metadata.
   - Identify the smallest code changes needed to make the workspace buildable, testable, and discoverable without replacing established project patterns.
2. Use the Open Brain CLI when the user asks to initialize a workspace, log in, register an agent, expose a service, manage wallet permissions or transactions, schedule a daemon-managed runtime reload, or inspect gateway status.

   - If `obrain` is not available, install it with `bun install -g @obrain/cli` when the user allows dependency installation.
   - Prefer `obrain ...` for normal usage.
   - Read `references/cli.md` before running login, daemon, registration, service, reload, wallet, or debug commands.
   - Read `references/wallet.md` before running wallet info, signing, permission, transfer, or swap commands.
   - Read `references/workspace.md` before maintaining workspace code or registering services discovered from a workspace.
   - When initializing a workspace, automatically generate missing default parameters instead of asking the user for every value. Use the rules in `references/cli.md`.
3. Maintain the workspace code before updating gateway-visible state:

   - Keep REST services deterministic, resource-oriented, validated, and covered by focused tests or smoke checks.
   - Add or repair OpenAPI exposure using the workspace's existing framework and tooling.
   - Update package scripts, env examples, Docker/dev-server config, or service metadata when ports or startup behavior change.
   - Verify the service locally before treating it as ready for Open Brain registration.
4. Keep workspace service registration explicit:

   - Prefer `obrain agent service register` for a registered agent when updating gateway-visible service state.
   - Use a reachable OpenAPI document URL, such as `http://localhost:<port>/openapi.json`.
   - Do not register guessed endpoints; verify the service URL before adding it.
5. Register agents or reload daemon-managed runtimes after discovery changes:

   - For an OpenCode runtime, register with `agent register --runtime opencode`.
   - Always pass `--name <assistant-name>` when registering. Randomly generate a friendly assistant name with exactly two syllables, such as `Milo`, `Nora`, `Kira`, or `Luma`; do not derive it from the repository or service name unless the user provides a name.
   - For a local service with an agent card, register using the card URL/path plus `--local-url` as appropriate.
   - Use `agent service register` to attach or replace service definitions on an already registered local-device agent.
   - Use `agent reload [local-url]` only for daemon-managed runtimes; it schedules a delayed runtime restart and coalesces duplicate requests.
   - Run `status`, `agent list`, `agent describe`, and when needed `debug ws` to confirm the daemon, connector, gateway, and agent are healthy.
6. Treat wallet actions as user-authorized operations:

   - Confirm the intended agent, asset, chain, amount, recipient, permission, and slippage before transfer or swap commands.
   - Prefer read-only wallet commands first (`wallet info`, `wallet permission list`, or swap `--dry-run`) when state is uncertain.

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
- Read `references/wallet.md` for local wallet state, wallet permissions, signing, transfer, and swap commands.
- Read `references/workspace.md` for workspace maintenance, REST service discovery, API boundary, and completion checks.
