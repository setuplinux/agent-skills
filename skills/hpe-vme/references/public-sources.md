# Public sources and verification boundary

Last reviewed: 2026-07-10

Use the documentation for the installed product version. Deep links can move; if one no longer resolves, start from the HPE Support Center and search by document ID or page title.

## Public starting points

- Compatibility Matrix for HPE Morpheus VM Essentials Software: https://support.hpe.com/hpesc/public/docDisplay?docId=sd00006551en_us&page=GUID-EA7C0803-E66B-4B17-B994-30D4025A258F.html
- VME 9.0.0 HVM Cluster Layout Upgrade: https://support.hpe.com/hpesc/public/docDisplay?docId=sd00008058en_us&docLocale=en_US&page=GUID-D678172F-CF40-43CE-9EC6-7CD47F9EDFD5.html
- HVM High Availability Failover Methodology: https://cdn.support.hpe.com/hpesc/public/docDisplay?docId=sf000111669en_us&docLocale=en_US
- HPE Support Center: https://support.hpe.com/

These links are references, not an assertion that every feature or procedure applies to every release.

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
