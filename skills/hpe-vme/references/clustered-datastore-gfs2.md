# HPE Clustered Datastore, GFS2, DLM, iSCSI, and multipath

This reference captures reusable field observations. It is not a substitute for installed-version HPE documentation or support guidance. Cluster ownership and service expectations vary by layout and build.

## Investigate by layer

1. Product version, cluster layout, expected nodes, and HA owner.
2. Control-plane cluster/host/datastore/VM state, alerts, tasks, and events.
3. Corosync membership and quorum as seen by every expected node.
4. DLM lockspace membership.
5. GFS2 mount state and kernel/storage errors.
6. Storage transport sessions and target presentation.
7. Multipath map, shared WWID identity, and path health on every node.
8. Libvirt pool state and actual VM disk placement.
9. VM state, guest reachability, and safe application check.

Do not skip directly from “session connected” to filesystem repair.

## Read-only host gate

Run only after read-only SSH to the named hosts is approved. Check command availability first and bound potentially blocking commands with the local `timeout` utility when appropriate. Run the same block separately on each relevant node and label each result; do not publish SSH wrappers, key paths, private hosts, or loops.

```bash
corosync-quorumtool | grep -E 'Nodes:|Quorate:'
dlm_tool ls -n
findmnt -t gfs2
iscsiadm -m session
multipath -ll
virsh pool-list --all
virsh list --all --title
systemctl --no-pager status corosync dlm morpheus-morphd multipathd iscsid libvirtd
```

Service names and CLI flags vary. Treat this as a starting checklist, not universally valid copy/paste automation.

## Healthy invariants

- Every expected node agrees on cluster membership and reports quorate.
- DLM lockspace membership matches the intended clustered-datastore membership.
- The expected GFS2 filesystem is mounted where intended, without withdrawal/read-only/I/O symptoms.
- Every required host sees the intended storage sessions and the same shared device identity/WWID.
- Required multipath maps exist and retain sufficient active/ready/running paths for the design.
- Libvirt pools are active where expected and resolve to the intended datastore.
- VM disk paths correspond to the intended datastore.
- Control-plane placement/state, host state, guest state, and application checks agree, allowing for measured polling lag.

## Hard stops

Do not mount, unmount, repair, force, start workloads, alter paths, or change cluster services when:

- any node is non-quorate or nodes disagree on membership;
- DLM membership is unexpected or differs between nodes;
- hosts see different WWIDs for what should be the same shared LUN;
- a GFS2 unmount or block I/O is stuck;
- the filesystem is withdrawn, read-only, or reporting I/O errors;
- the same VM may be active on more than one host;
- HA/reconciliation is already moving or changing the affected VM;
- the installed layout and current HA owner are not understood.

Never use `systemctl stop corosync` as a casual reversible test on a node with an active GFS2 lockspace. Field testing has shown that this can strand DLM/GFS2 state and leave unmounts in uninterruptible sleep, potentially requiring node-level recovery.

Never force quorum or DLM membership based on a single node's view.

## Interpretation patterns

### iSCSI session exists, but `multipath -ll` is empty

A session proves login to a target, not presentation of a usable shared block device. Check LUN mapping, initiator ACLs, SCSI device discovery, stable device identity, and multipath configuration. Treat this as storage presentation/identity until evidence shows a VME datastore problem.

### Multipath differs between hosts

Compare target/LUN mapping and WWIDs. Do not create the clustered datastore or mount GFS2 until all intended hosts see the same shared devices with expected paths.

### GFS2 or unmount is stuck during reboot

Look for blocked GFS2 unmount, DLM recovery, or storage I/O rather than assuming a random boot problem. Recovery may require a controlled node restart/power action; that is a mutation requiring exact approval, impact review, and a post-recovery full health gate.

### Pacemaker is inactive or masked

Do not call that a failure without identifying the cluster layout. On some newer HVM layouts, the Morpheus Agent rather than Pacemaker owns HA decisions while Corosync/DLM/GFS2 remain relevant. Verify the installed-version architecture and expected services.

### `morpheus-morphd` or witness/worker connectivity is interrupted

Do not assume either no impact or immediate failover. Time control-plane, host, storage, workload, and GUI behavior. A narrow worker/witness outage may not surface promptly in the UI, while a host-agent outage can affect HA arbitration. Restore only through an approved action and verify every layer afterward.

### GUI disagrees with CLI/workload

Record four timestamps when possible:

- first lower-layer/workload symptom;
- first GUI/API symptom;
- lower-layer/workload recovery;
- GUI/API clear.

Report lag or mismatch; do not mutate healthy lower layers merely to make a stale UI change.

## Failure testing boundary

Production incident recovery and deliberate resilience testing are different modes. Do not inject failures without a written test scope, maintenance approval, workload protection, expected signals, abort criteria, recovery owner, and green pre-test gate. Test one failure at a time. Do not continue until all layers are healthy or a documented known lag is explicitly accepted.

## Post-change gate

After any approved storage/cluster recovery, verify on every expected node:

- membership and quorum;
- DLM lockspace;
- GFS2 mount and kernel errors;
- transport sessions;
- shared WWID and multipath paths;
- libvirt pool;
- VM placement and duplicate-instance absence;
- guest/application health;
- VME UI/API status, alerts, tasks, and events.

Preserve minimal redacted evidence and the exact approved action. Do not publish raw IQNs, WWIDs, paths, hostnames, IPs, VM names, or customer topology.
