# fi-claim-to-intersight-saas

Claims a Cisco UCS Fabric Interconnect in Intersight using serial number and claim/security token.

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
| `fi_serial_number` | fallback from `device_serial_number` or `device_id` | Fabric Interconnect serial number |
| `fi_claim_code` | fallback from `device_claim_code` or `claim_code` | FI claim/security token |
| `claim_resource_path` | `/api/v1/asset/DeviceClaims` | Intersight claim endpoint |

## Backward Compatibility

Legacy/generic variable names are supported:
- `device_serial_number` or `device_id` -> `fi_serial_number`
- `device_claim_code` or `claim_code` -> `fi_claim_code`

## Exported Outputs

- `claim_submitted`
- `claim_changed`
- `fi_serial_number`
- `claim_resource_path`

## Tags

- `validate`
- `claim`
- `intersight`
- `fi`
- `outputs`
- `teardown`
- `always`

## Example

```bash
ansible-playbook fi-claim-to-intersight-saas/playbook.yaml \
  -i localhost, \
  -e api_key_id="$INTERSIGHT_API_KEY_ID" \
  -e api_private_key="$INTERSIGHT_API_PRIVATE_KEY" \
  -e fi_serial_number="FCH1234ABC1" \
  -e fi_claim_code="YOUR_CLAIM_CODE"
```

## Teardown

`blueprints/fi-claim-to-intersight-saas-teardown.yaml` is intentionally no-op and exports `teardown_status: no_op`.
