# ansible/configure-firmware-policy/README.md

# configure-firmware-policy

Creates or updates an Intersight firmware policy for standalone rack servers.

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
| `organization` | `default` | Intersight organization |
| `firmware_policy_name` | `jv-firmware-policy` | Firmware policy name |
| `firmware_policy_description` | `Firmware policy for standalone servers` | Description |
| `target_platform` | `Standalone` | Target platform |
| `model_bundle_combo` | built-in default list | Firmware model family to bundle mapping |
| `deployment_type_tag` | `Jarvis` | Deployment tag value |
| `managed_by_tag` | `Jarvis` | managed_by tag |
| `environment` | `dev` | environment tag |
| `state` | `present` | Resource state |

## Exported Outputs

- `firmware_policy_name`
- `firmware_policy_org`
- `firmware_policy_state`
- `firmware_policy_changed`

## Tags

- `validate`
- `firmware`
- `policies`
- `intersight`
- `outputs`
- `teardown`
- `always`

## Teardown

`teardown.yaml` deletes the firmware policy with `state: absent`.