---
name: jarvis-intersight-ansible-maintainer
description: Use when working in the jarvis-intersight-ansible repo to add, review, or update the rack-* Ansible automation workflows for Cisco Intersight, Redfish, and CIMC device-connector operations, including playbooks, teardown flows, Blueprint.yaml contracts, requirements, Torque outputs, and per-workflow README documentation.
---

# Jarvis Intersight Ansible Maintainer

Use this skill for tasks in this repository that involve reviewing or changing any `rack-*` workflow.

## Repo shape

- The repository is organized as top-level workflow folders, including:
  - `rack-boot-order-policy`
  - `rack-firmware-policy`
  - `rack-server-profile`
  - `rack-password-reset`
  - `rack-configure-device-connector`
  - `rack-claim-to-intersight-saas`
  - `rack-claim-to-intersight-pva`
- Most workflow folders contain the same contract surface:
  - `playbook.yaml`
  - `teardown.yaml`
  - `Blueprint.yaml`
  - `requirements.yaml`
  - `README.md`
- The top-level [`README.md`](/Users/tuspatel/Desktop/Jarvis_Dev/Jarvis_intersight_ansible_Avinash/jarvis-intersight-ansible/README.md) is currently minimal, so treat each workflow's `README.md` as the primary documentation source.
- This repo uses root-level `rack-*` folders instead of the `ansible/<grain-name>/...` layout commonly shown in Jarvis examples. Preserve the repo's structure unless the user explicitly wants a repo-wide reorganization.

## What to inspect first

For any change, read these files in the target workflow folder before editing:

1. `README.md` for supported inputs, outputs, and behavior.
2. `playbook.yaml` for real variable names, defaults, tags, and validation logic.
3. `teardown.yaml` to see whether destroy is real or intentionally no-op.
4. `Blueprint.yaml` to confirm the workflow contract exposed to Torque.
5. `requirements.yaml` to confirm needed collections.

If the request spans multiple workflows, compare them for naming and output consistency before making edits.

## Workflow families

- Intersight policy/profile workflows (`rack-boot-order-policy`, `rack-firmware-policy`, `rack-server-profile`) usually:
  - run on `localhost`
  - use `cisco.intersight.*` modules
  - define an `api_info` anchor with `api_private_key`, `api_key_id`, `api_uri`, `validate_certs`, and `state`
  - support organization, object naming, descriptive fields, and environment/tag defaults
- Intersight claim workflows split into two patterns:
  - SaaS claim flow is API-oriented and typically runs on `localhost`
  - PVA claim flow is host-targeted, polls workflow endpoints, and exports workflow completion details
- Device-facing workflows (`rack-password-reset`, `rack-configure-device-connector`) usually:
  - target `Cisco_CSeries_servers` unless overridden by `group`
  - use `ansible.builtin.uri`
  - validate required host inputs in `pre_tasks`
  - perform an action, then verify with a follow-up request or assertion

## Working rules

- Keep the workflow contract aligned across `README.md`, `playbook.yaml`, `teardown.yaml`, and `Blueprint.yaml`.
- When adding or renaming an input, update all places that declare, consume, or document it.
- When adding or renaming an output, update both the `set_fact` block and `torque.collections.export_torque_outputs`, then reflect it in `README.md` and `Blueprint.yaml`.
- Preserve the existing style:
  - `pre_tasks` assertions for required input validation
  - `tasks` for the main action
  - `post_tasks` assertions as the final production gate
  - `no_log: true` on credential-bearing tasks
- Favor idempotent behavior and explicit success checks over silent best-effort changes.
- Keep tags meaningful and consistent with nearby workflows.
- Prefer variable-driven behavior and defaults over hardcoded environment-specific values so the same workflow can run both locally and through Torque.

## Torque contract rules

- In `Blueprint.yaml`, the Ansible grain `spec.inputs` section is passed to Ansible as extra-vars. Treat those names as the public workflow interface.
- `inventory-file` is Torque-specific structured inventory content, so host groups, host vars, and group vars must match the playbook's `hosts:` target and variable expectations.
- `torque.collections.export_torque_outputs` is how Ansible grains produce Torque outputs. It must run on `localhost`, and the exported keys must match the `Blueprint.yaml` `outputs` list exactly.
- `requirements.yaml` or `requirements.yml` should live in the workflow root so Torque can auto-install dependencies for that module.
- `on-destroy` mirrors the main Ansible grain structure. When changing teardown behavior, check source path, inventory, and teardown inputs together rather than editing only `teardown.yaml`.
- Command-argument driven tag filtering is part of the Torque execution model, so output export tasks should remain tagged `always`, and functional tasks should keep useful coarse and fine-grained tags.

## Common patterns in this repo

- `pre_tasks` commonly enforce all required inputs up front with `ansible.builtin.assert`.
- Main work happens in `tasks`, followed by `set_fact` output shaping and `torque.collections.export_torque_outputs`.
- `post_tasks` act as the final production gate and should stay aligned with the success semantics of the workflow.
- Credential-bearing `uri` or Intersight tasks should retain `no_log: true`.
- Several workflows preserve backward-compatible variable aliases in addition to preferred names. Keep those aliases unless the request explicitly removes compatibility.
- Some teardowns are real deletes (`state: absent`), while others are intentionally no-op and only export teardown status. Confirm which pattern applies before changing lifecycle behavior.
- README docs should describe expected host groups, required variables, optional variables, exported outputs, tags, and teardown behavior because Torque authors rely on that contract when wiring blueprints.
- Defaults matter: workflow inputs should usually have sensible values so playbooks remain easy to test locally and in CI, while still allowing Torque to override them through `spec.inputs` or `inventory-file`.

## Important repo-specific caution

- `Blueprint.yaml` files currently reference `ansible/...` source paths, while this repository is organized with `rack-*` directories at the root. Treat those paths as a contract that must be verified, not assumed to be correct.
- Before changing any `Blueprint.yaml`, check whether the `ansible/...` path is an external packaging convention or an actual mismatch that should be corrected across the repo.
- README titles and Blueprint grain names do not always match folder names exactly. When updating docs or paths, use the implementation files as the source of truth rather than assuming names are normalized.
- Torque auto-generated blueprints use placeholder outputs, but this repo maintains explicit output contracts by hand. Do not assume Blueprint outputs can be inferred safely from the playbook without checking the export task.

## Validation workflow

After edits, validate as much as the environment allows:

1. Run `ansible-playbook --syntax-check <workflow>/playbook.yaml`.
2. Run `ansible-playbook --syntax-check <workflow>/teardown.yaml`.
3. Re-read `README.md` and `Blueprint.yaml` to confirm every documented input/output still exists.
4. Use `rg` to find renamed variables or outputs that were missed in sibling files.
5. If the workflow is device-facing, confirm host-group defaults, connection style, and verification logic still match the README contract.
6. If the change touches Torque mapping, verify `spec.inputs`, `inventory-file`, `outputs`, and `on-destroy` still describe the same lifecycle as the playbooks.
7. If tags were changed, verify the export task still runs under filtered execution and the README still describes the tags meaningfully.

If a change affects multiple workflow folders, syntax-check each touched playbook.

## Good outcomes

A complete change in this repo usually means:

- the playbook behavior is updated
- teardown behavior still matches the intended lifecycle
- Torque outputs stay stable or are intentionally revised everywhere
- the README matches the implementation
- the Blueprint contract still points to the right automation entry points

## Typical requests this skill should help with

- Add a new Intersight policy workflow
- Update input variables or defaults for an existing rack workflow
- Fix Torque output naming drift
- Review a workflow for broken Blueprint mappings or documentation mismatch
- Add validation gates to a playbook
- Align workflow README content with the implemented playbook behavior
