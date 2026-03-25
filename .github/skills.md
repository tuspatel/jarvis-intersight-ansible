# Skills Catalog

This file merges the repository automation skill guidance from `.github/SKILL.md`
with the Torque catalog and blueprint design guidance from `.github/torque-skill.md`.

## Skill 1: Jarvis Intersight Ansible Maintainer

### Use this when
- Working in this repository on any `rack-*` or related workflow
- Updating `playbook.yaml`, `teardown.yaml`, `Blueprint.yaml`, `requirements.yaml`, or `README.md`
- Fixing Torque outputs, input mappings, or workflow contract drift

### Core repo rules
- Keep `README.md`, `playbook.yaml`, `teardown.yaml`, `Blueprint.yaml`, and `requirements.yaml` aligned
- Preserve `pre_tasks` validation, main `tasks`, output export, and `post_tasks` production gates
- Keep credential-bearing tasks under `no_log: true`
- Preserve backward-compatible variable aliases unless intentionally removing them
- Keep Torque outputs synchronized between `set_fact`, export task, and `Blueprint.yaml`

### Validation checklist
1. Run `ansible-playbook --syntax-check` for touched playbooks where dependencies allow.
2. Re-read `README.md` and `Blueprint.yaml` for input/output parity.
3. Use `rg` to catch missed renamed variables or outputs.
4. Confirm teardown behavior still matches the intended lifecycle.

## Skill 2: Torque Catalog Designer

### Use this when
- Building a Torque/Quali catalog item or blueprint
- Designing a self-service wizard, stepper, or blueprint designer UX
- Generating Terraform modules for OVA, VMaaS, or BMaaS
- Mapping end-user form fields to Terraform variables and Torque grain inputs

### Non-negotiable UI rules
- Tabs must be exactly `Blueprint Designer`, `Automation Canvas`, and `Preview`
- Use `On-Prem Virtual Application` as the OVA offering label
- Keep the experience dark mode only
- Use the section names `Hypervisor Requirements`, `Virtual Application Requirements`, and `End-user Workflow Steps`

### Delivery workflow
1. Clarify the offering type, platform, workflow steps, and sizing.
2. Generate the React UI as a single `.jsx` file with inline styles.
3. Generate Terraform modules with `main.tf`, `variables.tf`, `outputs.tf`, `versions.tf`, and `README.md`.
4. Generate a Torque `spec_version: 2` blueprint with inputs wired from `{{ .inputs.<key> }}`.
5. Provide a wiring table from form keys to Terraform variables and grain inputs.

### Torque blueprint rules
- One grain per deployable module
- Use meaningful outputs
- Add `depends-on` only where required
- Keep form keys in `snake_case`

## Combined guidance

Use both skill sets together when a request spans:
- Repo workflow automation plus Torque blueprint authoring
- Ansible workflow changes plus catalog or service-template design
- Intersight automation exposed through a self-service Torque experience

When both apply:
1. Treat the Ansible workflow files as the implementation source of truth.
2. Treat the Torque blueprint as the public contract.
3. Keep variable names stable across playbooks, blueprint inputs, and exported outputs.
4. Validate both the automation behavior and the user-facing catalog shape before finishing.
