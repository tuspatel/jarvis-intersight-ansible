# rack-claim-to-intersight-saas

Claims a standalone UCS C-Series server in Intersight using serial number and claim/security token.

## Host Groups

- `localhost` (API-only playbook)

## Required Inventory Vars

- `api_key_id` (or env: `INTERSIGHT_API_KEY_ID`)
- `api_private_key` (or env: `INTERSIGHT_API_PRIVATE_KEY`)

## Optional Inventory Vars

- `api_uri`
- `validate_certs` (default: `true`)

## Extra-Vars / Inputs

| Variable | Default | Description |
|---|---|---|
| `device_serial_number` | fallback from `device_id` | C-Series serial number |
| `device_claim_code` | fallback from `claim_code` | CIMC security token / claim code |
| `claim_resource_path` | `/api/v1/asset/DeviceClaims` | Intersight claim endpoint |

## Backward Compatibility

Legacy variable names are supported:
- `device_id` → `device_serial_number`
- `claim_code` → `device_claim_code`

## Exported Outputs

- `claim_submitted`
- `claim_changed`
- `device_serial_number`
- `claim_resource_path`

## Tags

- `validate`
- `claim`
- `intersight`
- `cseries`
- `outputs`
- `teardown`
- `always`

## Teardown

`blueprints/rack-claim-to-intersight-saas-teardown.yaml` is intentionally no-op and exports `teardown_status: no_op`.
