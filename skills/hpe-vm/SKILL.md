---
name: hpe-vm
description: Use when helping an agent or user safely connect to HPE VM Essentials / Morpheus VME through MCP on 9.x+, REST API fallback, or approved read-only SSH.
---

# HPE VM Essentials Agent Onboarding Skill

Version: 0.3 draft
Default mode: read-only onboarding and discovery
Audience: AI agents, coding agents, CLI assistants, and technical operators learning HPE VM Essentials / Morpheus VME

## Purpose

Use this skill to train an agent to safely help a user connect to HPE VM Essentials / Morpheus VME, prove access, and gather a small read-only health or inventory summary.

Version gate: MCP is only expected on VME / Morpheus 9.x and newer. For 9.x+ systems, use MCP as the preferred path, with REST API as a read-only fallback when an MCP tool is missing or misbehaving. For older/pre-9 systems, skip MCP entirely and use REST API first; use SSH only as a last resort when the user explicitly approves shell access and the command set is read-only.

This is intentionally written as one onboarding skill first. Split it into separate `vme-mcp`, `vme-api`, and `vme-ssh` skills later only if the workflow becomes too large or if different teams need to own the paths independently.

## Operating principles

- Read-only first.
- Health-only before inventory.
- No infrastructure changes without exact explicit approval.
- No secrets in chat, tickets, commits, screenshots, or logs.
- Do not dump raw inventory JSON by default.
- Do not assume every appliance has MCP enabled.
- Do not attempt MCP on VME / Morpheus versions older than 9.x; use API first and SSH last.
- Do not assume every lab has trusted TLS.
- Treat MCP, API, UI, and SSH as different surfaces that may expose different behavior.

## Recommended skill structure

Keep this as one all-in-one VME onboarding skill while it is still being tested with friends or outside agents:

1. Common safety rules and user walkthrough.
2. MCP primary path.
3. REST API fallback path.
4. SSH last-resort path.
5. Output templates and handoff notes.

Reason: a new agent needs one place to start. Separate skills are useful later, but too many entry points make first-contact onboarding harder.

## Version-gated connection decision

Before choosing a connection surface, identify the appliance version/build if the user knows it or if `/api/ping` exposes it. Do not require the user to know the version before doing no-secret probes.

| Appliance version | First path | Fallback path | Do not use |
|---|---|---|---|
| VME / Morpheus 9.x or newer | MCP at `/api/mcp` | REST API, then approved SSH | Mutation tools without exact approval |
| Older than 9.x | REST API | Approved SSH | MCP; it is not expected to exist |
| Unknown | No-secret API probes first, then infer from `/api/ping` or UI | If 9.x+, MCP; otherwise API/SSH | Guessing that MCP exists |

If `/api/mcp` returns 404/not found on a known older system, report `MCP not available on this version` rather than treating it as a failure. If `/api/ping` works but `/api/mcp` is unavailable and the version is 9.x+, report it as `MCP unavailable or disabled` and continue with read-only REST only if the user approves that scope.

## Non-goals

This skill does not authorize infrastructure changes.

Do not use this skill to start, stop, restart, delete, resize, snapshot, backup, refresh, apply, reindex, acknowledge, or otherwise mutate VM Essentials resources unless the user separately approves the exact operation, target, and arguments.

## Required user inputs

The agent needs:

- VME appliance URL, usually `https://<vme-appliance>`
- VME / Morpheus version if known; if unknown, infer from no-secret `/api/ping` or UI
- For 9.x+ systems: MCP endpoint, usually `https://<vme-appliance>/api/mcp`
- For all supported systems: REST API bearer token or a local method to access one
- TLS mode, for example trusted certificate or insecure lab mode

Do not ask the user to paste a token into chat if a local environment variable, local config file, password manager, or existing authenticated CLI can be used instead.

## First questions for the user

Ask only what changes the next action:

1. What is the appliance address or URL? Example: `https://vme.example.com` or `https://10.10.10.10`.
2. Do you know the VME / Morpheus version? If not, say unknown and the agent should infer it from safe probes.
3. Where is the agent running from: same LAN, VPN, jump host, cloud VM, or laptop?
4. Is the TLS certificate trusted, private CA, or lab/self-signed?
5. Is data from VME allowed to leave the organization for an external LLM/service?
6. Who can revoke the token or disable access if needed?

Do not ask the user to paste passwords, bearer tokens, refresh tokens, SSH private keys, cookies, or generated credentials into chat.

## User walkthrough: create a safe API token

The agent must guide the user to create the API key in the VME UI without asking to see the token. Use this script in chat, adapted to the user's appliance:

> Please open VME / Morpheus in your browser. Go to the top-right user menu → `User Settings` → `API Keys` → `+ Add`. Name the key something like `<agent-name>-vme-readonly-smoke`. For the first smoke test, choose client id `morph-api` unless your organization has a different standard. Copy only the Access Token into your local password manager, shell secret prompt, or agent secret store. Do not paste the Access Token or Refresh Token into this chat. Save the Refresh Token only if you know where it will be stored and who can revoke it.

Agent rules for this step:

1. Do not request, display, log, screenshot, or commit the Access Token or Refresh Token.
2. Prefer the Access Token for first API/MCP smoke tests. Do not use the Refresh Token unless refresh behavior is explicitly needed.
3. Prefer client id `morph-api` for generic API/MCP onboarding. Do not use `morpheus-terraform` unless the task is Terraform-specific.
4. Ask the user to store the token locally through a secret prompt, password manager, OS keychain, encrypted secret store, or agent-specific secret mechanism.
5. If the user cannot store the token safely, stop and help choose a safer method before continuing.
6. Confirm who can revoke the key if the agent host is lost or the test ends.

Suggested local environment variables for Linux/macOS when the user is comfortable editing their own shell profile locally:

```bash
export HPE_VME_URL="https://<vme-appliance>"
# Only needed for VME / Morpheus 9.x and newer:
export HPE_VME_MCP_URL="https://<vme-appliance>/api/mcp"
# The user sets this locally; do not paste the real value into chat:
export HPE_VME_TOKEN="<access-token stored locally, never pasted into chat>"
```

Safer interactive shell pattern that avoids putting the token in shell history:

```bash
read -rsp "VME Access Token: " HPE_VME_TOKEN; echo
export HPE_VME_TOKEN
```

For Windows PowerShell, tell the user to paste the command first and then paste the token only when prompted:

```powershell
$env:HPE_VME_URL = "https://<vme-appliance>"
$token = Read-Host -AsSecureString "VME Access Token"
$env:HPE_VME_TOKEN = [Runtime.InteropServices.Marshal]::PtrToStringAuto([Runtime.InteropServices.Marshal]::SecureStringToBSTR($token))
# Only needed for VME / Morpheus 9.x and newer:
$env:HPE_VME_MCP_URL = "$($env:HPE_VME_URL)/api/mcp"
```

If the user is not comfortable with local environment variables, use their password manager or the agent platform's built-in secret store instead.

## No-secret reachability probes

Before using credentials, check network and endpoint shape. Use `-k` only for no-secret lab probes when TLS is not trusted.

```bash
VME_URL="https://<vme-appliance>"
HOST="${VME_URL#https://}"; HOST="${HOST%%/*}"

nc -vz "$HOST" 443
nc -vz "$HOST" 80 || true
curl -sk -i "$VME_URL/api/ping"
curl -sk -i "$VME_URL/api/mcp"
```

Interpretation:

- `/api/ping` returns HTTP 200 with success: appliance API is reachable.
- `/api/mcp` returns HTTP 401 unauthorized on a 9.x+ system: MCP path exists and needs auth.
- `/api/mcp` returns HTTP 404/not found on an older/pre-9 system: expected; use API/SSH path instead.
- Authenticated `/api/mcp` may return a session-header error until initialized; that can be normal.
- TLS failures must be understood before credentials are used.

## MCP connection model for 9.x and newer

Only use this section for VME / Morpheus 9.x and newer. Older systems should skip MCP and use the REST API fallback path.

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
| Health | `use_health_tools` or available health/admin loader | `get_ping` |
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

## REST API fallback path

Use REST as the primary path for older/pre-9 systems. Use REST as the fallback on 9.x+ when MCP is unavailable, disabled, missing a needed read-only tool, or appears to have a beta tool mapping problem.

Rules:

- Use the same account/token when comparing MCP and REST.
- Keep requests read-only: `GET` only.
- Do not print the bearer token.
- Use `-k` only for approved lab/self-signed TLS situations.
- Summarize fields instead of dumping raw JSON.

Example read-only checks:

```bash
VME_URL="${HPE_VME_URL:-https://<vme-appliance>}"
TOKEN="$HPE_VME_TOKEN"
CURL_TLS="--fail --silent --show-error"
# For approved lab/self-signed TLS only: CURL_TLS="-k --fail --silent --show-error"

vme_get() {
  path="$1"
  curl $CURL_TLS \
    -H "Authorization: Bearer ${TOKEN:?set HPE_VME_TOKEN}" \
    "${VME_URL}${path}"
}
vme_get "/api/ping"
vme_get "/api/whoami"
vme_get "/api/zones?max=25"
vme_get "/api/instances?max=25"
vme_get "/api/servers?max=25"
```

Common REST endpoint notes:

- `/api/ping` is the safest first API proof.
- `/api/whoami` proves token identity without revealing the token.
- `/api/zones`, `/api/instances`, `/api/servers`, `/api/networks`, `/api/storage-volumes`, and `/api/data-stores` are useful read-only checks when supported.
- Some builds may expose MCP datastore tooling as `/api/datastores` while REST uses `/api/data-stores`; report this as an endpoint mismatch, not proof that datastores are absent.

## SSH last-resort path

Use SSH only when MCP/API cannot answer the approved question and the user explicitly approves shell access.

Rules:

- Prefer a read-only account or least-privilege operational account.
- Do not ask for private keys or passwords in chat.
- Do not run service restarts, package updates, migrations, cleanup commands, or `morpheus-ctl reconfigure` without exact approval.
- Prefer commands that identify version, service status, and logs in a bounded way.

Example read-only SSH checks, if approved:

```bash
hostname
whoami
date
morpheus-ctl status
systemctl --no-pager --type=service --state=running | sed -n '1,80p'
```

If logs are needed, ask for approval and bound the output:

```bash
journalctl --no-pager -n 100
```

Do not use SSH to work around API authorization. If the API token lacks permission, report the permission boundary.

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

For VME / Morpheus 9.x and newer, run this sequence for a read-only MCP smoke test. For older systems, skip this MCP sequence and use the REST API fallback path instead:

1. Initialize MCP session
2. Verify HTTP 200
3. Verify `Mcp-Session-Id` was returned
4. Call `tools/list`
5. Load `use_health_tools` or the narrowest available health/admin loader
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

- "Use the HPE VM skill to run a read-only smoke test."
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
