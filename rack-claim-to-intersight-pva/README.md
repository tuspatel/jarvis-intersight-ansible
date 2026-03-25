# claim-ucs-to-intersight-pva

Claims a standalone Cisco UCS C-Series server to an Intersight PVA appliance,
then validates completion through workflow API polling.

## Host Groups

- Default: `Cisco_CSeries_servers`
- Override with variable `group`.

## Required Variables

- `server_cimc_ip`
- `server_cimc_username`
- `server_cimc_password`
- `intersight_pva_ip`
- `api_private_key_path`
- `api_key` or `api_key_id`

## Optional Variables

- `group` (default: `Cisco_CSeries_servers`)
- `validate_certs` (default: `true`)
- `platform_type` (default: `IMC`)
- `claim_workflow_name` (default: `Device registration request`)
- `claim_workflow_status` (default: `COMPLETED`)
- `claim_workflow_retries` (default: `120`)
- `claim_workflow_delay` (default: `15`)
- `claim_workflow_top` (default: `1`)

## Behavior

1. Validates required inputs.
2. Calls `/api/v1/appliance/DeviceClaims` to submit claim.
3. Polls `/api/v1/workflow/WorkflowInfos` until workflow completion.
4. Exports claim/workflow outputs to Torque.

## Exported Outputs

- `claim_submitted`
- `server_cimc_ip`
- `intersight_pva_ip`
- `claim_changed`
- `workflow_match_count`
- `claim_workflow_completed`

## Tags

- `validate`
- `claim`
- `verify`
- `workflow`
- `intersight`
- `ucs`
- `outputs`
- `teardown`
- `always`

## Teardown

`blueprints/rack-claim-to-intersight-pva-teardown.yaml` is intentionally no-op and exports `teardown_status: no_op`.
