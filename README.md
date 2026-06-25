# AWS Ansible Automation

This repository provisions and configures three EC2 instances in AWS Mumbai (`ap-south-1`): one control node and two managed nodes grouped as `dev` and `test`.

## Table of Contents

- [Overview](#overview)
  - [Infrastructure](#infrastructure)
  - [Ansible configuration](#ansible-configuration)
  - [Package management](#package-management)
  - [Role and template](#role-and-template)
  - [Login banner](#login-banner)
  - [Custom web server](#custom-web-server)
  - [Git integration](#git-integration)
- [Infrastructure Recreation Using Git](#infrastructure-recreation-using-git)
- [Repository Contents](#repository-contents)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Configuration](#configuration)
  - [Inventory](#inventory)
  - [Ansible config](#ansible-config)
- [Running Playbooks](#running-playbooks)
  - [Execution environment](#execution-environment)
  - [Commands](#commands)
  - [Verification examples](#verification-examples)
- [Playbook Reference](#playbook-reference)
- [Notes](#notes)

## Overview

### Infrastructure

Three EC2 instances in **AWS Mumbai (`ap-south-1`)**:

| Instance type | Name | Ansible group |
|---------------|------|---------------|
| Control node | `mumbai-control` | — |
| Client node 1 | `mumbai-client1` | `dev` |
| Client node 2 | `mumbai-client2` | `test` |

**Setup requirements:**

- `devops` user on all machines with SSH key-based authentication
- Project directory on control node: `/home/devops/ansible`
- All playbooks executed via **Docker** and **ansible-navigator** only

![Mumbai EC2 instances running in ap-south-1](docs/screenshots/mumbai-ec2-instances.jpg)

*Figure 1 — Three EC2 instances in AWS Mumbai (`mumbai-control`, `mumbai-client1`, `mumbai-client2`).*

```mermaid
flowchart LR
  subgraph control [Control Node — mumbai-control]
    A[Docker + ansible-navigator]
  end
  subgraph managed [Managed Nodes]
    B[mumbai-client1<br/>group: dev]
    C[mumbai-client2<br/>group: test]
  end
  A -->|SSH / Ansible EE| B
  A -->|SSH / Ansible EE| C
```

### Ansible configuration

| Deliverable | Path on control node |
|-------------|----------------------|
| Inventory | `/home/devops/ansible/inventory` |
| Ansible config | `/home/devops/ansible/ansible.cfg` |

Inventory rules:

- `mumbai-client1` → group `dev`
- `mumbai-client2` → group `test`

**Verification:** demonstrate connectivity with Ansible (via ansible-navigator).

![Ansible ping — both Mumbai clients reachable](docs/screenshots/mumbai-ansible-ping.jpg)

*Figure 2 — `ansible all -m ping` returns `pong` for `mumbai-client1` and `mumbai-client2`.*

### Package management

**File:** `packages.yml`

| Target | Packages / actions |
|--------|-------------------|
| Groups `dev` and `test` | `mariadb105`, `php` |
| Group `dev` only | `@Development Tools` group |
| Group `dev` only | Update all packages |

**Verification:** show successful playbook execution and installed packages.

![packages.yml executed via ansible-navigator](docs/screenshots/mumbai-packages-run.jpg)

*Figure 3 — MariaDB, PHP, and Development Tools installed successfully.*

### Role and template

| Deliverable | Path |
|-------------|------|
| Role | `/home/devops/ansible/roles/myrole` |
| Template | `roles/myrole/templates/index.j2` |
| Playbook | `/home/devops/ansible/myrole.yml` |

**Role requirements:**

- Configure Apache web server on group `test`
- Template renders: `Welcome to <full hostname> on <IP address>`

```jinja2
Welcome to {{ ansible_fqdn | default(ansible_hostname) }} on {{ ansible_default_ipv4.address }}
```

**Verification:** access the web page via browser or `curl`.

![Test server web page — hostname and IP from template](docs/screenshots/mumbai-test-webpage.jpg)

*Figure 4 — `myrole` output on `mumbai-client2`: Welcome to hostname on IP.*

### Login banner

**File:** `issue.yml`

Replace `/etc/issue` content:

| Group | Expected content |
|-------|------------------|
| `dev` | `development` |
| `test` | `test` |

**Verification:** display `/etc/issue` on each host.

![issue.yml execution and /etc/issue verification](docs/screenshots/mumbai-issue-verify.jpg)

*Figure 5 — `/etc/issue` shows `development` on dev and `test` on test.*

### Custom web server 

**File:** `custom.yml`

Configure Apache on group `dev`:

| Setting | Value |
|---------|-------|
| Document root | `/webdev` |
| Web page content | `development` |
| Symbolic link | `/webdev` → `/var/www/html` |

**Verification:** open the web page and confirm content.

![Dev server web page — development](docs/screenshots/mumbai-dev-webpage.jpg)

*Figure 6 — `custom.yml` serves `development` from the dev host.*

### Git integration

Push to this repository:

- `inventory`
- `ansible.cfg`
- `packages.yml`
- `myrole.yml`
- `issue.yml`
- `custom.yml`
- `roles/myrole`

![GitHub repository with all project files](docs/screenshots/github-repository.jpg)

*Figure 7 — Project pushed to [GitHub](https://github.com/adithyansk-2002/AWS-Ansible-Automation).*

---

## Infrastructure Recreation Using Git

We now recreate the infrastructure in a **new AWS Hyderabad region** environment:

| Instance | Name |
|----------|------|
| Control node | `hyderabad-control` |
| Client 1 | `hyderabad-client1` |
| Client 2 | `hyderabad-client2` |

- User: `clone` (instead of `devops`)
- Project directory: `/home/clone/ansible`
- Clone this Git repo, update inventory for the new IPs, and re-run all playbooks with **ansible-navigator**

**Validation:** successful execution of `packages.yml`, `myrole.yml`, `issue.yml`, and `custom.yml`.

![Hyderabad EC2 instances running in ap-south-2](docs/screenshots/hyderabad-ec2-instances.jpg)

*Figure 8 — Three EC2 instances in AWS Hyderabad.*

![Clone GitHub repo to /home/clone/ansible](docs/screenshots/hyderabad-git-clone.jpg)

*Figure 9 — Mission 1 repository cloned on `hyderabad-control`.*

![Ansible ping — both Hyderabad clients reachable](docs/screenshots/hyderabad-ansible-ping.jpg)

*Figure 10 — Connectivity verified after updating inventory for Hyderabad.*

![Hyderabad test web page](docs/screenshots/hyderabad-test-webpage.jpg)

*Figure 11 — Test host web page (role + template).*

![Hyderabad dev web page](docs/screenshots/hyderabad-dev-webpage.jpg)

*Figure 12 — Dev host web page (`development`).*

---

## Repository Contents

```
.
├── ansible.cfg              # Ansible defaults (inventory path, remote user)
├── ansible-navigator.yml    # Execution environment for ansible-navigator
├── inventory                # dev / test host groups
├── packages.yml             # package installation
├── issue.yml                # /etc/issue banners
├── custom.yml               # Apache on dev
├── myrole.yml               # applies myrole to test group
└── roles/myrole/            # Apache + templated web page
    ├── templates/index.j2
    ├── tasks/main.yml
    └── meta/main.yml
```

## Tech Stack

- **Cloud:** AWS EC2 (Mumbai; Hyderabad)
- **OS:** Amazon Linux 2023 (`dnf`)
- **Automation:** ansible-navigator (execution environment inside Docker)
- **Web server:** Apache (`httpd`)
- **Packages:** MariaDB 10.5 (`mariadb105`), PHP, Development Tools (dev only)

## Prerequisites

1. Three EC2 instances in the target region (Amazon Linux 2023)
2. User created on all nodes (`devops` or `clone`) with SSH key access from the control node
3. **Docker** and **ansible-navigator** on the control node
4. Project deployed under `/home/devops/ansible/` or `/home/clone/ansible/`
5. Python 3 on managed nodes (`/usr/bin/python3`)

## Configuration

### Inventory

Update `inventory` with your instance IPs before running playbooks:

```ini
[dev]
mumbai-client1 ansible_host=<dev-server-ip>

[test]
mumbai-client2 ansible_host=<test-server-ip>

[all:vars]
ansible_user=devops
ansible_python_interpreter=/usr/bin/python3
```

### Ansible config

```ini
[defaults]
inventory = /home/devops/ansible/inventory
host_key_checking = False
retry_files_enabled = False
remote_user = devops
```

## Running Playbooks

All playbooks **must** be executed with ansible-navigator from `/home/devops/ansible/` on the control node.

### Execution environment

`ansible-navigator.yml`:

```yaml
ansible-navigator:
  execution-environment:
    enabled: true
    image: ghcr.io/ansible/creator-ee:v0.22.0
    pull:
      policy: missing
  ansible:
    config:
      path: /home/devops/ansible/ansible.cfg
```

### Commands

Verify connectivity:

```bash
ansible-navigator exec -- ansible all -m ping
```

Run playbooks (recommended order):

```bash
ansible-navigator run packages.yml
ansible-navigator run issue.yml
ansible-navigator run custom.yml
ansible-navigator run myrole.yml
```

### Verification examples

```bash
ansible-navigator exec -- ansible dev -m command -a "cat /etc/issue"
ansible-navigator exec -- ansible dev -m command -a "cat /etc/issue"
ansible-navigator exec -- ansible test -m command -a "cat /etc/issue"

# Test web page (replace with test host IP)
curl http://<test-server-ip>/

# Dev web page (replace with dev host IP)
curl http://<dev-server-ip>/
```

## Playbook Reference

| Playbook | Target | Purpose |
|----------|--------|---------|
| `packages.yml` | `dev`, `test` | Installs `mariadb105` and PHP; updates packages and installs Development Tools on `dev` |
| `issue.yml` | `dev`, `test` | Sets `/etc/issue` to `development` (dev) or `test` (test) |
| `custom.yml` | `dev` | Apache with `/webdev` document root, `development` page content, and `/webdev` symlink |
| `myrole.yml` | `test` | Apache web server via `myrole` using `index.j2` template |

## Notes

- Package management uses `dnf` on Amazon Linux 2023 (MariaDB package may appear as `mariadb105` in the playbook).
- `host_key_checking` is disabled in `ansible.cfg` for lab use; enable it in production.
- Replace IPs in `inventory` before running against live instances.
