---
type: reference
title: "Terraform"
created: 2026-06-28
updated: 2026-06-28
tags:
  - terraform
  - opentofu
  - iac
  - automation
  - devops
status: developing
related:
  - "[[Ansible]]"
  - "[[Proxmox]]"
  - "[[Cloudflare]]"
  - "[[Docker]]"
---

> [!key-insight] Terraform is **declarative provisioning**: you describe the
> desired infrastructure (VMs, networks, DNS, cloud resources) in HCL and it
> computes the diff to reach that state. It *creates and destroys* resources; for
> configuring the OS inside them, hand off to [[Ansible]] (Terraform builds,
> Ansible configures).

> [!note] **OpenTofu** is the open-source (MPL) fork of Terraform after the
> BSL license change. The CLI is drop-in (`tofu` ↔ `terraform`); the HCL,
> providers, and workflow below are identical.

## Install

```bash
# Arch: pacman -S terraform   (or opentofu)
terraform -version
terraform -install-autocomplete
```

## The core loop

```bash
terraform init       # download providers, set up backend (run first / after provider changes)
terraform fmt        # canonical formatting
terraform validate   # static config check
terraform plan       # show what will change (the diff) — no changes made
terraform apply      # apply the plan (prompts; -auto-approve to skip)
terraform destroy    # tear it all down
```

> [!key-insight] **`plan` is the safety rail.** It diffs desired config against
> recorded state and shows create/update/**destroy** before anything happens.
> Always read the plan — especially the `-/+` (replace) and `-` (destroy) lines —
> before `apply`.

## Configuration (HCL)

```hcl
# providers.tf
terraform {
  required_version = ">= 1.6"
  required_providers {
    proxmox = {
      source  = "bpg/proxmox"
      version = "~> 0.60"
    }
  }
}

provider "proxmox" {
  endpoint = var.pve_endpoint
  # auth via env: PROXMOX_VE_API_TOKEN, etc. (keep secrets out of HCL)
}
```

```hcl
# main.tf
variable "vm_count" {
  type    = number
  default = 2
}

resource "proxmox_virtual_environment_vm" "web" {
  count     = var.vm_count
  name      = "web-${count.index}"
  node_name = "pve1"
  # ...disk, network, cloud-init blocks...
}

output "vm_ids" {
  value = proxmox_virtual_environment_vm.web[*].vm_id
}
```

- **Resources** — things Terraform manages (`resource "type" "name" {}`).
- **Data sources** — read-only lookups (`data "type" "name" {}`).
- **Variables** — inputs (`variable {}`), set via `terraform.tfvars`, `-var`, or
  `TF_VAR_*` env. **Outputs** — exported values (`output {}`).
- **Meta-args** — `count`, `for_each`, `depends_on`, `lifecycle`.
- **Interpolation/refs** — `resource_type.name.attr`, functions, `${}`.

## State — the critical concept

Terraform records what it manages in **`terraform.tfstate`** (a JSON map of
config → real resource IDs). The next `plan` diffs config against this state.

> [!warning] State is sensitive and authoritative. It can contain secrets, and
> losing/corrupting it orphans your real resources. **Never commit local state to
> git**; use a **remote backend** with locking for any shared/team use.

```hcl
# backend.tf — remote state with locking
terraform {
  backend "s3" {
    bucket         = "tf-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "tf-locks"     # state locking
    encrypt        = true
  }
}
```

Other backends: `pg` (Postgres), HTTP, Terraform/Scalr/Spacelift Cloud. State
commands:

```bash
terraform state list                 # resources under management
terraform state show <addr>          # inspect one
terraform import <addr> <real-id>    # adopt an existing resource
terraform state rm <addr>            # stop managing (doesn't destroy)
terraform refresh                    # reconcile state with reality
```

## Modules — reuse

A module is a directory of `.tf` files; call it with inputs:

```hcl
module "vm" {
  source   = "./modules/proxmox-vm"   # or a registry/git source
  for_each = var.vms
  name     = each.key
  cores    = each.value.cores
}
```

Sources can be local paths, the public Terraform Registry
(`source = "terraform-aws-modules/vpc/aws"`), or git URLs. Pin module/provider
**versions** for reproducibility.

## Workspaces & environments

```bash
terraform workspace new staging
terraform workspace select prod
terraform workspace list
```

Workspaces give separate state under one config (`terraform.workspace` in HCL).
For strong prod/staging isolation, many teams prefer **separate state files /
directories per environment** over workspaces.

## Where it fits here

- **[[Proxmox]]** — the `bpg/proxmox` (or `telmate/proxmox`) provider creates
  VMs/LXC from templates; combine with **cloud-init** (covered in [[Proxmox]]) so
  a VM boots already networked with an SSH key, then [[Ansible]] configures it.
- **[[Cloudflare]]** — the `cloudflare` provider manages DNS records, tunnels,
  and zone settings as code.
- **[[Docker]]** — the `kreuzwerker/docker` provider can manage containers/images,
  though Compose is usually simpler for single-host.

> [!key-insight] **Terraform vs Ansible**: Terraform owns *what exists*
> (declarative, state-tracked, builds/destroys infra); [[Ansible]] owns *how a
> box is configured* (procedural-ish, idempotent, no destroy). They compose:
> `terraform apply` → inventory → `ansible-playbook`.

## Related

- [[Ansible]] — configures the OS Terraform provisions
- [[Proxmox]] — primary provisioning target (provider + cloud-init)
- [[Cloudflare]] — DNS/zone management as code
