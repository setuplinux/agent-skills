# HPE VM Essentials / Morpheus MCP Function Status

Purpose: quick field-reference table for agents/operators testing what VME/Morpheus MCP can actually do in the lab. This is based on the `ubuntu-mcp-test-01` validation run and should be updated as additional tools are tested.

Test context:

- Target: HPE VM Essentials / Morpheus VME MCP endpoint
- Test VM: `ubuntu-mcp-test-01`
- Instance ID: `3`
- Server ID: `6`
- Access path: MCP first, REST/UI only for read-only comparison when needed
- Safety rule: do not run mutation tools unless the operator approves the exact action and target

## Tested MCP functions

| Area | MCP function / pattern | Status | Verified effect / result | Notes for agents |
|---|---|---:|---|---|
| Tool discovery | category loader tools such as `use_instances_tools`, `use_servers_tools`, `use_backups_tools`, `use_storage_tools` | Works | Tool categories can be loaded and then expose many child tools. | Load the needed category in the same MCP session before describing/calling child tools. |
| Tool inventory | `list-tools` / MCP tool listing | Works | Inventory showed about 970 tools after loading common categories. | Treat inventory as version-specific; save a fresh listing for each appliance/build. |
| VM inventory | `list_instances`, `get_instance` | Works | Existing and newly-created VM details were readable. | Prefer limited summaries; avoid dumping full raw inventory into chat/comments. |
| Server inventory | `list_servers`, `get_server` | Works | Server backing the test VM was readable and showed provision/power state. | Useful for verifying instance-to-server mapping and power/provision status. |
| Cloud/group discovery | `list_clouds`, `list_groups` | Works | Cloud/group IDs were discoverable for payload construction. | Use discovery rather than hard-coding IDs across environments. |
| Image/catalog discovery | virtual image / layout / plan discovery patterns | Partially works | Existing image ID and layout/plan IDs could be derived enough to build the VM payload. | Some library endpoints can return license/feature errors; use known working objects from inventory when approved. |
| VM creation | `create_instance` | Works | Created `ubuntu-mcp-test-01`; final state verified as instance `running`, server `provisioned`, power `on`. | Requires carefully discovered payload: group, cloud, layout, plan, network, image, and config values. Exact approval required. |
| Instance snapshot | `snapshot_instance` | Works | Snapshot `id=1` reached `complete` and active state. | Verify after the call with snapshot read tools until a terminal status is reached. |
| Instance backup | `backup_instance` | Works / started | Created backup/job data and backup result `id=1`; latest read showed `IN_PROGRESS` on `disk01`. | Starting backup works; continue polling result/job tools to confirm final success/failure. |
| Backup inventory | `get_instance_backups`, backup result/job list patterns | Works | Backup result/job details were readable enough to track progress. | Record result IDs and result paths, not secrets or full raw payloads. |
| Datastore listing via MCP | `list_datastores` / `create_datastores` path family | Suspicious / needs more validation | MCP tool path appeared to target `/api/datastores`; direct REST inventory appeared to work at `/api/data-stores`. | Do not assume datastore mutation works through generated MCP wrappers until the API path mismatch is resolved. |
| Datastore creation | `create_datastores` | Not tested | No datastore was created. | Requires exact target/path/rollback approval; current path mismatch makes this unsafe to try blindly. |
| Extra disk by instance resize | `resize_instance` with added disk payload | Did not work | MCP returned generic failure similar to `Looks like the server threw a gasket`. | Schema/payload requirements are underspecified. Compare UI/REST payloads before retrying. |
| Standalone storage volume create | `create_storage_volume` | Did not work | MCP returned `error saving volume` for attempted 1GB volume payloads. | Failure means current MCP/payload path is not proven, not that VME cannot add storage. |
| Storage attach / server volume mutation | server volume attach/create patterns | Not proven | No working attach path was confirmed. | Treat as blocked pending schema discovery from UI/REST/docs and explicit approval. |
| Power lifecycle | start/stop/restart instance/server patterns | Not tested | No power action was needed after VM creation. | These are state-changing; require exact approval and rollback plan before calling. |
| Clone | clone instance pattern | Not tested | Not exercised. | Likely exposed but unvalidated; mutation approval required. |
| Delete/remove | delete instance/server/storage patterns | Not tested | Not exercised. | Destructive; require explicit approval and confirm target IDs twice. |
| Admin/security objects | user, role, credential, policy, integration create/update/delete patterns | Not tested | Not exercised. | High risk; do not use for capability testing unless specifically requested. |

## Current quick answer

MCP is not read-only: VM creation, snapshot, and backup execution were proven against a lab VM. Storage/datastore mutation is not yet reliable through the observed MCP wrappers/payloads and should be treated as blocked until schema/path issues are resolved.

## Update checklist for future agents

When adding rows or changing a status:

1. Include the exact tool/function pattern.
2. Mark the status as `Works`, `Partially works`, `Did not work`, `Not tested`, or `Suspicious / needs more validation`.
3. Add the verified effect, including safe object IDs when useful.
4. Add notes about approval, risk, fallback, or required follow-up.
5. Do not include tokens, passwords, bearer headers, session cookies, private keys, or full sensitive inventory dumps.
