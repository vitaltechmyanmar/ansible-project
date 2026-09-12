# Ansible Linux Server Management

Ansible project for Linux server management: inventory definition and automation playbooks.

## Requirements

- Ansible installed on the control node: `pip install ansible`
- SSH access to managed hosts (user `waiyan`, sudo for escalated plays)
- Python + pyyaml on the control node for local YAML validation

## Repository layout

```
├── ansible.cfg              # global config: inventory path, SSH, sudo escalation
├── inventory/
│   └── hosts.yml            # static inventory (webservers, dbservers, monitoring, staging)
└── playbooks/
    └── nginx-upgrade.yml    # upgrade nginx, backup config, verify, reload
```

## Quick start

```bash
# validate inventory YAML (Ansible not installed in dev env)
python3 -c "import yaml; yaml.safe_load(open('inventory/hosts.yml'))"

# list inventory
ansible-inventory -i inventory/hosts.yml --list

# connectivity test
ansible all -i inventory/hosts.yml -m ping

# run a playbook
ansible-playbook -i inventory/hosts.yml playbooks/nginx-upgrade.yml
```

## Inventory

Groups in `inventory/hosts.yml`:

| Group        | Hosts        | Notes                                   |
|--------------|--------------|-----------------------------------------|
| `webservers` | web01, web02 | nginx hosts                             |
| `dbservers`  | db01         | database hosts                          |
| `monitoring` | mon01        | monitoring hosts                        |
| `production` | (children)   | aggregates the three groups above       |
| `staging`    | stg-web01    | non-production environment              |

All entries are placeholders — replace `ansible_host` / `ansible_user` with real values before use.

## Playbooks

### `nginx-upgrade.yml`

Upgrades nginx to the latest package on all `webservers`:

1. Records current version via `nginx -v`
2. Backs up `/etc/nginx` to `/var/backups/nginx/nginx-<timestamp>`
3. Updates package index and installs latest `nginx` (apt/dnf/yum auto-detected)
4. Records new version
5. Runs `nginx -t`; reloads service only if the config is valid, otherwise fails and points to the backup

Run: `ansible-playbook -i inventory/hosts.yml playbooks/nginx-upgrade.yml`

> Note: `ansible.cfg` enables `become = True` globally, so playbooks escalate to root unless they opt out with `become: false`.

## Upgrade workflow

```mermaid
flowchart TD
    A([Run ansible-playbook]) --> B[Record old version nginx -v]
    B --> C[Backup /etc/nginx to /var/backups]
    C --> D{Package manager?}
    D -->|apt| E[Update cache + install latest]
    D -->|dnf</br>yum| F[Install latest]
    E --> G[Record new version nginx -v]
    F --> G
    G --> H{nginx -t valid?}
    H -->|Yes| I[Reload nginx service]
    I --> J[Print old → new + backup path]
    H -->|No| K[Fail with backup location]
    J --> L([Done])
    K --> L
```

## Related

- `AGENTS.md` — environment gotchas and conventions for agents working in this repo