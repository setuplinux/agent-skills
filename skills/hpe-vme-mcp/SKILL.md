# HPE VM Essentials MCP Skill

Version: 0.1 draft
Default mode: read-only discovery
Audience: AI agents, coding agents, CLI assistants, and technical operators testing HPE VM Essentials MCP

## Purpose

Use the HPE VM Essentials MCP endpoint for safe, read-only discovery and inventory.

This skill helps an agent connect to the MCP server, discover available tools, inspect tool metadata, call only read-only GET tools, and summarize results without leaking secrets or dumping raw operational metadata.

## Non-goals

This skill does not authorize infrastructure changes.

Do not use this skill to start, stop, restart, delete, resize, snapshot, backup, refresh, apply, reindex, acknowledge, or otherwise mutate VM Essentials resources unless the user separately approves the exact operation, target, and arguments.

## Required user inputs

The agent needs:

- VME appliance MCP endpoint, usually `https://<vme-appliance>/api/mcp`
- A bearer token or a local method to access one
- TLS mode, for example trusted certificate or insecure lab mode

Do not ask the user to paste a token into chat if a local environment variable, local config file, password manager, or existing authenticated CLI can be used instead.

## Connection model

The VME MCP endpoint uses JSON-RPC over HTTP.

Default endpoint path:

```text
/api/mcp
```

Initialize first, then capture and reuse the MCP session id returned by the server.

Required sequence:

1. `initialize`
2. Capture `Mcp-Session-Id`
3. `tools/list`
4. Load one narrow category with a `use_*_tools` loader
5. Inspect the target tool with `get_tool_details`
6. Call the tool only if it passes the safety gate
7. Summarize results without exposing tokens, headers, or raw full inventory JSON

## Initialize request

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "2025-06-18",
    "capabilities": {},
    "clientInfo": {
      "name": "vme-mcp-agent",
      "version": "0.1"
    }
  }
}
```

## List tools request

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/list",
  "params": {}
}
```

## Inspect tool request

Always inspect the exact tool before calling it.

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call",
  "params": {
    "name": "get_tool_details",
    "arguments": {
      "tool_name": "list_instances"
    }
  }
}
```

## Safety gate

Before every real `tools/call`, verify all of the following from `get_tool_details`:

- `annotations.readOnlyHint` is `true`
- `httpMethod` is `GET`
- `apiPath` is the expected read-only endpoint
- the input schema does not require a destructive target or effect
- the tool name does not match the blocked mutation patterns

If any value is missing, false, contradictory, or unknown, do not call the tool.

Tool annotations are hints, not proof. Use annotations plus HTTP method, API path, schema, and name-based risk scan.

## Blocked mutation patterns

Never call tools whose names start with or contain these patterns without explicit user approval for the exact operation:

```text
create_
update_
delete_
start_
stop_
restart_
resize_
snapshot_
backup_
clone_
refresh_
apply_
run_
maintenance
reindex
acknowledge
attach_
detach_
remove_
upgrade_
```

## Mutation approval contract

Before any non-read-only tool is called, the agent must present:

- exact tool name
- target object id and name
- full arguments
- expected effect
- risk
- rollback or recovery plan, if any

The user must explicitly approve that exact operation.

Generic approval such as "test it", "try it", "fix it", or "do what you need" is not enough.

## Recommended read-only categories

Load the narrowest useful category.

| Area | Preferred loader | Example safe tools |
|---|---|---|
| Health | `use_admin_tools` | `get_ping` |
| Clouds / zones | `use_clouds_tools` | `list_clouds`, `get_cloud`, `list_zone_types` |
| Clusters | `use_clusters_tools` | `list_clusters`, `get_cluster`, `list_cluster_workers`, `list_cluster_masters` |
| Instances / VMs | `use_instances_tools` | `list_instances`, `get_instance`, `get_instance_state` |
| Servers / hosts | `use_servers_tools` | `list_servers`, `get_server` |
| Networks | `use_networks_tools` | `list_networks`, `get_network`, `list_subnets` |
| Storage | `use_storage_tools` | `list_storage_volumes`, `get_storage_volume`, `list_datastores` |

Loaders are category loaders only. They are not safety boundaries. A loader can expose both read-only and write tools.

## Known beta behavior to watch

- Datastore tooling may expose `list_datastores` mapped to `/api/datastores` while REST may use `/api/data-stores`.
- Treat MCP/REST disagreement as a possible tool mapping issue, not proof that the resource is absent.
- Some MCP tools may return HTTP 200 with `success=false`. Treat that as a failed tool result, not a pass.
- Tool names and schemas may change across beta/pre-release builds. Re-run `tools/list` and `get_tool_details` after upgrades.

## REST comparison rule

Only compare MCP and REST results when using the same token/account/role.

If MCP and REST use different users, count and permission differences are not valid endpoint comparisons.

## Classification states

Use these states in reports:

| State | Meaning |
|---|---|
| PASS | Expected MCP result returned and matched required checks |
| FAIL | Tool returned an error, wrong endpoint, wrong data, or unexpected result |
| BLOCKED | Could not test due to missing permissions, missing token, missing endpoint, or unavailable loader |
| NOT TESTED | Conditions were not present, such as zero VMs for VM detail testing |
| BETA ISSUE | Behavior appears to be a pre-release tool/schema/path mismatch |
| EMPTY | Read-only list worked and returned zero objects |

Do not mark an empty inventory as PASS for feature validation. Mark it EMPTY or NOT TESTED depending on what was being proven.

## Standard smoke test

Run this sequence for a read-only MCP smoke test:

1. Initialize MCP session
2. Verify HTTP 200
3. Verify `Mcp-Session-Id` was returned
4. Call `tools/list`
5. Load `use_admin_tools`
6. Inspect `get_ping`
7. Call `get_ping`
8. Load `use_clouds_tools`
9. Inspect and call `list_clouds`
10. Load `use_clusters_tools`
11. Inspect and call `list_clusters`
12. Load `use_instances_tools`
13. Inspect and call `list_instances`
14. Load `use_servers_tools`
15. Inspect and call `list_servers`
16. Load `use_networks_tools`
17. Inspect and call `list_networks`
18. Load `use_storage_tools`
19. Inspect and call `list_storage_volumes`
20. Inspect and call `list_datastores`
21. Report PASS, FAIL, BLOCKED, NOT TESTED, BETA ISSUE, or EMPTY per area

## Output contract: inventory summary

Use this format for user-facing summaries:

```text
VME MCP Inventory Summary

Endpoint: https://<redacted-or-host-only>/api/mcp
Protocol: <protocol version>
MCP server: <name/version>
VME build: <build version if available>

Counts:
- Clouds/zones: <count/status>
- Clusters: <count/status>
- Hosts/servers: <count/status>
- Instances/VMs: <count/status>
- Networks: <count/status>
- Storage volumes: <count/status>
- Datastores: <count/status>

Findings:
- PASS: <items>
- FAIL: <items>
- BLOCKED: <items>
- NOT TESTED: <items>
- BETA ISSUE: <items>

Sensitive data omitted:
- bearer tokens
- raw session headers
- full raw inventory JSON
- customer-sensitive operational metadata
```

## Output contract: tool test result

```text
Tool Test Result

Tool: <tool name>
Loader: <loader name>
Details checked: yes/no
readOnlyHint: true/false/missing
httpMethod: GET/other/missing
apiPath: <path/missing>
Arguments: <sanitized arguments>
MCP result: <HTTP/tool status and summarized count>
REST comparison: <same-token comparison or not run>
Status: PASS/FAIL/BLOCKED/NOT TESTED/BETA ISSUE/EMPTY
Notes: <short notes>
```

## Sensitive data handling

Never print, store, commit, or screenshot:

- bearer tokens
- API tokens
- authorization headers
- full MCP session headers
- passwords or secrets
- private customer names unless already approved
- full raw inventory JSON unless explicitly requested and sanitized

Prefer summarized output.

## Good prompts

- "Use the VME MCP skill to run a read-only smoke test."
- "Summarize my VME inventory through MCP without raw JSON."
- "Check whether datastores are visible through MCP and compare with REST using the same token."
- "List available MCP tools but do not call mutation tools."

## Risky prompts

These require clarification and explicit safety boundaries:

- "Try all tools."
- "Fix my cluster through MCP."
- "Restart anything that looks broken."
- "Run the storage tools and see what happens."
- "Snapshot everything."

## Agent behavior

- Prefer read-only discovery.
- Ask for missing endpoint/token source only when needed.
- Do not ask the user to paste secrets into chat when local access is possible.
- Do not infer resource absence from an MCP tool failure if REST succeeds.
- Mark missing tools as BLOCKED or NOT AVAILABLE, not FAIL.
- Mark endpoint mismatch as BETA ISSUE.
- Keep summaries short and operational.
- When uncertain, stop before mutation.
