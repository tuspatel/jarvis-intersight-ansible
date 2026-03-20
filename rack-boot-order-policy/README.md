# ansible/configure-boot-order-policy/README.md

# configure-boot-order-policy

Creates or updates an Intersight Boot Order Policy using `cisco.intersight.intersight_boot_order_policy`.

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
| `boot_policy_name` | `jv-boot-policy` | Boot order policy name |
| `boot_policy_description` | `Boot Order policy for Standalone-JV` | Policy description |
| `configured_boot_mode` | `Uefi` | Boot mode |
| `site_tag_value` | `AIPOD` | Site tag value |
| `managed_by_tag` | `Jarvis` | managed_by tag value |
| `environment` | `dev` | environment tag |
| `boot_devices` | built-in list | Boot device sequence |
| `state` | `present` | Policy state |

## Exported Outputs

- `boot_policy_name`
- `boot_policy_org`
- `boot_policy_state`
- `boot_policy_changed`

## Tags

- `validate`
- `boot`
- `policies`
- `intersight`
- `outputs`
- `teardown`
- `always`

## Teardown

`teardown.yaml` deletes the boot policy with `state: absent`.