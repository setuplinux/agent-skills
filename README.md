# Agent Skills

Reusable skills for AI agents and command-line assistants.

## Available skills

| Skill | Purpose | Status |
|---|---|---|
| `skills/hpe-vme-mcp` | Read-only HPE VM Essentials MCP discovery and inventory | v0.1 draft |

## Design rules

- Skills should be safe by default.
- Skills should avoid secrets, tokens, customer names, and lab-only assumptions.
- Skills should include clear allowed and blocked actions.
- Skills should give agents exact workflows and output formats.
- Skills should separate reusable instructions from field notes and test evidence.

## Security note

Never commit MCP tokens, API tokens, screenshots containing tokens, raw headers, private customer data, or full inventory JSON unless it has been reviewed and sanitized.
