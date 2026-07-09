---
name: hpe-vm
description: Use when safely onboarding an agent to HPE VM Essentials / Morpheus VME through MCP, REST API fallback, or approved read-only SSH.
---

# HPE VM Essentials Agent Onboarding Skill

Version: 0.3 draft
Default mode: read-only onboarding and discovery
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
- In runbooks and customer-facing examples, show commands as if the operator is already on the target host. Do not include SSH wrapper commands, credential paths, or host loops; spell out that the same command block should be run on each relevant host.
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

## HPE Clustered Datastore / GFS2 troubleshooting

Use this section when the VME environment uses HPE Clustered Datastore, shared block storage, GFS2, DLM, iSCSI, or multipath. Prefer MCP first, REST second, and SSH only when the approved question cannot be answered through the control plane.

Layer the investigation instead of jumping straight to filesystem repair:

1. Product/version and cluster layout.
2. Control-plane status: cluster, hosts, datastores, VMs, alerts, tasks, events.
3. Host quorum: Corosync node count and quorate state.
4. Locking: DLM lockspace membership for the GFS2 datastore.
5. Filesystem: GFS2 mount present on every expected host.
6. Storage transport: iSCSI sessions or equivalent shared-block transport.
7. Multipath: same shared WWID visible on every expected host, paths active/ready/running.
8. Libvirt: storage pool active and VM disks actually placed on the datastore.
9. Workload: guest VM power state, reachability, and safe application check.

Copy/paste SSH fallback health gate, only after SSH is approved:

```bash
corosync-quorumtool | egrep 'Nodes:|Quorate:'
dlm_tool ls -n
findmnt -t gfs2
iscsiadm -m session
multipath -ll
virsh pool-list --all
virsh list --all --title
systemctl --no-pager status corosync dlm morpheus-morphd multipathd iscsid libvirtd
```

Field cautions:

- Pacemaker/PCS may be inactive or masked on newer HVM cluster layouts where Morpheus Agent owns HA; do not call that a failure without confirming the expected layout.
- Do not use `systemctl stop corosync` as a casual reversible test on a node with active GFS2. It can strand DLM/GFS2 lockspaces or leave unmounts stuck in uninterruptible sleep, requiring node-level recovery.
- A GUI can lag lower-layer truth. Time both directions: first CLI/workload symptom, first GUI symptom, first CLI/workload recovery, and first GUI clear.
- If the VME appliance VM is down, the REST/API and GUI may be unavailable even while host-level quorum/storage is healthy. Recover through the managed control plane when available; use host-side `virsh start` only when explicitly approved as an emergency or lab recovery path.
- Do not proceed to the next failure test until quorum, DLM, GFS2 mount, transport sessions, multipath, libvirt pool, workload state, and control-plane/API/GUI status are all green or the remaining GUI lag is deliberately recorded.

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
