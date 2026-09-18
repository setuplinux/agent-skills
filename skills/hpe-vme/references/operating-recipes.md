# Practical operating recipes

Apply the safety contract in `../SKILL.md` and the [version gates](version-capabilities.md). These are workflows, not an assertion that every edition exposes every feature. Use the live UI/schema/help for IDs, required fields and exact action names. An unavailable interface is a reason to try another authorized interface, not to demand broader credentials immediately.

## Route the request

| Goal | Start with | Prerequisites and approval | Completion evidence |
|---|---|---|---|
| Learn what the environment can do | Appliance version/edition, role, clusters, clouds, available layouts and actions | Existing authorized read access | Capability map labeled available, tested, unavailable, or unknown |
| Inventory VMs or find a guest | UI/API inventory, cluster nodes and VM placement | Bounded reads; include every relevant host, not just one | Managed record plus actual host/guest state where needed |
| Provision a VM | Managed provisioning wizard/API schema for the selected cloud/layout | Image, plan, datastore, network/IPAM, capacity, policy and exact deployment approval | Task completes, correct placement/disks/NICs, guest boots, intended access/app works |
| Start/stop/resize/clone a VM | Managed instance/VM action | Exact VM identity, health/HA/tasks, impact, backup/rollback and approval | Managed task and guest/application outcome; no duplicate or unintended HA action |
| Migrate VM or disk | Managed migrate/move operation supported by this layout | Destination capacity, compatible network/storage, task conflicts and approval | Placement or active disk source changes and guest app remains/re-becomes usable |
| Add storage or networking | Managed configuration; `hvmcli tui` was field-observed on inspected 9.1 hosts | Version-specific design, device identity/uplink/VLAN mapping, maintenance and recovery access | Every intended host agrees; workloads can actually use the resource |
| Run a container or Helm app | Discovered HKS cluster controls and Kubernetes API | Entitlement/layout, namespace, RBAC, image/chart access and approved manifest/values | Rollout, ready backends, HTTP/application response; see [HKS](hks-workloads-and-routing.md) |
| Investigate a failure | History/task details, alerts, bounded logs, then the failing layer | Read-only first; no repair under uncertain ownership | Evidence-backed cause/next action, not merely “service active” |

## VM provisioning: minimum useful input

1. Resolve the intended group/tenant, cloud/cluster, instance type/layout and service plan. Names are not IDs; select records through live inventory.
2. Confirm image version/OS, vCPU/RAM, each disk's size and datastore, network/VLAN/IPAM/DHCP, hostname and authorized guest-access mechanism. Check quotas, licensing and free capacity. Do not paste keys/passwords into reports.
3. Preview the UI form or current API request schema. Document the exact planned objects and charges/resource consumption. Do not submit as part of discovery.
4. After approval, submit once; retain the operation/task ID and poll boundedly. A transport success or an enclosing `success=true` is not proof of a usable VM.
5. Verify managed inventory, host placement, power state, actual VM disk paths, guest IP/agent state and intended application access. If it fails, inspect the specific History detail before retrying.
6. Cleanup is separately scoped: confirm resource ownership and disk/data-retention effects before deleting failed/temporary objects.

Distinguish a **managed VM/server**, an **Instance/application grouping**, and a **container host/cluster**. A Docker workload consumes an eligible container host; a Docker Cluster workflow provisions/registers one. Field-observed duplicate name/IP registration failures are not solved by deleting the original managed VM record or changing its IP to bypass checks. Verify supported reuse/conversion paths and compatible HVM layouts.

## Inventory and placement

Use management inventory first, then help-verified `hvmcli` reads or bounded libvirt evidence when the target's authorized access allows it:

```bash
virsh list --all --title
virsh domblklist <vm> --details
virsh domiflist <vm>
```

Report appliance/control-plane VMs separately from workloads. Include other placement hosts before claiming complete cluster inventory. Compare datastore mount paths to actual VM disk sources; an ISO/CD-ROM attachment is not a datastore-backed guest data disk. Configured virtual capacity and physically allocated sparse bytes are different values.

For stale guest IPs, compare VM MAC, DHCP/ARP/router data and guest-agent/console evidence. LAN and VPN reachability are independent. Avoid dumping guest files, entire domain XML or all secrets to solve an addressing question.

## Host network configuration

For an approved 9.1 configuration, use the discovered TUI rather than translating older OVS or `hpe-vm` recipes blindly. Before saving record intended management address/routes, physical uplink/bond, switch backend, VLAN/tagging, MTU and guest network mapping. Ensure out-of-band/console recovery survives loss of the management link.

After saving verify: configured switch -> kernel bridge/uplink -> guest tap/VLAN -> guest addressing -> target application. A reachable appliance does not prove the changed workload VLAN works. See [version-specific read-only network checks](version-capabilities.md#91-host-configuration-and-network-discovery).

## Storage and recovery decision boundaries

Use [clustered-datastore-gfs2.md](clustered-datastore-gfs2.md) for shared block identity, DLM, quorum and hard stops. Recovery starts from the failing layer:

- **iSCSI session but no usable multipath device:** investigate array/target presentation, initiator ACL/LUN mapping and WWID first. Do not format or mount a guessed device.
- **Healthy device paths but absent/stale mount:** compare filesystem, pool and cluster-locking state. A transport reconnect does not prove filesystem recovery.
- **Quorum/identity disagreement or duplicate VM risk:** stop mutations, preserve bounded evidence, establish ownership/fencing with the recovery owner/support. There is no generic safe repair command.
- **Appliance down with otherwise healthy hosts:** identify the appliance VM and its dependencies. If normal managed lifecycle is unavailable, propose the smallest host-side recovery explicitly; never start a second copy.
- **Healthy prerequisites and ordinary lifecycle fault:** propose one managed action with a rollback/verification plan. Do not loop starts while reconciliation keeps turning a VM off.

Changing a certificate, DNS/FQDN, proxy, NTP, host network or storage path can affect host-agent connectivity even if the browser login still works. Validate dependent services after a managed change; separate application gateway certificates from the appliance certificate and Kubernetes API certificate.

## Integration and tenant scope

Discover images/registries, SCM, backup, monitoring, DR and automation integrations without opening stored secret values. Existence of a connector does not prove an installed plug-in is current, tenant-aware, licensed, reachable or successfully protecting a workload. Check each provider's public support matrix and test the intended tenant/role. Successful VM provisioning is not proof of successful backup, DR or monitoring onboarding; verify downstream state and recovery capability separately.

## Report enough to act

For routine success: exact target, interface used, task/result, user-visible verification and remaining limits. For a failure: failed layer, minimal evidence, competing explanation, smallest next check/action, required approval and recovery boundary. Do not bury a usable result under a generic warning dump or teach an experienced operator their own established infrastructure.
