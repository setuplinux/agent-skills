# HKS workloads, Helm, and browser routing

## Applicability and source boundary

HKS/Kubernetes controls are not a promise about every VME license/build. The public [Enterprise 9.0.2 Kubernetes chapter](https://support.hpe.com/hpesc/public/docDisplay?docId=sd00008433en_us&page=GUID-3159C5BA-04D5-4035-9842-86197618F7D2.html) and [Helm Blueprints chapter](https://support.hpe.com/hpesc/public/docDisplay?docId=sd00008433en_us&page=GUID-E02DD704-C987-41BB-AF3F-A23F35C097BD.html) document the paths below. Similar surfaces were field-observed on a 9.1.0.1 appliance with HKS. Verify edition/entitlement, role, target cloud/layout and actual UI before promising access. No confidential reference-architecture material is included here.

## Choose GUI or CLI by the task, not by assumption

- `Infrastructure > Clusters > <cluster> > Actions > Run Workload`: Deployment, StatefulSet, DaemonSet or Job in a namespace. Inspect image/registry, command and custom-spec options before submitting.
- Cluster `Network`: Services, Endpoints, Ingress and Network Policies. A field-observed Add Service form accepts YAML; an empty Ingress list does not prove no ingress controller.
- Cluster `Control`: browser `kubectl`; `Packages` inventories add-ons. Packages are not necessarily Helm releases, and a GUI package label may differ from running image versions.
- `Actions > View Kube Config`: obtain Kubernetes API access without node SSH. This reveals a credential, not ordinary inventory: require authorization to retrieve it, save through a secure local mechanism with owner-only permissions, never print/commit it, and do not overwrite the client's default kubeconfig.
- `Access`: namespaces and RBAC; `Storage`: StorageClasses, PVCs/PVs, ConfigMaps and Secrets. Avoid opening secret values during routine discovery.
- `History`, events and workload status: examine exact failed step and namespace before blaming the entire cluster. Inventory can lag; compare with the live Kubernetes API.

Use an explicit protected kubeconfig path/context. Do not assume appliance read access grants cluster-admin, or request cluster-admin as the default fix for a missing permission.

### Bounded first Kubernetes checks

After authorized credential retrieval, set `KUBECONFIG` to the protected local file without displaying its contents. These checks require corresponding RBAC; a Forbidden response is not an empty cluster:

```bash
kubectl --request-timeout=15s get nodes -o wide
kubectl --request-timeout=15s get storageclasses
kubectl --request-timeout=15s get namespaces
kubectl --request-timeout=15s -n <namespace> get deployment,pod,service,endpointslice
kubectl --request-timeout=15s get gatewayclasses
kubectl --request-timeout=15s -n <namespace> get gateways,httproutes
kubectl --request-timeout=15s get ingressclasses
```

Use namespace/label filters and pagination when scale requires it. If Gateway resource types are missing, discover installed CRDs rather than treating that error as proof that nothing routes traffic. Check both the configuration objects and actual controller workloads. Do not execute `kubectl config view --raw` or dump Secrets into the transcript.

## First nginx application

Starter: [assets/hks-nginx.yaml](../assets/hks-nginx.yaml), a namespaced ConfigMap + Deployment + ClusterIP Service. It is a lab example, not a production-hardened application. Review the image version/digest, resource limits, namespace policy and security context against the target's requirements; one replica is not uninterrupted HA.

1. Discover target/context, namespace ownership, RBAC, registry egress, quotas, admission policy and any conflicting resource names. The sample owns `hks-demo`; choose a new dedicated namespace if that name exists. Do not adopt or overwrite someone else's objects.
2. Review the manifest and obtain approval for the exact namespace/resources. No changes are implied by reading this guide.
3. After approval, validate against the target API/admission policy and apply the reviewed file. Server-side dry-run still sends a write-shaped request and may invoke admission webhooks; treat it as part of the approved operation, not ordinary read-only discovery.
4. Verify rollout and backends, not just successful apply:

```bash
kubectl --request-timeout=15s -n hks-demo rollout status deployment/nginx-hello --timeout=180s
kubectl --request-timeout=15s -n hks-demo get pod,service,endpointslice -l app=nginx-hello
```

5. For a workstation-local browser test, an approved foreground `kubectl -n hks-demo port-forward --address=127.0.0.1 service/nginx-hello 8080:80` exposes only the client's loopback interface. Browse `http://localhost:8080` **on the machine running that command**; a remote agent's localhost is not the user's laptop. Stop only the specific forwarding process afterward.
6. For a LAN/VPN test, propose a NodePort Service or the existing shared gateway. NodePort needs an available assigned port and client-to-node reachability. Discover the allocated port instead of assuming a fixed number; `externalTrafficPolicy: Cluster` can route to a ready Pod on another node.
7. Verify the expected page content with HTTP and a real browser from the intended access network. Retain task/resource identifiers and the applied manifest privately.
8. Cleanup only after confirming the namespace is dedicated and all objects are owned by this test. Namespace deletion deletes everything in it, including later additions; do not use it as a casual rollback shortcut.

Deployment rollbacks restore Pod templates, not every change in a multi-object manifest. Keep previous manifests and consider ConfigMap content, Service changes and storage/data separately.

## Helm and repositories

Documented Enterprise path: integrate the Git repository, then `Library > Blueprints > App Blueprints > Add > Helm`; select repository, branch/tag and chart path; deploy the saved blueprint through `Provisioning > Apps`. Confirm the target cluster selection and edition/role support in the live wizard.

Three different things are often called a repository:

- **Image registry / Docker repository integration:** where container images and pull credentials live.
- **Git/SCM integration:** source files and chart directory used by the documented Helm Blueprint workflow.
- **HTTP/OCI Helm chart repository:** packaged charts, usable by Helm clients; do not assume an identically named Morpheus GUI integration exists.

A Helm dropdown or successful form save is not a tested deployment. If the UI cannot load chart configuration, inspect the repository integration, branch/path, permissions and error details. External Helm is an alternative with authorized kubeconfig/RBAC, not a reason to silently bypass organizational deployment policy. Pin chart/app versions; inspect rendered manifests, CRDs, cluster-wide RBAC, hooks, storage and secret handling before approval. Helm uninstall or rollback may leave CRDs/PVCs and cannot guarantee data recovery.

## A normal internal URL

DNS alone cannot remove a NodePort suffix. Proposed path:

```text
chosen hostname -> reserved private LAN IP:80/443
  -> gateway proxy/data-plane Service
  -> Gateway listener + HTTPRoute host/path match
  -> app Service -> ready Pod endpoints
```

[Gateway API](https://kubernetes.io/docs/concepts/services-networking/gateway/) separates `GatewayClass` (controller), `Gateway` (listeners/entry point) and `HTTPRoute` (app routing). **CRDs are definitions, not a running proxy.** The Kubernetes API/control-plane VIP is also not the application entry-point IP.

Discovery and selection:

1. Identify installed Ingress/Gateway controllers, classes, CRD bundle/channel, running images and any existing LoadBalancer implementation. Reuse a compatible supported implementation rather than installing competitors blindly.
2. Choose the controller against Kubernetes and Gateway API compatibility, required conformance/features, support lifecycle, HKS ownership and operator support requirements. Envoy Gateway and Traefik are candidates, not HPE-certified defaults here. See their current public [compatibility matrix](https://gateway.envoyproxy.io/news/releases/matrix/) and [provider requirements](https://doc.traefik.io/traefik/reference/install-configuration/providers/kubernetes/kubernetes-gateway/).
3. **Do not overwrite HKS-managed CRDs with a tutorial's latest or experimental bundle.** Render the exact chart; exclude provider-managed CRDs through its documented mechanism while including controller-specific CRDs when needed. A generic `--skip-crds` flag may not cover templated CRD dependencies.
4. Allocate a reachable entry-point IP. A `LoadBalancer` Service does not create an on-premises IP advertiser by itself. If appropriate and approved, [MetalLB L2](https://metallb.io/concepts/layer2/) is one option: reserve addresses outside DHCP/static conflicts, verify L2/network policy and platform compatibility, and account for one-node ingress bandwidth and failover. It is not the HTTP Gateway controller.
5. Create listeners and namespace-scoped routes. For a shared Gateway in another namespace, configure `allowedRoutes` to admit only approved namespaces. Keep each route's backend Service in the app namespace when practical; cross-namespace backend references need a suitable `ReferenceGrant`. See [cross-namespace routing](https://gateway-api.sigs.k8s.io/guides/multiple-ns/).
6. Configure operator-approved DNS and certificates separately. An internal IP is not itself access control: verify intended LAN/VPN routing/firewall scope, no public NAT/tunnel/IPv6 path, and keep admin endpoints private. Namespace separation alone is not a NetworkPolicy.
7. Verify current-generation GatewayClass/Gateway/HTTPRoute status (`Accepted`, `Programmed`, `ResolvedRefs` where applicable), assigned address, Service/EndpointSlices, then host-header HTTP and hostname/SNI/certificate-correct HTTPS from the intended client network. Typing only an IP may not match a hostname route. Keep the old NodePort during validation when safe; remove it only as a separately approved cleanup.

Do not install community **ingress-nginx** for a new deployment simply because old HKS screenshots show it. Kubernetes announced maintenance/security updates ending March 2026 in its [retirement notice](https://kubernetes.io/blog/2025/11/11/ingress-nginx-retirement/). This does not retire the nginx web-server application or every other NGINX-named controller. This guide does not establish a universal HPE 9.1 replacement controller.

## Persistent storage and operations

Discover the actual StorageClass/CSI, access mode, binding mode, expansion support, and each PV's reclaim policy before adding a PVC or deleting a chart. RWO is one-node access, not necessarily one-Pod access. A listed class does not prove a Pod can mount it. Ceph replication is not backup, and VM-backed replicas can share an underlying physical-storage failure domain. Verify binding, mount, guest/application writes, rescheduling and recovery in the approved test scope; a successful Pod recreation alone is not durability proof. See [Kubernetes persistent volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/).

Keep dashboards and diagnostic port-forwards private. For upgrades use the HKS managed action and exact version prerequisites; do not replace managed lifecycle with ad-hoc `kubeadm` changes. HVM host upgrades, appliance upgrades, HKS Kubernetes upgrades and controller/chart upgrades are separate operations requiring compatibility and disruption planning.
