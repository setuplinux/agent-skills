# Public sources and verification boundary

Updated: 2026-09-18. Version claims and new workload sources were reviewed; historical starting links are not guarantees of current applicability or availability.

Use the documentation for the installed product version. Deep links can move; if one no longer resolves, start from the HPE Support Center and search by document ID or page title.

## Public starting points

- Compatibility Matrix for HPE Morpheus VM Essentials Software: https://support.hpe.com/hpesc/public/docDisplay?docId=sd00006551en_us&page=GUID-EA7C0803-E66B-4B17-B994-30D4025A258F.html
- VME 9.0.0 HVM Cluster Layout Upgrade: https://support.hpe.com/hpesc/public/docDisplay?docId=sd00008058en_us&docLocale=en_US&page=GUID-D678172F-CF40-43CE-9EC6-7CD47F9EDFD5.html
- HVM High Availability Failover Methodology: https://cdn.support.hpe.com/hpesc/public/docDisplay?docId=sf000111669en_us&docLocale=en_US
- HPE Support Center: https://support.hpe.com/

These links are references, not an assertion that every feature or procedure applies to every release.

## Version-scoped and workload sources

- **VME 8.0.11 networking:** https://support.hpe.com/hpesc/public/docDisplay?docId=sd00007027en_us&page=GUID-61426C77-F236-4AFF-A1E3-92893A1FAA9F.html — `hpe-vm`, OVS bridges, Libvirt Networks and port groups.
- **VME 8.0.11 API access:** https://support.hpe.com/hpesc/public/docDisplay?docId=sd00007027en_us&page=GUID-74590319-12A6-4101-9043-8DDB56BB0720.html — API/CLI tokens and `morph-api`.
- **VME 8.1.2 manual:** https://support.hpe.com/hpesc/public/docDisplay?docId=sd00007735en_us — historical HVM provisioning guidance distinguishes layouts 1.1 and 1.2+; not all 8.x deployments use one layout.
- **VME 9.0.0 release notes:** https://support.hpe.com/hpesc/public/docDisplay?docId=sd00008079en_us — layout 1.3, AI-agent read-only mode/RBAC, built-in MCP, two-node witness and other HVM features. Inspect prerequisites rather than treating a bullet as deployment approval.
- **VME 9.0.0 manual/networking:** https://support.hpe.com/hpesc/public/docDisplay?docId=sd00008058en_us&page=GUID-61426C77-F236-4AFF-A1E3-92893A1FAA9F.html — still documents `hpe-vm` and OVS.
- **VME 9.0.2 manual:** https://support.hpe.com/hpesc/public/docDisplay?docId=sd00008401en_us — distinct from Enterprise 9.0.2 (`sd00008433en_us`). Enterprise 9.0.0 release notes (`sd00008051en_us`) are likewise not the VME 9.0.0 manual.
- **Enterprise 9.0.2 Kubernetes Clusters:** https://support.hpe.com/hpesc/public/docDisplay?docId=sd00008433en_us&page=GUID-3159C5BA-04D5-4035-9842-86197618F7D2.html — Run Workload, View Kube Config, Control/kubectl, Services/Ingress, storage and history. Its HA load-balancer caveat concerns the Kubernetes API/control-plane endpoint, not an application ingress recommendation. Older external-cluster service-account examples and network requirements must be reconciled with the actual Kubernetes release; do not blindly copy cluster-admin grants or secret-token extraction commands.
- **Enterprise 9.0.2 Helm Blueprints:** https://support.hpe.com/hpesc/public/docDisplay?docId=sd00008433en_us&page=GUID-E02DD704-C987-41BB-AF3F-A23F35C097BD.html — integrated Git repository, branch/tag/chart path and Provisioning > Apps. Does not establish HTTP/OCI Helm-repository UI support in VME.
- **Kubernetes Gateway API:** https://kubernetes.io/docs/concepts/services-networking/gateway/ — resource/controller model, listeners and routes; not HPE support certification.
- **Cross-namespace Gateway routes:** https://gateway-api.sigs.k8s.io/guides/multiple-ns/ — explicit route attachment permissions.
- **Ingress NGINX retirement:** https://kubernetes.io/blog/2025/11/11/ingress-nginx-retirement/ — community ingress-nginx maintenance ends March 2026; not retirement of nginx application containers or every NGINX controller.
- **Envoy Gateway compatibility:** https://gateway.envoyproxy.io/news/releases/matrix/ — select a maintained release matching Kubernetes/Gateway API rather than hardcoding this guide's research-time versions.
- **Traefik Gateway provider:** https://doc.traefik.io/traefik/reference/install-configuration/providers/kubernetes/kubernetes-gateway/ — version-specific support/requirements; latest documentation can outpace HKS-managed CRDs.
- **MetalLB Layer 2:** https://metallb.io/concepts/layer2/ — ARP/NDP advertisement, single-node ingress and failover; no automatic HTTP routing or private-network access control.
- **Kubernetes persistent volumes:** https://kubernetes.io/docs/concepts/storage/persistent-volumes/ — access modes, binding and reclaim policies; no implication of HKS storage entitlement/defaults.

For the claim-by-claim 8.x/9.0/9.1 boundary see [version-capabilities.md](version-capabilities.md). Unsourced release-wide claims are deliberately left unverified. A 9.1 field observation is not a substitute for a public support matrix.

## Source hierarchy

For product behavior and supported operations, prefer:

1. Documentation and release notes matching the installed VME version/build.
2. HPE support advisories and an active support case.
3. Live UI/API capability discovery on the target appliance.
4. Command help and service/package information on the target host.
5. This skill's field guidance as a troubleshooting hypothesis.

When sources conflict, stop and surface the conflict. Do not silently combine commands or architecture assumptions from different VME/HVM releases.

## MCP and emerging feature boundary

MCP/AI Services behavior can change by build, role, configuration, and plugin state. Do not treat this skill's mention of MCP as official confirmation that a specific appliance supports it. Verify through the target appliance UI, live tool discovery, release notes, and current official documentation. Do not infer an in-product feature from an “Ask AI” control on a documentation website.

## Contribution and sanitization rule

Only reusable, publicly safe knowledge belongs in this package. Before adding field observations:

- remove people, customer, partner, tenant, account, case, and project names;
- remove private/public IPs, domains, email addresses, phone numbers, URLs to private systems, credentials, tokens, keys, cookies, and headers;
- replace VM, host, cluster, datastore, network, IQN, WWID, path, and topology details with generic placeholders;
- remove screenshots and raw outputs unless fully sanitized and necessary;
- describe behavior as “field-observed” unless verified in public version-matched documentation;
- avoid wording that implies HPE authorship, endorsement, support, certification, or product policy;
- retain enough safety context that another agent cannot turn an observation into an unsafe universal command.

Do not copy, paraphrase for publication, or attach confidential/partner-only/draft documents, screenshots, excerpts or proprietary deployment details just because names have been removed. Independently substantiate publishable claims from public documentation or authorized sanitized firsthand observations. Keep private source archives and environment runbooks outside this repository.
