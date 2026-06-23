# ITFS Task 1 - Mission 1

## Objective
This repository contains the Ansible-based infrastructure automation work completed for Mission 1.

## Environment
- AWS Region: Mumbai (`ap-south-1`)
- OS used: Amazon Linux 2023
- Control node: `mumbai-control`
- Managed nodes:
  - `mumbai-client1`
  - `mumbai-client2`

## Tools Used
- Ansible
- ansible-navigator
- Docker
- AWS EC2

## Files Included
- `inventory` — inventory file with `dev` and `test` groups
- `ansible.cfg` — Ansible configuration
- `packages.yml` — installs required packages and dev tools
- `issue.yml` — configures `/etc/issue` on dev and test servers
- `custom.yml` — configures the custom web page on the dev server
- `myrole/` — role used to configure the test server web page

## Tasks Completed
1. Created 3 EC2 instances in AWS Mumbai region
2. Configured passwordless SSH access for `devops` user
3. Installed Docker and ansible-navigator on the control node
4. Configured Ansible inventory and connectivity
5. Executed package installation playbook
6. Created and executed role for test server web page
7. Configured `/etc/issue` on dev and test servers
8. Created custom web page on dev server

## Notes
Mission 1 was completed on Amazon Linux 2023 using `dnf`-based package management.
