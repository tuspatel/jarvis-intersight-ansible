---
name: jarvis-intersight-ansible-and-torque-skills
description: >
  Consolidated skill guidance for this repository. Use this file when working on
  jarvis-intersight-ansible automation workflows, Torque blueprint contracts,
  Quali/Torque catalog items, self-service designer experiences, Terraform module
  generation, and the end-to-end mapping between user inputs, blueprint grains,
  and Ansible or Terraform automation.
---

# Skills Catalog

This single file replaces the older split skill docs and combines:
- repository maintenance guidance for the `jarvis-intersight-ansible` workflows
- Torque and Quali catalog/blueprint design guidance

## Skill 1: Jarvis Intersight Ansible Maintainer

Use this skill for tasks in this repository that involve reviewing or changing any
`rack-*` workflow or related automation folder such as `fi-*`.

### Repo shape

- The repository is organized as top-level workflow folders, including:
  - `rack-boot-order-policy`
  - `rack-firmware-policy`
  - `rack-server-profile`
  - `rack-password-reset`
  - `rack-configure-device-connector`
  - `rack-claim-to-intersight-saas`
  - `rack-claim-to-intersight-pva`
  - `fi-claim-to-intersight-saas`
- Most workflow folders contain the same contract surface:
- Most workflow folders contain the same implementation surface:
  - `playbook.yaml`
  - `requirements.yaml`
  - `README.md`
- Workflow blueprints are centralized under `blueprints/` and named with their
  former parent directory as a prefix, for example
  `blueprints/rack-server-profile-Blueprint.yaml`.
- Workflow teardown playbooks are also centralized under `blueprints/` and named
  with their former parent directory as a prefix, for example
  `blueprints/rack-server-profile-teardown.yaml`.
- The top-level `README.md` is minimal, so treat each workflow `README.md` as the
  primary documentation source.
- The repository now also includes root support folders such as `blueprints/` and
  `.github/` for catalog-level assets and shared guidance.

### What to inspect first

For any workflow change, read these files in the target folder before editing:

1. `README.md` for supported inputs, outputs, and behavior.
2. `playbook.yaml` for variable names, defaults, tags, and validation logic.
3. The corresponding centralized teardown file under `blueprints/` to confirm
   whether destroy is real or intentionally no-op.
4. The corresponding file under `blueprints/` to confirm the Torque-facing contract.
5. `requirements.yaml` to confirm needed collections.

If the request spans multiple workflows, compare them for naming and output
consistency before making edits.

### Workflow families

- Intersight policy/profile workflows usually:
  - run on `localhost`
  - use `cisco.intersight.*` modules
  - define an `api_info` anchor with `api_private_key`, `api_key_id`, `api_uri`,
    `validate_certs`, and sometimes `state`
- Intersight claim workflows split into two patterns:
  - SaaS claim flow is API-oriented and typically runs on `localhost`
  - PVA claim flow is host-targeted, polls workflow endpoints, and exports workflow
    completion details
- Device-facing workflows usually:
  - target `Cisco_CSeries_servers` unless overridden by `group`
  - use `ansible.builtin.uri`
  - validate required host inputs in `pre_tasks`
  - perform an action, then verify with a follow-up request or assertion

### Working rules

- Keep the workflow contract aligned across `README.md`, `playbook.yaml`,
  the centralized teardown file under `blueprints/`, the centralized blueprint
  file under `blueprints/`, and `requirements.yaml`.
- When adding or renaming an input, update all places that declare, consume, or
  document it.
- When adding or renaming an output, update both the `set_fact` block and
  `torque.collections.export_torque_outputs`, then reflect it in `README.md` and
  the matching blueprint file under `blueprints/`.
- Preserve the existing style:
  - `pre_tasks` assertions for required input validation
  - `tasks` for the main action
  - `post_tasks` assertions as the final production gate
  - `no_log: true` on credential-bearing tasks
- Favor idempotent behavior and explicit success checks over silent best-effort
  changes.
- Keep tags meaningful and consistent with nearby workflows.
- Prefer variable-driven behavior and defaults over hardcoded environment-specific
  values so the same workflow can run both locally and through Torque.

### Torque contract rules for repo workflows

- In the centralized blueprint file under `blueprints/`, the Ansible grain
  `spec.inputs` section is passed to Ansible as extra-vars. Treat those names as
  the public interface.
- `inventory-file` is Torque-specific structured inventory content, so host groups,
  host vars, and group vars must match the playbook target and variable expectations.
- `torque.collections.export_torque_outputs` is how Ansible grains produce Torque
  outputs. It must run on `localhost`, and the exported keys must match the
  blueprint `outputs` list exactly.
- `requirements.yaml` should live in the workflow root so Torque can auto-install
  dependencies for that module.
- `on-destroy` mirrors the main Ansible grain structure. When changing teardown
  behavior, check source path, inventory, and teardown inputs together.
- Export tasks should remain tagged `always`, and functional tasks should keep
  meaningful coarse and fine-grained tags.

### Common patterns in this repo

- `pre_tasks` commonly enforce all required inputs up front with
  `ansible.builtin.assert`.
- Main work happens in `tasks`, followed by `set_fact` output shaping and
  `torque.collections.export_torque_outputs`.
- `post_tasks` act as the final production gate and should stay aligned with the
  success semantics of the workflow.
- Credential-bearing `uri` or Intersight tasks should retain `no_log: true`.
- Several workflows preserve backward-compatible variable aliases in addition to
  preferred names. Keep those aliases unless the request explicitly removes them.
- Some teardowns are real deletes, while others are intentionally no-op and only
  export teardown status.

### Important repo-specific caution

- Treat blueprint source paths as a contract that must be verified, not assumed.
- README titles and blueprint grain names do not always match folder names exactly.
  Use the implementation files as the source of truth.
- Torque auto-generated blueprints use placeholder outputs, but this repo maintains
  explicit output contracts by hand. Do not assume outputs can be inferred safely
  without checking the export task.

### Validation workflow

After edits, validate as much as the environment allows:

1. Run `ansible-playbook --syntax-check <workflow>/playbook.yaml`.
2. Run `ansible-playbook --syntax-check <centralized-teardown-path>`.
3. Re-read `README.md` and the matching centralized blueprint file to confirm every
   documented input and output still exists.
4. Use `rg` to find renamed variables or outputs that were missed in sibling files.
5. If the workflow is device-facing, confirm host-group defaults, connection style,
   and verification logic still match the README contract.
6. If the change touches Torque mapping, verify `spec.inputs`, `inventory-file`,
   `outputs`, and `on-destroy` still describe the same lifecycle as the playbooks.
7. If tags were changed, verify the export task still runs under filtered execution
   and the README still describes the tags meaningfully.

### Good outcomes

A complete change in this repo usually means:
- the playbook behavior is updated
- teardown behavior still matches the intended lifecycle
- Torque outputs stay stable or are intentionally revised everywhere
- the README matches the implementation
- the blueprint contract still points to the right automation entry points

## Skill 2: Torque Catalog Designer

Use this skill whenever the user wants to:
- create a service catalog item or blueprint in Torque or Quali
- build a guided deployment stepper or wizard for end users
- generate Terraform modules for VMaaS, BMaaS, or OVA-based appliance deployments
- map form inputs to Terraform variables
- templatize Cisco appliances such as FTD, FMC, Splunk, ISE, CSR, or Nexus
- produce Torque `spec_version: 2` YAML blueprints from a designer UI
- simplify DevOps complexity into self-service catalog experiences

Always trigger for requests that mention:
- `catalog`
- `blueprint`
- `Torque`
- `Quali`
- `VMaaS`
- `BMaaS`
- `OVA deployment`
- `Intersight automation`
- `self-service portal`
- `stepper wizard`
- `service template`
- `Cisco appliance deployment`
- `on-prem virtual application`
- `figma`
- `blueprint designer`
- `automation canvas`
- `hypervisor requirements`
- `virtual application requirements`
- `end-user workflow`
- `new solution`

### Architecture

```
Catalog Item Definition
    -> React Catalog Designer (UX)
       -> Blueprint Designer
       -> Automation Canvas
       -> Preview
    -> Terraform Modules (automation)
       -> ova_vsphere/
       -> vmaas_vsphere/
       -> bmaas_intersight/
```

### Immutable UI rules

#### Tab names
The three designer tabs must always be named exactly:
- `Blueprint Designer`
- `Automation Canvas`
- `Preview`

#### Offering type labels
Use exactly these labels:

| ID | Label | Icon | Color |
|---|---|---|---|
| `bmaas` | BMaaS | 🖥️ | #f59e0b |
| `ova` | On-Prem Virtual Application | 📦 | #6366f1 |
| `cloud` | Cloud Application | ☁️ | #0ea5e9 |
| `vmaas` | VMaaS | 💿 | #8b5cf6 |
| `custom` | Custom | ⚙️ | #10b981 |

#### Dark mode
The application is dark mode only.
Background: `#0f172a`
Surfaces: `#1e293b`
Borders: `#334155`

#### Section names in Blueprint Designer
- `Hypervisor Requirements`
- `Virtual Application Requirements`
- `End-user Workflow Steps`

### Workflow

#### Step 1 — Clarify the offering
Determine or infer:
- Offering type: On-Prem Virtual Application | VMaaS | BMaaS | Cloud Application | Custom
- Hypervisor: VMware 7.0 / 8.x or Nutanix AHV
- Cisco appliance or third-party application
- End-user steps and required fields
- Minimum CPU, memory, and storage
- OVA source: curated repo or BYO path

#### Step 2 — Generate React UI
Read:
- `references/ui-patterns.md`
- `references/ova-workflow.md`

Required views:
- Dashboard / Home
- `Blueprint Designer` tab
- `Automation Canvas` tab
- `Preview` tab

Implementation rules:
- single `.jsx` file
- dark mode only
- no external CSS files
- inline styles only

#### Step 3 — Generate Terraform modules
Read the relevant reference before writing `.tf`:
- `references/terraform/ova_vsphere.md`
- `references/terraform/vmaas_vsphere.md`
- `references/terraform/bmaas_intersight.md`

Generate per module:

```
modules/<offering_type>/
├── main.tf
├── variables.tf
├── outputs.tf
├── versions.tf
└── README.md
```

#### Step 4 — Generate Torque Blueprint YAML
- use `spec_version: 2`
- create one `grain` per Terraform module
- wire all inputs from `{{ .inputs.<form_key> }}`
- add `depends-on` where ordering matters
- expose meaningful outputs

#### Step 5 — Present deliverables
Output in this order:
1. React `.jsx`
2. Terraform module files
3. Torque blueprint `.yaml`
4. Wiring table: form key -> Terraform variable -> grain input

### Canonical OVA 3-step end-user workflow

When offering type is `ova`, default to:

#### Step 1 — Basic Information
- Deployment Name
- Description
- Location

#### Step 2 — Deployment Configuration
- vCenter / Hypervisor
- Cluster
- VM Name
- OVA Source
- Port Group
- Datastore
- CPU
- Memory GB

#### Step 3 — Review & Confirm
- Summary table
- Deploy action

### Variable naming convention

Form field keys should be `snake_case` and align with downstream Terraform variables.

#### vSphere
| Form Key | TF Variable | Notes |
|---|---|---|
| `deployment_name` | `deployment_name` | Human-readable deployment name |
| `vm_name` | `vm_name` | VM display name in vCenter |
| `vcenter` | `vcenter_server` | vCenter FQDN/IP |
| `cluster` | `cluster_name` | vSphere compute cluster |
| `datastore` | `datastore_name` | Target datastore |
| `network_interface` | `network_name` | Port group |
| `ovf_path` | `ovf_remote_url` | OVA source |
| `num_cpus` | `num_cpus` | vCPU count |
| `memory` | `memory_mb` | RAM in GB from form; TF multiplies by 1024 |
| `location` | `datacenter_name` | vSphere datacenter |

#### Intersight
| Form Key | TF Variable | Notes |
|---|---|---|
| `name` | `server_name` | Server profile name |
| `organization` | `org_name` | Intersight org name or moid |
| `target_platform` | `server_profile_template` | Profile template name |
| `server_assignment_mode` | `assignment_mode` | Static or Pool |

### Torque YAML structure reference

```yaml
spec_version: 2
description: "<catalog item description>"

inputs:
  <form_key>:
    type: string | numeric | boolean
    description: "<field label from designer>"

grains:
  <offering_grain_id>:
    kind: terraform
    spec:
      source:
        store: cisco-automation-repo
        path: modules/<offering_type>
      inputs:
        - vm_name: "{{ .inputs.vm_name }}"
      outputs:
        - resource_id
        - management_ip
        - status
    depends-on: []

outputs:
  deployment_id:
    value: "{{ .inputs.deployment_name }}"
  management_ip:
    value: "{{ .grains.<grain_id>.outputs.management_ip }}"
```

### Cisco appliance defaults

| Appliance | OVA Path | vCPU | RAM GB | Disk GB |
|---|---|---|---|---|
| FTD 7.4 | `curated://cisco/ftd-7.4.ova` | 4 | 8 | 50 |
| FMC 7.4 | `curated://cisco/fmc-7.4.ova` | 8 | 32 | 250 |
| Splunk 9.2 | `curated://splunk/splunk-9.2.ova` | 8 | 16 | 300 |
| ISE 3.3 | `curated://cisco/ise-3.3.ova` | 16 | 64 | 600 |
| CSR 1000v | `curated://cisco/csr-17.3.ova` | 4 | 8 | 16 |
| Nexus 9000v | `curated://cisco/n9kv-10.3.ova` | 4 | 8 | 10 |

### Dark mode color reference

```js
bg = "#0f172a"
surface = "#1e293b"
border = "#334155"
textPri = "#f1f5f9"
textSec = "#cbd5e1"
indigo = "#6366f1"
sky = "#0ea5e9"
amber = "#f59e0b"
violet = "#8b5cf6"
emerald = "#10b981"
```

### Quality checklist

- Tab names are exactly `Blueprint Designer`, `Automation Canvas`, and `Preview`
- Offering type label is `On-Prem Virtual Application`
- Section label is `End-user Workflow Steps`
- Dark mode colors are used consistently
- Form keys are `snake_case`
- Torque YAML uses `{{ .inputs.<key> }}`
- `depends-on` chains are valid
- Terraform variables and outputs are complete
- React UI is a single `.jsx` file with inline styles only
- OVA defaults follow the canonical 3-step workflow

### Reference files

- `references/terraform/ova_vsphere.md`
- `references/terraform/vmaas_vsphere.md`
- `references/terraform/bmaas_intersight.md`
- `references/ova-workflow.md`
- `references/ui-patterns.md`

## Combined guidance

Use both skill sets together when a request spans:
- repo workflow automation plus Torque blueprint authoring
- Ansible workflow changes plus catalog or service-template design
- Intersight automation exposed through a self-service Torque experience

When both apply:
1. Treat the Ansible workflow files as the implementation source of truth.
2. Treat the Torque blueprint as the public contract.
3. Keep variable names stable across playbooks, blueprint inputs, and exported outputs.
4. Validate both the automation behavior and the user-facing catalog shape before finishing.
