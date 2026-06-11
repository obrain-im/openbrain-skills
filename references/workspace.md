# Open Brain Workspace Maintenance

Use this reference when maintaining an OpenBrain workspace code repository, exposing workspace REST services, or deciding what must be updated before registering or reloading an agent.

## Workspace Role

An OpenBrain workspace is code maintained by an agent. The workspace should contain deterministic service code, tests, service startup commands, and discoverable REST/OpenAPI contracts. Open Brain registration is done by registering the agent; service definitions can be attached to the remote agent and then exposed through the gateway, including GraphQL conversion for OpenAPI-backed services.

## Maintenance Flow

1. Read the workspace's local agent instructions, such as `AGENT.md`, before editing code.
2. Identify the REST services the workspace owns.
3. Verify each service has a reachable OpenAPI document.
4. Update service registration through `obrain agent service register` when the agent is already registered.
5. Register or reload the agent after service configuration, runtime URL, or OpenAPI exposure changes.
6. Verify with `obrain status`, `obrain agent list`, `obrain agent describe <agent-id>`, and `obrain debug ws` when connectivity is uncertain.

## API Boundary

Keep deterministic API behavior in code:

- Accept structured inputs.
- Return structured outputs.
- Enforce validation and resource invariants.
- Keep persistence, conflict handling, and resource lifecycle in the REST service.
- Expose stable resource-oriented REST endpoints.

Keep AI behavior outside the REST API runtime:

- Interpret natural language.
- Decide which service operation to call.
- Translate between user intent and structured request fields.
- Format structured results for the user.

When a task mixes both, write the boundary first and keep the REST API contract stable.

## Service Configuration

For an already registered local-device agent, register service definitions from the workspace:

```bash
obrain agent service register
obrain agent service register --agent-id <agent-id>
```

Use `--agent-id` when the current device has more than one registered agent.

`obrain agent service register` scans the workspace directory for `openbrain-service.json`. Keep that file next to the service workspace metadata and update it whenever the service port or OpenAPI URL changes:

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

`name` can be any stable service name. `port` must be the latest running service port, and `url` should use that same port for the reachable OpenAPI document.

Use `references/cli.md` for current service registration commands and verification steps.

If the workspace still contains a legacy `.openbrain.json`, inspect it as useful context, but prefer the current CLI behavior when deciding what will actually be registered.

## REST Service Discovery Checklist

Search for:

- OpenAPI or Swagger files.
- Framework route declarations.
- API package scripts and dev server ports.
- Docker Compose services and exposed ports.
- Environment variables that affect base URLs or auth headers.
- Existing smoke tests or route tests.

Do not register guessed endpoints. If no OpenAPI document exists, add one through the workspace's existing framework or ask for the intended service contract.

## Agent Runtime Checklist

For daemon-managed OpenCode agents:

- Register with `obrain agent register <workspace-path> --runtime opencode`.
- The runtime publishes an A2A card at `http://localhost:<port>/.well-known/agent-card.json`.
- The local agent URL usually resolves to `http://localhost:<port>/a2a/jsonrpc`.
- Default registration starts at port `4001` and tries later ports on conflicts.
- If the runtime fails during startup, check that `BETTER_AUTH_JWKS_URI` is available and inspect `~/.openbrain/cli.log`.

## Completion Standard

Treat the task as complete only after:

1. Relevant code changes are implemented and verified.
2. OpenAPI exposure matches the running service.
3. Open Brain service registration reflects the reachable service schema through `agent service register`.
4. The agent has been registered or reloaded when discovery state changed.
5. The gateway-visible state has been checked with CLI status or agent inspection commands; use `debug ws` and `~/.openbrain/cli.log` when connector health is part of the task.
