---
type: reference
title: "Ansible"
created: 2026-06-28
updated: 2026-06-28
tags:
  - ansible
  - automation
  - configuration-management
  - iac
  - devops
status: developing
related:
  - "[[Terraform]]"
  - "[[Proxmox]]"
  - "[[Linux Server Hardening]]"
  - "[[Python]]"
  - "[[Tailscale]]"
---

> [!key-insight] Ansible is **agentless push-based configuration management**: it
> SSHes into hosts and runs idempotent tasks. Use it to configure and maintain the
> *inside* of machines (packages, files, services, users). For provisioning the
> machines themselves, pair it with [[Terraform]] (Terraform builds, Ansible
> configures).

Ansible describes desired state in YAML **playbooks** and applies them over SSH
(WinRM for Windows). No agent runs on the target — only Python + SSH are needed.

## Install & layout

```bash
pipx install ansible            # or: pip install ansible (in a venv → [[Python]])
ansible --version
```

A typical project:

```
inventory/
  hosts.ini            # or hosts.yml — the machines
  group_vars/
    all.yml            # vars for every host
    web.yml            # vars for the 'web' group
  host_vars/
    db01.yml           # vars for one host
roles/
  common/              # reusable role (see structure below)
playbooks/
  site.yml             # top-level playbook
ansible.cfg            # project config
```

## Inventory

```ini
# inventory/hosts.ini
[web]
web01 ansible_host=web01.example
web02 ansible_host=web02.example

[db]
db01 ansible_host=db01.example

[all:vars]
ansible_user=deploy
ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

> [!note] Over [[Tailscale]], target the tailnet name/IP in `ansible_host` and you
> get SSH to every node without exposing ports — the same posture as
> [[Linux Server Hardening]].

## Ad-hoc commands

For one-off actions without writing a playbook:

```bash
ansible all -i inventory/hosts.ini -m ping
ansible web -m apt -a "name=nginx state=present" --become
ansible all -a "uptime"            # default module is 'command'
```

## Playbooks

```yaml
# playbooks/site.yml
- name: Configure web servers
  hosts: web
  become: true                     # sudo
  vars:
    nginx_port: 80
  tasks:
    - name: Install nginx
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: true

    - name: Deploy config
      ansible.builtin.template:
        src: nginx.conf.j2          # Jinja2 template
        dest: /etc/nginx/nginx.conf
        mode: "0644"
      notify: Restart nginx         # triggers a handler

  handlers:
    - name: Restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
```

```bash
ansible-playbook -i inventory/hosts.ini playbooks/site.yml
ansible-playbook ... --check --diff   # dry-run, show changes
ansible-playbook ... --limit web01    # one host
ansible-playbook ... --tags nginx     # only tagged tasks
```

> [!key-insight] **Idempotency is the whole point.** Modules converge to a state,
> so re-running a playbook changes nothing if the host already matches. Prefer
> modules (`apt`, `copy`, `template`, `service`, `lineinfile`) over `command`/
> `shell`, which aren't idempotent unless you guard them with `creates:`/`when:`.

## Roles

Reusable, shareable units. `ansible-galaxy init roles/common` scaffolds:

```
roles/common/
  tasks/main.yml       # the task list
  handlers/main.yml    # handlers
  templates/           # Jinja2 templates
  files/               # static files to copy
  defaults/main.yml    # default vars (lowest precedence)
  vars/main.yml        # role vars (higher precedence)
  meta/main.yml        # role dependencies
```

Consume roles in a play:

```yaml
- hosts: all
  become: true
  roles:
    - common
    - { role: nginx, nginx_port: 8080 }
```

Pull community roles/collections from Ansible Galaxy:

```bash
ansible-galaxy collection install community.general
ansible-galaxy role install geerlingguy.docker
```

## Variables, facts, templates

- **Precedence (low→high)**: role defaults → inventory group_vars → host_vars →
  play vars → `-e` extra-vars (wins).
- **Facts**: `ansible_facts` (gathered automatically) expose host details
  (`ansible_distribution`, `ansible_default_ipv4.address`).
- **Jinja2** in templates/strings: `{{ var }}`, filters (`| default()`,
  `| to_json`), conditionals (`when:`), loops (`loop:`).

## Ansible Vault — secrets

Encrypt sensitive vars/files at rest:

```bash
ansible-vault create secrets.yml          # new encrypted file
ansible-vault edit secrets.yml
ansible-vault encrypt_string 'p@ss' --name 'db_password'   # inline var
ansible-playbook site.yml --ask-vault-pass    # or --vault-password-file
```

> [!warning] Vault protects data **at rest**, not from a playbook that prints it.
> Keep the vault password out of git; store it in a password manager or a CI
> secret. Never commit unencrypted secrets.

## ansible.cfg

```ini
[defaults]
inventory = inventory/hosts.ini
roles_path = roles
host_key_checking = False        # convenient in labs; keep True in prod
retry_files_enabled = False
forks = 20                        # parallelism

[privilege_escalation]
become = True
become_method = sudo
```

## Where it fits

- **Provision vs configure**: [[Terraform]] creates VMs/cloud resources; Ansible
  configures the OS inside them. A common flow is `terraform apply` →
  dynamic-inventory → `ansible-playbook`.
- On [[Proxmox]], `community.general.proxmox*` modules can manage VMs/LXC, but
  most provisioning there is cloud-init + Terraform; Ansible handles post-boot
  config.
- Pairs naturally with [[Linux Server Hardening]] — encode the hardening baseline
  as a role and apply it fleet-wide.

## Related

- [[Terraform]] — provisioning the infrastructure Ansible then configures
- [[Proxmox]] — VMs/LXC Ansible targets post-boot
- [[Python]] — Ansible is Python; install via pipx/venv
