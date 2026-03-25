---
name: torque-catalog-designer
description: >
  Full-stack Torque/Quali catalog item builder. Use this skill whenever the user wants to:
  create a service catalog item or blueprint in Torque/Quali; build a guided deployment
  stepper/wizard for end users; generate Terraform modules for VMaaS (vSphere), BMaaS
  (UCS/Intersight), or OVA-based appliance deployments; map form inputs to Terraform
  variables; templatize Cisco appliances (FTD, FMC, Splunk, ISE, CSR); produce
  Torque spec_version:2 YAML blueprints from a designer UI; or bridge DevOps complexity
  in Torque to expose simplified self-service to end users.
  Always trigger for: "catalog", "blueprint", "Torque", "Quali", "VMaaS", "BMaaS",
  "OVA deployment", "Intersight automation", "self-service portal", "stepper wizard",
  "service template", "Cisco appliance deployment", "on-prem virtual application",
  "figma", "blueprint designer", "automation canvas", "hypervisor requirements",
  "virtual application requirements", "end-user workflow", "new solution".
---

# Torque Catalog Designer Skill

Generates two tightly coupled artifacts from a single catalog item definition:
1. **React UI** — full dark-mode Catalog Designer with Blueprint Designer, Automation Canvas, and Preview tabs
2. **Terraform modules** — production `.tf` files pre-wired to Torque grain inputs

## Architecture

```
Catalog Item Definition
    ├─► React Catalog Designer (UX)
    │     ├── Blueprint Designer   (designer: offering type → config → steps → assets)
    │     ├── Automation Canvas    (maps form keys → TF/Ansible/Script variable names)
    │     └── Preview              (live end-user stepper experience)
    └─► Terraform Modules (automation)
          ├── ova_vsphere/         → deploys OVA on vSphere
          ├── vmaas_vsphere/       → provisions VM from template on vSphere
          └── bmaas_intersight/    → bare metal via Intersight
```

---

## IMMUTABLE UI RULES

### Tab Names
The three designer tabs must always be named exactly:
- **Blueprint Designer**
- **Automation Canvas**
- **Preview**

### Offering Type Labels
Use exactly these labels:

| ID | Label | Icon | Color |
|---|---|---|---|
| `bmaas` | BMaaS | 🖥️ | #f59e0b |
| `ova` | On-Prem Virtual Application | 📦 | #6366f1 |
| `cloud` | Cloud Application | ☁️ | #0ea5e9 |
| `vmaas` | VMaaS | 💿 | #8b5cf6 |
| `custom` | Custom | ⚙️ | #10b981 |

### Dark Mode
The application is dark mode only.
Background: `#0f172a`, surfaces: `#1e293b`, borders: `#334155`.

### Section Names in Blueprint Designer
- `Hypervisor Requirements`
- `Virtual Application Requirements`
- `End-user Workflow Steps`

## Workflow

### Step 1 — Clarify the offering
Determine or infer:
- Offering type: On-Prem Virtual Application | VMaaS | BMaaS | Cloud Application | Custom
- Hypervisor: VMware (version 7.0 / 8.x) or Nutanix AHV
- Cisco appliances or third-party application
- End-user steps and required fields
- Minimum CPU / memory / storage
- OVA source: curated repo or BYO path

### Step 2 — Generate React UI
Read:
- `references/ui-patterns.md`
- `references/ova-workflow.md`

Required views:
- Dashboard / Home
- `Blueprint Designer` tab
- `Automation Canvas` tab
- `Preview` tab

Single `.jsx` file. Dark mode only. No external CSS files. Use inline styles.

### Step 3 — Generate Terraform Modules
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

### Step 4 — Generate Torque Blueprint YAML
- `spec_version: 2`
- One `grain` per Terraform module
- Inputs use `{{ .inputs.<form_key> }}`
- Add `depends-on` where ordering matters
- Expose meaningful `outputs`

### Step 5 — Present deliverables
Output in this order:
1. React `.jsx`
2. Terraform module files
3. Torque blueprint `.yaml`
4. Wiring table: form key → TF variable → grain input

## Canonical OVA 3-Step End-User Workflow

When offering type is `ova`, default to:

**Step 1 — Basic Information**
- Deployment Name
- Description
- Location

**Step 2 — Deployment Configuration**
- vCenter / Hypervisor
- Cluster
- VM Name
- OVA Source
- Port Group
- Datastore
- CPU
- Memory GB

**Step 3 — Review & Confirm**
- Summary table
- Deploy action

## Variable Naming Convention

Form field keys should be `snake_case` and align with downstream Terraform variables.

### vSphere
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

### Intersight
| Form Key | TF Variable | Notes |
|---|---|---|
| `name` | `server_name` | Server profile name |
| `organization` | `org_name` | Intersight org name or moid |
| `target_platform` | `server_profile_template` | Profile template name |
| `server_assignment_mode` | `assignment_mode` | Static or Pool |

## Torque YAML Structure Reference

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

## Cisco Appliance Defaults

| Appliance | OVA Path | vCPU | RAM GB | Disk GB |
|---|---|---|---|---|
| FTD 7.4 | `curated://cisco/ftd-7.4.ova` | 4 | 8 | 50 |
| FMC 7.4 | `curated://cisco/fmc-7.4.ova` | 8 | 32 | 250 |
| Splunk 9.2 | `curated://splunk/splunk-9.2.ova` | 8 | 16 | 300 |
| ISE 3.3 | `curated://cisco/ise-3.3.ova` | 16 | 64 | 600 |
| CSR 1000v | `curated://cisco/csr-17.3.ova` | 4 | 8 | 16 |
| Nexus 9000v | `curated://cisco/n9kv-10.3.ova` | 4 | 8 | 10 |

## Dark Mode Color Reference

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

## Quality Checklist

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

## Reference Files

- `references/terraform/ova_vsphere.md`
- `references/terraform/vmaas_vsphere.md`
- `references/terraform/bmaas_intersight.md`
- `references/ova-workflow.md`
- `references/ui-patterns.md`
