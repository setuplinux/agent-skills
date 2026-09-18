# Unofficial VME Field Guide for Agents

Independent, community-authored guidance for AI agents working with HPE VM Essentials / Morpheus VM Essentials. It is not official HPE documentation, is not endorsed or supported by HPE, and must not be presented as product policy. Product names belong to their respective owners.

The portable Agent Skills package is this directory:

```text
skills/hpe-vme/
├── SKILL.md
├── README.md
├── assets/
│   └── hks-nginx.yaml
└── references/
    ├── access-and-onboarding.md
    ├── clustered-datastore-gfs2.md
    ├── hks-workloads-and-routing.md
    ├── incident-triage.md
    ├── operating-recipes.md
    ├── public-sources.md
    └── version-capabilities.md
```

`SKILL.md` is the agent-facing entry point. Send the complete directory so its references remain available. The package uses standard `name` and `description` YAML frontmatter and does not require a specific agent runtime.

## Intended agents

- Codex and other coding/CLI agents
- Hermes and OpenClaw
- Claude or other Agent Skills-compatible assistants
- Technical operators using an agent as a read-only copilot

## What it enables

- Safe read-only onboarding and capability discovery
- Explicit 8.x / 9.0 / 9.1 guidance, with documented facts separated from field observations and unresolved layout behavior
- Practical VM provisioning/lifecycle, placement, migration and host-configuration recipes
- HKS GUI and Kubernetes API access without assuming node SSH
- Helm/Git/image-registry distinctions and a namespaced nginx starter
- Internal browser URLs through Services and Gateway API, with controller/IP/DNS/TLS kept distinct
- Web research and evidence analysis for Solutions Architects without target access
- Version-aware public-source research with clear source, supplied-evidence, hypothesis, and unexecuted-plan boundaries
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

Start with [version-capabilities.md](references/version-capabilities.md). The guide identifies the documented VME 9.0/layout 1.3 HA transition, field-observed 9.1 host CLI/Linux-bridge workflows, and where 8.x or layout 2.0 details still require evidence. It does not turn Enterprise documentation into VME entitlement claims or publish confidential source material.

Agent entry point: [SKILL.md](SKILL.md). For an ordinary task, use [operating-recipes.md](references/operating-recipes.md); for containers and application URLs, use [hks-workloads-and-routing.md](references/hks-workloads-and-routing.md). The nginx asset is a lab template; validate it against the target admission/image policy before any approved deployment.

## Share/review checklist

Before sending a revision to anyone:

- Validate standard YAML frontmatter in `SKILL.md`.
- Confirm every linked reference exists.
- Search for people/customer/project names, private domains, email addresses, IPs, credentials, tokens, keys, cookies, internal paths, VM names, hostnames, IQNs, and WWIDs.
- Ensure examples use placeholders and contain no SSH wrappers, key paths, host loops, or real infrastructure identifiers.
- Ensure field observations are labeled and do not imply official product behavior.
- Run an agent pressure test where urgency and broad permission tempt it to make a change; verify it diagnoses read-only and asks for exact approval.
