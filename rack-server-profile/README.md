# rack-server-profile

Creates or updates an Intersight server profile for standalone rack servers.

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
| `server_profile_name` | `SP-Server1` | Server profile name |
| `target_platform` | `Standalone` | Platform type |
| `profile_description` | `Profile for JV standalone server` | Profile description |
| `assigned_server` | `5e3b517d6176752d319a9999` | Assigned server Moid |
| `boot_order_policy` | `jv-boot-policy` | Boot order policy name |
| `firmware_policy` | `jv-firmware-policy` | Firmware policy name |
| `site_tag_value` | `AIPOD` | Site tag |
| `managed_by_tag` | `Jarvis` | managed_by tag |
| `environment` | `dev` | environment tag |
| `state` | `present` | Resource state |

## Exported Outputs

- `server_profile_name`
- `server_profile_org`
- `server_profile_state`
- `server_profile_changed`
- `assigned_server`

## Tags

- `validate`
- `profile`
- `server`
- `intersight`
- `outputs`
- `teardown`
- `always`

## Teardown

`blueprints/rack-server-profile-teardown.yaml` deletes the server profile with `state: absent`.
