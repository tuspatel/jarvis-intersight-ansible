# rack-password-reset

Rack password reset grain for Cisco C-Series servers.
Designed to be production-ready and Torque/Jarvis-compatible.

## Host groups

- Default host group: `Cisco_CSeries_servers`
- Can be overridden using variable `group`

## Required variables

- `server_ip`
- `user`
- `default_password`
- `new_password`

## Optional variables

- `group` (default: `Cisco_CSeries_servers`)
- `protocol` (default: `https`)
- `validate_certs` (default: `true`)
- `account_id` (default: `1`)
- `accounts_uri` (default: `/redfish/v1/AccountService/Accounts`)
- `password_change_required_fallback` (default: `true`)

## Exported outputs

- `password_reset_performed`
- `redfish_query_status`
- `password_reset_status`
- `password_verify_status`
- `redfish_account_uri`

## Tags

- `validate`
- `redfish`
- `precheck`
- `reset`
- `verify`
- `outputs`
- `teardown`
- `always`

## Teardown

`blueprints/rack-password-reset-teardown.yaml` is intentionally no-op and exports `teardown_status: no_op`.
