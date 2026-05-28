# HPE VM Essentials Agent Onboarding Skill

This skill helps AI agents safely onboard to HPE VM Essentials / Morpheus VME for read-only discovery, smoke testing, and capability mapping.
It is written for Claude, Codex-style agents, OpenClaw/Hermes-style agents, generic CLI assistants, and technical operators.

Use [`SKILL.md`](./SKILL.md) as the agent-facing source of truth.

## Status

Draft field-enablement skill. Not official HPE product documentation. Intended for internal review, lab validation, and safe agent onboarding experiments.

## Why this matters

This skill gives AI agents a safe, repeatable onboarding path for HPE VM Essentials / Morpheus VME.
It reduces the risk of agents guessing endpoints, mishandling credentials, calling mutation tools, or dumping customer-sensitive inventory.
The first use case is read-only smoke testing and inventory discovery for MCP-enabled VME environments, with REST and approved SSH fallback paths.

## Supported paths

- MCP for VME/Morpheus 9.x+.
- REST fallback when MCP is unavailable, incomplete, or not supported by the installed version.
- REST first for versions older than 9.x.
- Read-only SSH fallback only with explicit approval.

If exact MCP tool names, REST endpoints, or command names are unknown, the agent must discover available capabilities first instead of inventing them.

## What this skill refuses to do

The skill refuses to:

- request passwords, API tokens, private keys, bearer tokens, session cookies, or one-time codes in chat
- print secrets in logs, summaries, screenshots, examples, commits, or issue comments
- call create/update/delete/reboot/power/migrate/network/storage mutation tools without explicit approval
- dump full raw inventory or customer-sensitive details by default
- bypass the VME UI/API by manually changing host state unless explicitly approved
- use SSH as a shortcut around API authorization boundaries

## Quick install/use notes for generic agents

1. Put the `skills/hpe-vm/` directory where your agent reads portable skills, or provide the agent with the contents of `SKILL.md` directly.
2. Confirm the agent can parse standard YAML frontmatter with `name` and `description` fields.
3. Start in read-only mode.
4. Identify product, version, endpoint, and available access method.
5. Use MCP first for VME/Morpheus 9.x+ if available.
6. Use REST as fallback, or as the first path for older systems.
7. Use SSH only after explicit approval for read-only troubleshooting.
8. Require the mutation approval template before any state-changing action.

No client-specific runtime is required.
The skill does not assume shell, browser, MCP, persistent memory, or secret storage.
If those capabilities are missing, the agent should report the blocker and ask for the safest next approved path.

## Validation notes

Before publishing or sharing changes, verify:

- `SKILL.md` starts with valid YAML frontmatter.
- Frontmatter fields are on separate lines.
- Markdown headings, lists, and fenced code blocks are readable.
- Secret-handling rules are explicit.
- Mutation approval is explicit.
- MCP/REST/SSH version gating is clear.
- No examples contain real secrets, tokens, IPs, passwords, or customer data.
- Destructive words appear only in safety/refusal/approval guidance, not as runnable examples.
- A generic agent can understand the skill without HPE internal context.

Optional tooling, if available:

```bash
sed -n '1,20p' skills/hpe-vm/SKILL.md
head -n 5 skills/hpe-vm/SKILL.md
grep -RniE 'password|token|secret|apikey|api_key|bearer|private key' skills/hpe-vm/ || true
grep -RniE 'delete|remove|destroy|reboot|shutdown|poweroff|migrate|update|create' skills/hpe-vm/ || true
gh skill publish --dry-run
```
