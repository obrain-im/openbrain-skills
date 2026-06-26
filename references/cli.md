# Open Brain CLI Reference

This reference describes current Open Brain CLI usage.

## Installation And Invocation

Install the CLI globally when `obrain` is not already available:

```bash
bun install -g @obrain/cli
```

Then invoke it with:

```bash
obrain <command>
```

The package name is `@obrain/cli`, and its binary is `obrain`.

## Main Commands

```bash
obrain login [--url <login-page-url>]
obrain status
obrain restart
obrain logout
obrain init workspace --name <name> [--repo <git-url>]
obrain skill update [path]
obrain agent register [agent-card-url-or-path] [--card <path>] [--local-url <url>] [--runtime opencode] [--token <jwt>] [--url <login-page-url>] [--port <number>] [--creation-id <id>] [--name <name>] [--description <text>]
obrain agent list
obrain agent describe <agent-id>
obrain agent rename <agent-id> --name="new name" --describe="new desc"
obrain agent reload [local-url]
obrain agent service register [--agent-id <agent-id>]
obrain wallet info
obrain wallet sign-message [message] [--file <path>] [--agent-id <agent-id>]
obrain wallet permission request <subject> <quota> <hour|day|week|month> [--agent-id <agent-id>]
obrain wallet permission info <id>
obrain wallet permission list [--agent-id <agent-id>] [--limit <1-100>]
obrain wallet transfer <asset> <amount> <to> [--rpc-url <url>]
obrain wallet swap <fromAsset> <toToken> <amount> [--rpc-url <url>] [--slippage-bps <bps>] [--dry-run]
obrain debug ws [--gateway-url <url>] [--token <jwt>] [--timeout <ms>]
```

Internal daemon commands exist but should not be used for normal workflows:

```bash
obrain daemon connect [--gateway-url <url>]
obrain daemon run-agent --runtime opencode --port <number> [--path <path>] [--ready-file <path>] [--name <name>] [--description <text>]
```

Most user-facing commands call `ensureDaemonIsRunning()`, which starts the daemon automatically when needed.

## Login, Daemon, And Environment

`obrain login` starts the daemon, performs device authorization when no reusable token exists, creates a remote device, stores auth state, initializes or reuses the local encrypted crypto wallet, syncs wallet state with the gateway, and reconnects the daemon connector.

Login and daemon state are stored in:

```text
~/.openbrain/gateway.json
```

Fields include:

```ts
{
  token?: string;
  userId?: string;
  deviceId?: string;
  gatewayUrl?: string;
  daemon?: { pid: number; startedAt: string };
}
```

Other paths:

```text
~/.openbrain/cli.log
daemon socket under ~/.openbrain
```

Useful environment variables:

- `OPEN_BRAIN_JWT_TOKEN`: reusable login token source.
- `OPENBRAIN_URL`: login page URL override, default `http://localhost:3001/login`.
- `OPENBRAIN_GATEWAY_URL`: gateway URL override, default `http://localhost:8787`.
- `OPENBRAIN_AUTH_TOKEN_URL`: JWT exchange endpoint override.
- `OPENBRAIN_LOG_LEVEL=debug` or `LOG_LEVEL=debug`: enable verbose gateway connector logging.
- `BETTER_AUTH_JWKS_URI`: required by the internal OpenCode runtime command.

`obrain status` prints login state, user ID, device ID, gateway URL, log path, daemon PID, socket path, connector connection state, and registered agent count.

`obrain restart` stops and restarts the background daemon, then attempts to re-associate the local wallet with agents registered on the current device. `obrain logout` clears token, user ID, and device ID from `gateway.json` and reconnects the daemon.

## Skill Update

Update the installed OpenBrain skill:

```bash
obrain skill update [path]
```

`path` is the skills directory to update. When omitted, `~`, or `~/.agents/skills`, the CLI updates `~/.agents/skills/open-brain`. For other values, the CLI resolves `~/...` relative to the home directory and other paths relative to the current working directory.

The command removes and recreates the target `open-brain` skill directory from the OpenBrain skill repository, then prints the synced path and number of files updated.

## Workspace Initialization

`obrain init workspace --name <name>` clones the workspace repository into:

```text
~/.openbrain/workspace/<name>
```

When the user does not provide initialization parameters, generate defaults before invoking the CLI:

- `name`: use the current Git repository basename when available, otherwise use the current directory basename. Normalize to lowercase kebab-case, strip `/` and `\`, collapse repeated separators, trim leading/trailing separators, and fall back to `openbrain-workspace`.
- `repo`: use `git remote get-url origin` when it exists and is not a local filesystem path. If no suitable remote exists, omit `--repo` and let the CLI use its built-in default repository.

Example with generated defaults:

```bash
obrain init workspace --name my-service --repo https://github.com/example/my-service.git
```

Example when no remote repository is available:

```bash
obrain init workspace --name my-service
```

If `~/.openbrain/workspace/<name>` already exists, pick the next available suffix, such as `<name>-2`, before running the command.

The default repository is:

```text
https://github.com/obrain-im/openbrain-workspace-.git
```

Workspace names must be non-empty and must not contain `/` or `\`.

## Agent Registration

Register an OpenCode runtime for a workspace:

```bash
obrain agent register <workspace-path> --runtime opencode
```

Useful options:

- `--port <number>`: first local runtime port to try; default is `4001`, with fallback attempts on later ports.
- `--token <jwt>`: log in with a JWT before registration.
- `--url <login-page-url>`: override the login page URL.
- `--creation-id <id>`: mark an agent creation flow complete after registration.
- `--name <name>` and `--description <text>`: fallback runtime card metadata when the server does not provide registration config.

Register from an A2A card URL:

```bash
obrain agent register <agent-card-url>
```

Register a local agent with explicit card and URL:

```bash
obrain agent register --card <agent-card-path> --local-url <local-agent-url>
```

Rename an existing agent:

```bash
obrain agent rename <agent-id> --name="new name" --describe="new desc"
```

The CLI prints the agent ID, name, optional card URL, local URL, gateway URL, wallet address, and optional creation ID after successful registration.

Registration behavior:

- Commands require a login token and device ID; run `obrain login` first unless `agent register --runtime opencode --token <jwt>` is used.
- Runtime registration starts or reuses a daemon-managed OpenCode A2A runtime, fetches the runtime card at `http://localhost:<port>/.well-known/agent-card.json`, registers the derived local URL with the remote gateway, associates the local wallet with the agent, refreshes the gateway card, and reconnects the daemon.
- `agent register <input>` treats the positional as an A2A card location unless `--runtime` is set.
- `agent register --card <path> --local-url <url>` registers an already running local agent; the card path is used only for preview/metadata, while the local URL is registered.
- `agent rename <agent-id>` updates an existing remote agent's display name and description. Use `--name="new name"` for the new name and `--describe="new desc"` for the new description.

## Agent Card URL Handling

When fetching a card, the CLI normalizes common inputs:

- `localhost:4001` becomes `http://localhost:4001/.well-known/agent-card.json`.
- A base URL resolves to `/.well-known/agent-card.json` under that base path.
- An `/a2a/jsonrpc` or `/a2a/rest` URL resolves to the sibling `/.well-known/agent-card.json`.
- A path ending in `.json` is treated as the card URL directly.

The registered local URL is derived from the fetched card in this order:

1. `card.url`.
2. An endpoint or `additionalInterfaces` entry whose `transport` is `jsonrpc` or whose URL contains `/a2a/jsonrpc`.
3. Any endpoint URL.
4. For a well-known card URL, the default sibling `../a2a/jsonrpc`.

If none of these are available, registration fails.

## Agent Runtime Reload

Schedule a delayed reload of daemon-managed runtimes when runtime discovery state must be refreshed, such as after adding a skill:

```bash
obrain agent reload
obrain agent reload http://localhost:4001/a2a/jsonrpc
```

Reload is scheduled with a short delay and coalesces duplicate requests. With no local URL, all daemon-managed runtimes are scheduled. If there are no managed runtimes, the command reports that nothing can be reloaded.

Reload restarts the affected agent runtime, which can interrupt the current agent session if it is serving itself. When running reload from an agent conversation, finish the user-facing response first, then run the reload as a delayed background command, such as `(sleep 2; obrain agent reload <local-url>) >/tmp/obrain-agent-reload.log 2>&1 &`.

## Service Registration

Register service definitions from the workspace for a registered local-device agent:

```bash
obrain agent service register
obrain agent service register --agent-id <agent-id>
```

The command scans the workspace directory for `openbrain-service.json`:

```json
{
  "version": 1,
  "services": [
    {
      "name": "posts",
      "url": "http://localhost:17298/openapi.json",
      "port": 17298,
      "type": "openapi",
      "enabled": true
    }
  ]
}
```

`17298` is only an example port. Use the actual service port discovered or configured in the current workspace, and keep `url` and `port` in sync.

Before registering services, confirm each enabled service URL belongs to the intended registered agent runtime or production-ready local service. Do not register development-only services, temporary dev server ports, mock APIs, test fixtures, or incidental ports discovered while scanning the workspace. `port` must be the latest running service port, and `url` should use that same port for the reachable OpenAPI document.

Use `--agent-id` when the current device has multiple registered agents. The command updates the remote agent service and asks the daemon connector to reconnect.

## Wallet

Use wallet commands to inspect the local encrypted crypto wallet, sign messages, request or inspect permissions, and submit approved transfers, swaps, or transactions:

```bash
obrain wallet info
obrain wallet sign-message [message] [--file <path>] [--agent-id <agent-id>]
obrain wallet permission request <subject> <quota> <hour|day|week|month> [--agent-id <agent-id>]
obrain wallet permission info <id>
obrain wallet permission list [--agent-id <agent-id>] [--limit <1-100>]
obrain wallet transfer <asset> <amount> <to> [--rpc-url <url>]
obrain wallet swap <fromAsset> <toToken> <amount> [--rpc-url <url>] [--slippage-bps <bps>] [--dry-run]
```

Read `references/wallet.md` before running wallet commands, especially transfer or swap commands.

## GraphQL Gateway Behavior

Open Brain converts configured REST/OpenAPI services into GraphQL schemas with `@omnigraph/openapi`. Multiple configured services are merged into one GraphQL schema with `@graphql-tools/schema`.

GraphQL requests use payloads shaped like:

```ts
{
  type: "graphql";
  query: string;
  variables?: Record<string, unknown>;
  operationName?: string;
}
```

If no configured service has `url`, `source`, or `graphql.source`, the gateway reports that no GraphQL service is configured for the agent.

The daemon connector also forwards raw `serviceFetch` gateway requests to the configured local service URL and returns status, response headers, and response body.

## Debugging

Check websocket connectivity:

```bash
obrain debug ws
obrain debug ws --gateway-url http://localhost:8787 --timeout 10000
```

The command prints the gateway URL, websocket URL, auth presence, events, and success/failure detail. It exits nonzero on failure.

For connector logs, set `OPENBRAIN_LOG_LEVEL=debug` or `LOG_LEVEL=debug`, restart the daemon, and inspect:

```text
~/.openbrain/cli.log
```
