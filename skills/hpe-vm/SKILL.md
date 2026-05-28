---
name: hpe-vm
description: Use when safely onboarding an agent to HPE VM Essentials / Morpheus VME through MCP, REST API fallback, or approved read-only SSH.
---

# HPE VM Essentials Agent Onboarding Skill

Version: 0.4 draft
Default mode: read-only onboarding and discovery; exact-approval mutation testing only
Audience: AI agents, coding agents, CLI assistants, and technical operators

## Purpose

Use this skill to safely onboard an agent to HPE VM Essentials / Morpheus VME for read-only discovery, smoke testing, and capability mapping.
It is portable guidance for capable agents and does not assume Claude, Codex, Hermes, OpenClaw, MCP, shell access, browser access, or persistent secret storage.

This skill is a draft field-enablement aid. It is not official HPE product documentation.

## Portability rules

- Do not assume the agent is Claude-only or tied to any single vendor runtime.
- Do not assume MCP is available.
- Do not assume shell access is available.
- Do not assume browser access is available.
- Do not assume the agent can store secrets.
- Do not assume the agent can safely call mutation tools.
- Write and follow instructions that work for any capable agent.
- If an exact MCP tool name, REST endpoint, or command is unknown, discover available capabilities first instead of inventing names.

## Safety rules

- Default to read-only discovery.
- Never request passwords, API tokens, private keys, session cookies, or bearer tokens in chat.
- Never print secrets in logs, summaries, screenshots, examples, commits, or issue comments.
- Never call create/update/delete/reboot/power/migrate/network/storage mutation tools without explicit user approval.
- Before any mutation, require:
  - exact action
  - target object
  - exact arguments
  - expected effect
  - risk
  - rollback or recovery plan
  - explicit user approval
- Treat SSH as a last resort.
- If SSH is used, prefer read-only commands only.
- Do not bypass the VME UI/API by manually changing host state unless explicitly approved.
- Do not dump full raw inventory by default; summarize only the minimum needed.

## Version-gating rules

- If VME/Morpheus version is 9.x or newer, try MCP first.
- If version is older than 9.x, use REST API first.
- If version is unknown, perform safe read-only discovery first.
- If MCP is unavailable, fall back to REST.
- If REST is unavailable, ask for approved read-only SSH access or stop.

## Agent workflow

1. Identify environment:
   - product name
   - version
   - target URL or endpoint
   - access method available: MCP, REST, SSH, unknown
2. Confirm safety mode:
   - read-only
   - mutation requested
   - troubleshooting only
3. Discover capabilities:
   - MCP tools available, if any
   - REST API reachable, if any
   - SSH reachable, only if approved
4. Run minimal read-only smoke tests.
5. Summarize findings using the fixed report format below.
6. Ask for approval before anything destructive or state-changing.

## First-contact questions

Ask only what changes the next safe action:

1. What product are we targeting: HPE VM Essentials, Morpheus VME, or unknown?
2. What version is installed, if known?
3. What target URL or endpoint should be used?
4. Which access methods are available: MCP, REST, SSH, browser-only, or unknown?
5. Is the requested mode read-only discovery, troubleshooting, or a specific mutation?
6. Is the agent allowed to receive summarized inventory data, or must all details stay local?
7. Who can revoke credentials or disable access after testing?

Do not ask the user to paste credentials or secrets into chat.
If credentials are needed, instruct the user to place them in a local password manager, approved secret store, environment prompt, or existing authenticated client that the agent can use without exposing the value.

## MCP path for VME/Morpheus 9.x+

Use MCP first only when the environment is known or reasonably discovered to be VME/Morpheus 9.x or newer.

1. Discover available MCP tools first.
2. List tool names, descriptions, schemas, and read-only annotations if the MCP client exposes them.
3. Treat tool annotations as hints, not proof.
4. Prefer tools that are clearly read-only by name, schema, method, and description.
5. Do not call mutation tools unless the user has approved the exact action using the mutation approval template.
6. If MCP is unavailable, disabled, incomplete, or ambiguous, fall back to REST for read-only checks.

Do not invent MCP tool names. If the client cannot list tools or inspect tool schemas, stop and report MCP as unavailable or insufficient for safe onboarding.

### Observed MCP write-capability notes

When exact user approval is granted for a lab VM, MCP can expose and execute real change-bearing tools. Verify each tool with schema/details before calling it, and record the process or result IDs for follow-up.

Observed VME/Morpheus 9.x-style tool patterns:

| Capability | Tool pattern | Method/path shape | Notes |
|---|---|---|---|
| Create VM | `create_instance` | `POST /api/instances` | VM provisioning can work through MCP when the payload is based on discovered layout, plan, cloud/group, network, and image IDs. |
| Instance snapshot | `snapshot_instance` | `PUT /api/instances/{id}/snapshot` | Returns success plus process IDs. Verify with `get_instance_snapshots` until the snapshot reaches a terminal status such as `complete`. |
| Instance backup | `backup_instance` | `PUT /api/instances/{id}/backup` | May create a backup configuration/job and a backup result. Verify with `get_instance_backups` or backup result/job list tools until progress is clear. |
| Extra storage | `resize_instance`, `create_storage_volume`, server volume attach tools | varies | Schemas may be underspecified and failures may return generic messages such as `Looks like the server threw a gasket` or `error saving volume`. Treat this as MCP/API insufficiency, not proof storage is impossible. Capture the exact payload shape attempted and fall back to documented REST/UI only with approval. |

Storage-specific caveat: generated MCP storage tools may point at paths that differ from working REST paths. If a datastore or storage-volume MCP tool fails, verify the tool's `apiPath`, then use read-only REST discovery to distinguish a bad MCP wrapper/path from an absent product capability.

## REST API fallback path

Use REST first for versions older than 9.x. Use REST as fallback when MCP is unavailable or cannot safely answer the read-only question.

Rules:

- Use only documented or discovered read-only endpoints.
- Prefer safe identity, health, version, and limited inventory checks.
- Keep requests read-only.
- Do not print bearer tokens, headers containing credentials, session cookies, or full raw responses with sensitive inventory.
- Do not assume a REST path exists unless it is documented, discovered from the product, or provided by the user.
- If TLS, authentication, or authorization is unclear, stop and report the blocker instead of guessing.

Use the user's approved local client or secret mechanism. Avoid examples that include real hostnames, IP addresses, tokens, customer object names, or full inventory output.

## SSH fallback path

Use SSH only when MCP and REST cannot answer the approved question and the user explicitly approves read-only SSH access.

Rules:

- Prefer a read-only or least-privilege operational account.
- Do not ask for private keys, passwords, or one-time codes in chat.
- Confirm the exact host and purpose before connecting.
- Prefer bounded, read-only commands that identify product/version, service status, or recent logs.
- Do not run service restarts, package changes, database writes, host reconfiguration, migration commands, cleanup commands, or direct state changes without explicit mutation approval.
- Do not use SSH to bypass the VME UI/API authorization model.

If SSH access is not approved, stop and report that SSH was not used.

## Minimal read-only smoke tests

Choose the narrowest tests supported by the available access method:

- Confirm product identity.
- Confirm version or version family.
- Confirm endpoint reachability.
- Confirm authentication method is available without exposing secrets.
- Confirm a read-only capability list or limited inventory summary can be accessed.
- Confirm mutation-capable tools or endpoints were detected but not used, if visible.

If a test requires unknown endpoints, unknown tool names, or elevated permissions, mark it blocked instead of guessing.

## VME Agent Onboarding Result

Environment:

- Product:
- Version:
- Endpoint:
- Access method:
- Auth method:
- Mode: read-only

Checks:

- MCP available:
- REST available:
- SSH used:
- Inventory readable:
- Mutation tools detected:

## Findings:

-
-

## Risks / Blockers:

-

## Recommended next step:

## Mutation Approval Required

Requested action:
Target:
Tool/API/command:
Arguments:
Expected effect:
Risk:
Rollback/recovery plan:

Do not proceed until the user explicitly approves this exact action.

## Refusal conditions

Refuse or pause when:

- The user asks the agent to handle secrets directly in chat.
- The next step would expose credentials or sensitive customer inventory.
- The requested action is destructive or state-changing but lacks exact approval.
- The agent cannot tell whether a tool/API/command is read-only.
- The environment version or access method is unknown and no safe discovery path is available.
- The user asks the agent to bypass VME UI/API controls through direct host changes.

## Handoff notes

When handing off to another agent or operator, include only:

- Product and version if known.
- Access method tested.
- Read-only checks completed.
- High-level findings.
- Blockers and risks.
- Recommended next safe step.

Do not include secrets, raw headers, private keys, session cookies, bearer tokens, full inventory dumps, or customer-sensitive details.
