# Access and onboarding

Use this reference for a first agent connection or when choosing among UI, MCP, REST, and SSH.

## Minimum questions

Ask only what changes the next safe action:

1. Is the target the VME appliance/control plane, an HVM host, or a managed workload?
2. What product version/build and cluster layout are installed, if known?
3. What appliance URL and target object are in scope?
4. Which access methods are already authorized: UI/browser, MCP, REST/API, or SSH?
5. Is this read-only discovery, incident triage, or a proposed change?
6. Must all inventory/evidence remain local?
7. Who can revoke access and approve changes?

Do not ask anyone to paste credentials into chat.

## Credential handling

Prefer, in order:

1. The agent platform's secret store or existing authenticated integration.
2. A least-privilege environment variable injected outside the conversation.
3. An existing authenticated CLI/browser session.
4. A local protected configuration file only when the operator explicitly chooses it.

Use a read-only or least-privilege role for discovery. The agent should report the authenticated role when visible, but never the credential value. If safe secret storage is unavailable, stop and explain the required mechanism.

## MCP

MCP availability and tool catalogs vary by VME build, role, plugins, and configuration. Verify the live appliance rather than hardcoding a catalog.

1. Confirm product/version/build through a read-only path.
2. Inspect the appliance UI for configured MCP/AI Services and the advertised service URL.
3. Initialize through the client's supported transport and preserve any required session identifier privately.
4. Call `tools/list` or the client's equivalent.
5. Inspect an unfamiliar tool's schema/details before invoking it.
6. Start with identity, status, health, inventory, task, event, and alert reads.
7. Treat loaders and dynamically exposed tool families as capability discovery, not authorization.
8. Classify every tool as read-only, change-bearing, or unknown. Unknown means do not invoke.

Tool names such as `list_*` and `get_*` often indicate reads, while `create_*`, `update_*`, `delete_*`, `start_*`, `stop_*`, `restart_*`, `resize_*`, `clone_*`, `execute_*`, `apply_*`, and `upload_*` often indicate changes. Names are hints only; inspect the schema and enforce permissions at the VME role/token boundary.

A reachable endpoint returning `401` is not an authenticated MCP connection. Streamable HTTP responses may use JSON or server-sent-event framing. A wrapper/tool error does not prove the underlying VME API is broken; verify the equivalent read-only operation through REST when available.

## REST/API

- Use only routes found in current official documentation, live discovery, or an operator-provided API contract.
- Start with ping/reachability, identity/current user, version/about, health, limited inventory, tasks, events, and alerts.
- Avoid POST, PUT, PATCH, and DELETE during discovery.
- Parameterize and paginate queries; do not fetch every object when a narrow query answers the question.
- Record base URL, route, status code, timestamp, and version/build without recording authorization headers.
- Treat TLS or authentication ambiguity as a blocker; do not disable verification or guess credentials without operator approval.

## SSH

SSH is a fallback for questions that the control plane cannot answer, not a shortcut around VME authorization. “You may SSH if needed” authorizes only bounded read-only diagnosis when the target hosts and purpose are clear; it does not authorize changes.

- Confirm each target host and purpose.
- Check command availability/help before relying on version-specific syntax.
- Use timeouts for commands that may block on storage, network, or clustered services.
- Run evidence checks one host at a time; label host and timestamp.
- Show runbook commands as if already logged into the host. Do not publish SSH wrappers, key paths, passwords, host loops, or private addresses.
- SSH commands must remain read-only unless the exact mutation is separately approved.

Useful read-only starting points, when installed and appropriate:

```bash
hvmcli version
hvmcli cluster status --json
hvmcli cluster nodes --list --json
hvmcli health check --json
hvmcli vm list --details --json
hvmcli node show --json
hvmcli interfaces list --json
hvmcli storage list --json
hvmcli storage multipath status --json
```

Do not assume these commands exist or have identical flags on every release. Check `hvmcli --help` and subcommand help first.

## First discovery pass

Collect a bounded summary of:

- appliance identity, version/build, time, and health;
- cluster/layout and host membership;
- VM count and affected VM placement/state;
- datastore/storage and network status;
- active alerts, failed/recent tasks, events, and maintenance operations;
- available MCP tools or API capabilities;
- whether any write-capable surface is visible but unused.

Finish with the result format in `SKILL.md`, explicitly stating `Writes performed: none`.
