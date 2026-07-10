# Incident triage patterns

Use this reference after applying the safety contract in `SKILL.md`. These are reusable diagnostic patterns, not universal product behavior. Verify every command and interpretation against the installed version and environment.

## Read-only discovery order

1. **Impact and scope:** appliance/control-plane or workload; one VM, host, datastore, network, or cluster-wide.
2. **Control plane:** UI/API reachability, identity, alerts, tasks, events, maintenance, placement, and reconciliation.
3. **Host/hypervisor:** expected placement, domain state, service health, time synchronization, and recent errors.
4. **Storage:** capacity/inodes, mount state, pool state, transport sessions, paths, identity, latency/timeouts, and I/O errors.
5. **Network:** VME network/segment, bridge or switch, VLAN, uplinks, guest MAC/IP, ARP/DHCP, firewall/security policy.
6. **Guest/application:** console/agent state, boot, filesystem capacity, network reachability, and a safe application check.
7. **Cross-layer comparison:** identify where UI/API, host, guest, and user-visible reality disagree.

Bound every log query with a time window, unit/source, line limit, and timeout. Record timezone. Redact before sharing.

## Result-to-action guide

| Observation | Interpretation to test | Safe next step |
|---|---|---|
| VM off; storage, quorum, and HA healthy | ordinary lifecycle failure or intentional policy | inspect tasks/events and placement; propose one managed start only after approval |
| VM starts then powers off | HA/reconciliation, policy, health check, storage heartbeat, or placement may be turning it off | stop repeated starts; correlate exact timestamps across control plane and host |
| VM shown in UI but absent on expected host | stale placement/control-plane record, migration, or missing domain definition | locate actual placement and preserve metadata/XML before any repair |
| VM may exist on two hosts | duplicate-instance and disk-corruption risk | hard stop; do not start either copy; escalate and establish ownership |
| Control plane unavailable; hosts/storage healthy | appliance outage, not necessarily workload outage | identify appliance placement and status; do not use host-side lifecycle without exact recovery approval |
| Datastore active but no VM disk uses it | availability and workload placement are separate facts | enumerate VM block devices and compare source paths; report “available but unused” |
| Pool inactive/missing on one host | mount/export reachability, pool autostart, or host-specific configuration | compare that host with a healthy peer; do not start pools or edit config without approval |
| Storage transport connected but no block/multipath device | presentation, ACL/LUN mapping, identity, or multipath issue | investigate storage presentation before blaming VME datastore logic |
| UI remains stale after lower layers recover | control-plane polling/reconciliation lag | time symptom and recovery at each layer; do not “fix” healthy lower layers to chase the UI |

## VM lifecycle and placement

- Prefer VME-managed lifecycle actions over raw `virsh` when the control plane is available.
- Before any start, verify exact VM identity, expected placement, no duplicate running instance, storage availability, quorum where relevant, HA/reconciliation state, and pending tasks.
- Do not loop starts or retries. One failed action should trigger evidence collection.
- If history shows a libvirt hook failure, inspect the named hook and dependencies before pursuing unrelated network or storage theories. Third-party hooks can block start, deploy, or move before the guest launches.
- For migrations or storage moves, compare the control-plane task with hypervisor block-job state, destination growth, guest reachability, and errors. A high copy percentage may still be active dirty-block convergence; do not cancel solely because progress appears slow.

## Storage patterns

Treat these as separate layers:

- capacity and inode availability;
- backing filesystem/export availability;
- host mount state;
- libvirt storage pool state;
- storage transport (NFS, iSCSI, FC, NVMe-oF, or other);
- multipath/device identity and path health;
- clustered filesystem/locking where used;
- actual VM disk placement;
- guest filesystem/application I/O.

“Available” at one layer does not prove healthy I/O at the next. A recovered transport can coexist with a withdrawn, read-only, stale, or failing filesystem.

If libvirt commands hang after a host reboot, check storage-pool autostart and blocking mount helpers before calling the hypervisor wedged. An unavailable NFS export or backing filesystem can block pool activation and libvirt calls. Use bounded process/service/journal checks and verify export reachability; configuration fixes require approval and rollback.

## Network and guest identity

- Do not trust only the UI IP field.
- Correlate VM placement, domain NIC MAC, network/segment, bridge or virtual switch, DHCP/ARP/router data, guest tools/agent, and console.
- Test LAN and overlay/VPN reachability separately; one can be healthy while the other is stale or unrouted.
- Do not change VLANs, bridges, bonds, firewall rules, or guest networking until the failing layer and blast radius are established.

## Read-only disk inspection

For a Linux VM disk, discover the owning host and disk path through managed inventory and/or libvirt. Prefer a powered-off VM, snapshot, or disk copy when consistency matters. If inspection of a running disk is unavoidable and approved, use guest-aware tooling or a read-only method designed for shared access.

Never:

- mount a running VM disk read-write;
- run filesystem repair during diagnosis;
- break a QEMU/libvirt lock;
- assume thin allocated bytes equal configured capacity;
- expose guest files or private data in the report.

## UI/login/authentication

Reproduce the browser-visible symptom first. Compare expected unauthenticated responses with actual behavior, then inspect read-only user/session/config and server logs. A `401` may be correct authentication enforcement; a `500` may indicate an application bug. Credential resets, database edits, or auth-table changes require approval, backup, and immediate browser verification.

## Controlled recovery verification

After an approved action, verify:

1. command/tool/API result and task completion;
2. control-plane object state;
3. host/hypervisor state;
4. storage/network prerequisites;
5. guest reachability and application check;
6. no duplicate instance, new alert, failed task, or unexpected HA action;
7. rollback no longer needed or remains available.

Report what was not verified. Do not equate an exit code of zero with recovery.
