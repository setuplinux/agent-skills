# Version and capability gates: 8.x, 9.0, and 9.1

Read this before selecting tools, interpreting services, configuring hosts, or proposing an upgrade. **9.1 is part of 9.x, but is a separate operational branch here.** Appliance version, HVM host release, HVM cluster layout, Kubernetes version, and add-on/chart versions are different identifiers. None is a substitute for the others.

Evidence labels: **Documented** = cited public product documentation for the stated scope; **Field-observed** = sanitized direct observations, not a vendor support promise; **Unverified** = resolve before relying on it. See [public sources](public-sources.md). Checked 2026-09-18; recheck release notes and target state before execution.

## Version comparison

| Topic | 8.x | 9.0-era | 9.1-era |
|---|---|---|---|
| Selecting procedures | Match the exact 8.x minor/build and actual layout. Do not backport a 9.x procedure by changing its version string. | Distinguish 9.0.0 VME documentation from later Enterprise manuals and host-layout changes. | Begin with fresh discovery; older host-configuration and network assumptions can be wrong. |
| Host configuration tool | **Documented in 8.0.11:** `hpe-vm` handles host bond configuration. Match installed help and guide. | **Documented in 9.0.0:** the networking guide still uses `hpe-vm`; do not project the 9.1 TUI backward to all 9.x. | **Field-observed:** `hvmcli tui` is the primary host-configuration entry point, replacing the older `hpe-vm` workflow on inspected 9.1 hosts. Confirm tool/build and version-matched guidance before applying. |
| Clustered-datastore HA | Establish actual layout/HA owner. Pacemaker-based layouts must not be treated as Agent-owned simply because a newer guide says so. | **Documented for VME 9.0.0:** layout 1.3 replaces 1.2; Morpheus Agent takes over resource/HA ownership previously handled by Pacemaker. Corosync remains part of the architecture. [Upgrade source](https://support.hpe.com/hpesc/public/docDisplay?docId=sd00008058en_us&docLocale=en_US&page=GUID-D678172F-CF40-43CE-9EC6-7CD47F9EDFD5.html). | **Unverified in this public guide:** layout 2.0 has been reported as an expected target, but its full behavior is not established here. Read the actual layout and supported architecture; do not relabel the 1.3 recovery procedure as 2.0. |
| General virtual-switch backend | **Documented in 8.0.11:** OVS bridges, Libvirt Networks and OVS port groups. Verify the actual switch. | **Documented in 9.0.0:** the same OVS model. OVS procedures apply only to confirmed OVS-backed switches. | **Field-observed:** inspected general switches use `linux-bridge` with VLAN filtering. OVS packages/services may remain installed without owning those switches. This does not describe every SDN/network type. |
| Host/appliance OS | Use the exact release's compatibility/install guide. | Do not generalize the appliance OS requirement to all host images/layouts. | **Field-observed, pre-release baseline:** host Ubuntu 26.04 and appliance Ubuntu 24.04 differed. This is not a supported-OS matrix or permission to upgrade the OS manually. |
| API, MCP and AI Services | **Documented in 8.0.11:** API/CLI access tokens, with `morph-api` for API access. No MCP evidence found in the reviewed 8.x sources; not a universal absence claim. | **Documented in 9.0.0 release notes:** AI Services/MCP management, read-only AI-agent mode/RBAC, and built-in MCP over Streamable HTTP or legacy SSE. REST remains available. | Re-discover advertised MCP tools/UI, RBAC and enabled state. Absence of an `mcp` host CLI command says nothing conclusive about HTTP MCP. No publicly established 9.1-specific API/MCP delta here. |
| Kubernetes/HKS and Helm | Verify edition, entitlement, layouts, target cloud and role; no release-wide availability claim here. | Public **Enterprise 9.0.2** docs describe Kubernetes workload/Service/kubeconfig controls and Git-backed Helm blueprints; they are not a VME entitlement matrix. | Similar controls were **field-observed** on a 9.1.0.1 appliance with HKS. Gateway API CRDs were present without a configured GatewayClass/route or identified gateway controller. That is an observation, not a universal 9.1 default. |

Sources: [VME 8.0.11 networking](https://support.hpe.com/hpesc/public/docDisplay?docId=sd00007027en_us&page=GUID-61426C77-F236-4AFF-A1E3-92893A1FAA9F.html), [8.0.11 API access](https://support.hpe.com/hpesc/public/docDisplay?docId=sd00007027en_us&page=GUID-74590319-12A6-4101-9043-8DDB56BB0720.html), [9.0.0 networking](https://support.hpe.com/hpesc/public/docDisplay?docId=sd00008058en_us&page=GUID-61426C77-F236-4AFF-A1E3-92893A1FAA9F.html), [9.0.0 release notes](https://support.hpe.com/hpesc/public/docDisplay?docId=sd00008079en_us).

The [8.1.2 manual](https://support.hpe.com/hpesc/public/docDisplay?docId=sd00007735en_us) describes Ubuntu 24.04 hosts with layout 1.2+ and Ubuntu 22.04 with layout 1.1. This is a historical version-scoped example, not permission to treat all 8.x clusters as layout 1.2. No applicable public 9.1 manual/release notes were found in this review; the 9.1 column is explicitly not a vendor support statement.

## Documented 9.0 capabilities worth discovering

The VME [9.0.0 release notes](https://support.hpe.com/hpesc/public/docDisplay?docId=sd00008079en_us) list these capabilities. Confirm prerequisites, edition, host agents, plugins, installed layout and exposed actions; a release-note item is not an instruction to enable it:

- **HA/storage:** two-node GFS2 clusters with an external quorum witness, stretched clusters, and shared virtual disks. Shared vDisks require guest/application coordination; they are not permission for multiple independent writers to an ordinary filesystem.
- **VM lifecycle/migration:** revised dynamic-placement profiles/cooldowns, VirtIO driver injection in bulk VMware-to-HVM migration, and Secure Boot live-migration support. Validate guest and host compatibility and the actual migration method.
- **Guest configuration:** post-deployment firmware/Secure Boot options, boot-to-BIOS, PXE/network boot, and vTPM/BitLocker-aware backup recovery. Firmware changes can prevent boot; protect recovery material and plan application downtime.
- **Capacity/networking:** memory overallocation, SR-IOV, and standard-network access/hybrid/trunk modes. These have capacity, physical-switch and mobility implications; inventory before choosing them.
- **Agent integration:** built-in MCP, AI-agent read-only mode/RBAC, and revised API-key management. Discover tool schemas and enforce least privilege on the target; labels alone are not authorization.

These are documented 9.0 feature entries, not a claim that every capability was absent from every 8.x build or is available in every edition. For a task workflow, use [operating-recipes.md](operating-recipes.md).

## Do not conflate product capabilities

- **VM Essentials / VME:** identify the licensed product and role before promising a feature.
- **Morpheus Enterprise:** shares concepts and APIs, but an Enterprise chapter does not prove the same feature is enabled or licensed in VME.
- **HVM:** the virtualization hosts and their cluster layout; not the same version axis as the appliance.
- **HKS:** Kubernetes clusters/workloads on supported infrastructure; Kubernetes RBAC and workload lifecycle are a separate layer from HVM VM lifecycle.
- UI visibility, API schema existence, license entitlement, authorization, and a successfully exercised workflow are separate facts. Report which ones were verified.

A menu item or shared Morpheus API schema is not proof that a cloud/layout supports the requested deployment. Do not promise backup/DR/monitoring tenant support from an integration name alone; check that integration's current product/version support and tenant scope.

## 9.0 layout 1.2 to 1.3: specific upgrade boundary

The public VME 9.0.0 upgrade chapter documents a managed, phased transition. This is **not** permission to stop or remove Pacemaker manually, nor a generic appliance upgrade procedure.

Preconditions: confirm the supported source build/layout, healthy shared mounts and old cluster, enough evacuation capacity, backups, maintenance window and recovery owner. Inspect the target's managed upgrade action; UI paths vary.

- Phase 1 updates host agents, uses rolling maintenance/evacuation and prepares Pacemaker resources. A failure can roll back the phase except for host-agent upgrades.
- Once Phase 1 succeeds and Phase 2 begins, rollback to Pacemaker/layout earlier than 1.3 is no longer available. Phase 2 transfers HA ownership and revalidates Corosync.
- Afterward verify the managed task, installed layout/agents, expected services, membership, DLM, shared mounts, VM placement and actual guest I/O on every relevant host.

For an 8.x target with unknown layout, the correct next step is read-only identification and a documented supported upgrade path, not executing this transition. For 9.1/layout 2.0, establish its own procedure and recovery boundary.

## 9.1 host configuration and network discovery

Use the installed `hvmcli tui` for the verified 9.1 host-configuration workflow; managed VM lifecycle still belongs to the appliance when available. A TUI can apply changes: inspect screens/help without saving during discovery, and obtain scoped approval before configuration.

Run from an already authorized host shell, with local command availability checked:

```bash
command -v hvmcli
hvmcli --help
hvmcli version
ip -d link show type bridge
bridge link show
bridge -j vlan show
```

Where installed help confirms them, use `hvmcli virtswitch list --json`, `hvmcli virtswitch status --json`, and `hvmcli network list --json`. Inspect/summarize VLAN JSON locally; large VLAN sets can flood the context. Do not install tools simply because a sample command is absent.

Compare configured switch backend, kernel bridge, physical uplink, guest tap, VLAN membership, addressing and routing. If a general switch is Linux-bridge-backed, do not apply `ovs-vsctl` mutation recipes. Conversely, one Linux bridge does not prove every switch is non-OVS. Package/service presence alone is insufficient.

A stale media README is evidence to reconcile, not permission to install a legacy tool. A fresh unenrolled host lacking agents, datastores or membership is not automatically a release regression.

## Capability discovery receipt

Record privately before selecting a recipe:

```text
Product/edition and appliance build:
HVM host release(s), enrollment state and cluster layout:
Document title/version/URL and evidence date:
Role and authorized interfaces:
Installed host CLI and help-verified operations:
Network backend for the target switch:
Storage transport, datastore type and current HA owner:
If HKS: Kubernetes version, CNI/CSI, add-ons, RBAC and routing controller:
Documented facts / live observations / unresolved assumptions:
```

If one layer is unknown, continue bounded discovery through another already authorized layer rather than inventing it. Stop before a change that depends on unresolved version or ownership facts.
