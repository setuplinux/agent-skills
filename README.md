# Agent Skills

Reusable, independent skills for AI agents and command-line assistants.

These are community-authored field guides. They are not official vendor documentation and must not be presented as vendor-authored, endorsed, certified, or supported. Product names belong to their respective owners. Current official documentation for the installed product version takes precedence.

## Available skills

| Skill | Purpose | Status |
|---|---|---|
| [`skills/hpe-vm`](./skills/hpe-vm/) | Portable VME agent field guide: web research/evidence analysis, safe onboarding, MCP/REST/approved SSH discovery, cross-layer incident triage, clustered-storage diagnostics, and controlled recovery | field draft |

## Design rules

- Use the portable Agent Skills structure: `skills/<name>/SKILL.md` plus optional `references/`, `scripts/`, and `assets/`.
- Default to bounded read-only discovery.
- State allowed, approval-required, and prohibited actions clearly.
- Never include credentials, secrets, people/customer/project names, private systems, raw inventory, or lab-only assumptions.
- Replace infrastructure details with generic placeholders.
- Label field observations and distinguish them from official version-matched behavior.
- Require exact approval, risk/blast-radius review, rollback/recovery, and verification before state changes.
- Keep client-specific installation details outside the portable core unless required.

## Security and privacy

Before sharing, scan every changed file for passwords, tokens, keys, cookies, authorization headers, emails, phone numbers, public/private IPs, private domains, usernames, customer identifiers, hostnames, VM names, IQNs, WWIDs, internal paths, screenshots, and raw logs. When in doubt, remove or generalize the detail.
