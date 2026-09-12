# AGENTS.md

Ansible project for Linux server management. Two playable surfaces: `inventory/` and `playbooks/`.

## Environment gotchas

- **Ansible is NOT installed** in the dev environment (no `ansible` binary, no python module). You cannot run `ansible-playbook` / `ansible-inventory` to verify work. Instead validate YAML with: `python3 -c "import yaml; yaml.safe_load(open('<file>'))"` (pyyaml is installed).
- `ansible.cfg` sets `become = True` globally (sudo→root on every play). Playbooks that should NOT escalate must set `become: false` explicitly.
- Inventory host/users are **placeholders** (`192.168.x.x`, user `waiyan`, StrictHostKeyChecking=no set globally). Servers are unreachable — never rely on live connection tasks for verification.

## Structure & conventions

- Inventory: `inventory/hosts.yml` (YAML format, not `inventory/hosts`). Groups available: `webservers`, `dbservers`, `monitoring` (children of `production`), plus `staging`. `all` group sets `ansible_connection: ssh`.
- Playbooks live in `playbooks/` and target inventory groups (e.g. `nginx-upgrade.yml` → hosts `webservers`).
- Existing code uses fully-qualified module names (`ansible.builtin.*`), `register` + `failed_when`/`changed_when` on ad-hoc commands, and version info captured via `nginx -v` parsed with `regex_search('nginx/[\\d.]+')`. Follow this style for new playbooks.
- Real commands to hand the user (do not attempt to run here): `ansible-playbook -i inventory/hosts.yml playbooks/<name>.yml`, `ansible-inventory -i inventory/hosts.yml --list`, `ansible all -m ping`.