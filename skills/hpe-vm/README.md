# HPE VM Essentials / Morpheus VME Agent Onboarding Skill

Portable skill for teaching AI agents how to safely onboard to HPE VM Essentials / Morpheus VME.

Use [`SKILL.md`](./SKILL.md) as the agent-facing source of truth. It covers:

- read-only-first operating rules
- first-contact user questions
- no-secret token handling
- user guidance for creating an API key in VME
- version gating: MCP is expected only on VME / Morpheus 9.x and newer
- REST API as the primary path for older/pre-9 systems and fallback for 9.x+
- SSH as a last-resort, explicitly approved, read-only path
- output templates for health, inventory, and handoff summaries

## Recommended use

Give an agent the contents of [`SKILL.md`](./SKILL.md), then provide the VME appliance URL and the minimum local access details needed for read-only checks. Do not paste tokens, passwords, SSH keys, cookies, or raw headers into chat.

## Version guidance

| VME / Morpheus version | First path | Fallback path | Avoid |
|---|---|---|---|
| 9.x or newer | MCP at `/api/mcp` | REST API, then approved SSH | Mutation tools without explicit approval |
| Older than 9.x | REST API | Approved SSH | MCP; it is not expected to exist |
| Unknown | No-secret API probes first | Infer version, then choose MCP or API | Guessing that MCP exists |

## Safety boundary

This skill does not authorize infrastructure changes. Agents must not start, stop, restart, delete, resize, snapshot, backup, refresh, apply, reindex, acknowledge, or otherwise mutate VME resources unless the user separately approves the exact operation, target, and arguments.
