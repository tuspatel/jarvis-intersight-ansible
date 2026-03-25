# rack-configure-device-connector

Configure DNS NameServers and DomainName through Cisco CIMC Device Connector API.

## Host Groups

- Default: `Cisco_CSeries_servers`
- Override with `group` variable.

## Required Variables

- `server_ip`
- `user`
- `new_password`

## Optional Variables

- `group` (default: `Cisco_CSeries_servers`)
- `protocol` (default: `https`)
- `validate_certs` (default: `true`)
- `timeout` (default: `30`)
- `login_uri` (default: `/nuova`)
- `comm_configs_uri` (default: `/connector/CommConfigs`)
- `name_servers` (preferred CSV input, e.g. `8.8.8.8,1.1.1.1`)
- `domain_name` (preferred)
- Backward-compatible aliases also accepted:
  - `NameServers`
  - `DomainName`

## Behavior

1. Authenticates to CIMC (`/nuova`) and extracts `outCookie`.
2. Reads current communication config.
3. Updates `NameServers` and `DomainName`.
4. Verifies configuration endpoint is reachable post-update.
5. Exports outputs for downstream grains.

## Exported Outputs

- `dns_config_applied`
- `dns_config_status`
- `dns_verify_status`
- `server_ip`
- `configured_name_servers`
- `configured_domain_name`

## Tags

- `validate`
- `cimc`
- `auth`
- `precheck`
- `configure`
- `verify`
- `dns`
- `outputs`
- `teardown`
- `always`

## Teardown

`blueprints/rack-configure-device-connector-teardown.yaml` is intentionally no-op and exports `teardown_status: no_op`.
