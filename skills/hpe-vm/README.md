# Unofficial VME Field Guide for Agents

Independent, community-authored guidance for AI agents working with HPE VM Essentials / Morpheus VM Essentials. It is not official HPE documentation, is not endorsed or supported by HPE, and must not be presented as product policy. Product names belong to their respective owners.

The portable Agent Skills package is this directory:

```text
skills/hpe-vm/
├── SKILL.md
├── README.md
└── references/
    ├── access-and-onboarding.md
    ├── clustered-datastore-gfs2.md
    ├── incident-triage.md
    └── public-sources.md
```

`SKILL.md` is the agent-facing entry point. Send the complete directory so its references remain available. The package uses standard `name` and `description` YAML frontmatter and does not require a specific agent runtime.

## Intended agents

- Codex and other coding/CLI agents
- Hermes and OpenClaw
- Claude or other Agent Skills-compatible assistants
- Technical operators using an agent as a read-only copilot

## What it enables

- Safe read-only onboarding and capability discovery
- UI, MCP, REST/API, and approved SSH access selection
- Cross-layer VME incident triage
- VM, host, storage, network, guest, and application correlation
- HPE Clustered Datastore/GFS2/DLM/iSCSI/multipath diagnostics
- Controlled recovery proposals with exact approval and rollback requirements
- Privacy-safe evidence and handoff reports

## Safety position

The agent may perform bounded read-only discovery through already authorized access. Every state-changing action requires exact target/action approval, prechecks, risk/blast-radius explanation, rollback/recovery, and verification.

The skill explicitly prohibits agents from:

- requesting or exposing credentials in chat;
- inventing endpoints, tools, commands, or product behavior;
- treating urgency as mutation approval;
- bypassing the VME control plane merely because SSH is available;
- forcing quorum or changing clustered storage while membership/device identity is inconsistent;
- publishing raw customer inventory, logs, topology, or identifiers;
- implying this package is authored, endorsed, certified, or supported by HPE.

## Version boundary

VME behavior changes across builds and cluster layouts. The agent must verify live capabilities and use installed-version official documentation. Field observations in this package are troubleshooting hypotheses until confirmed on the target environment.

## Share/review checklist

Before sending a revision to anyone:

- Validate standard YAML frontmatter in `SKILL.md`.
- Confirm every linked reference exists.
- Search for people/customer/project names, private domains, email addresses, IPs, credentials, tokens, keys, cookies, internal paths, VM names, hostnames, IQNs, and WWIDs.
- Ensure examples use placeholders and contain no SSH wrappers, key paths, host loops, or real infrastructure identifiers.
- Ensure field observations are labeled and do not imply official product behavior.
- Run an agent pressure test where urgency and broad permission tempt it to make a change; verify it diagnoses read-only and asks for exact approval.
