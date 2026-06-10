[![Apache License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
![GitHub release (latest by date)](https://img.shields.io/github/v/release/OT-OSM/node_exporter)
[![Ansible](https://img.shields.io/badge/Ansible-Role-red.svg)](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html)
[![Platform](https://img.shields.io/badge/Platform-Ubuntu%2020.04%20%7C%2022.04-orange.svg)](https://ubuntu.com)

[![Opstree Solutions][opstree_avatar]][opstree_homepage]<br/>[Opstree Solutions][opstree_homepage]

[opstree_homepage]: https://opstree.github.io/
[opstree_avatar]: https://img.cloudposse.com/150x150/https://github.com/opstree.png

---

# Node Exporter — Ansible Role

A production-grade Ansible role to install, configure, and manage **Node Exporter** on Ubuntu systems. Node Exporter exposes a wide variety of hardware and kernel-related metrics via a Prometheus-compatible HTTP endpoint.

## Key Features

- [x] Installs Node Exporter from official Prometheus binaries
- [x] Creates a dedicated system user and group for security isolation
- [x] Configures a systemd service unit via Jinja2 templates
- [x] Manages service lifecycle using Ansible handlers
- [x] Idempotent — safe to re-run without side effects
- [x] All variables are role-namespaced to avoid conflicts

---

## Requirements

| Requirement | Details |
|-------------|---------|
| **OS** | Ubuntu `focal` (20.04) or `jammy` (22.04) |
| **Privileges** | Root or sudo access on target hosts |
| **Ansible Collection** | `community.general` |

Install the required collection:

```bash
ansible-galaxy collection install community.general
```

> **Security Note:** Sensitive defaults are placeholders in `defaults/main.yml`.  
> Always override secrets via **Semaphore environment variables** or **Ansible Vault** — never commit credentials to source control.

---

## Role Structure

```
node_exporter/
├── defaults/
│   └── main.yml          # Default variable values (lowest precedence)
├── vars/
│   └── main.yml          # Internal role variables (higher precedence)
├── tasks/
│   └── main.yml          # Main task entry point
├── handlers/
│   └── main.yml          # Service restart / reload handlers
└── templates/
    └── node_exporter.service.j2   # Jinja2 systemd unit template
```

---

## File Descriptions

### `tasks/main.yml`

Orchestrates all installation and configuration steps:

1. Create dedicated `node_exporter` system user and group
2. Download Node Exporter binary from the Prometheus GitHub releases
3. Extract and install the binary to the system PATH
4. Render and install the systemd service unit from template
5. Enable and start the `node_exporter` service

### `handlers/main.yml`

Triggered automatically when configuration changes are detected:

- **`restart node_exporter`** — restarts the Node Exporter service
- **`reload systemd`** — reloads the systemd daemon after service file updates

### `templates/node_exporter.service.j2`

Jinja2 template that renders the systemd unit file installed to `/etc/systemd/system/node_exporter.service`. Supports variable substitution for listen address, port, and extra flags.

---

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `node_exporter_version` | `1.7.0` | Version of Node Exporter to install |
| `node_exporter_port` | `9100` | Port to expose metrics on |
| `node_exporter_user` | `node_exporter` | System user that runs the service |
| `node_exporter_group` | `node_exporter` | System group for the service user |
| `node_exporter_install_dir` | `/usr/local/bin` | Directory for the binary |

> Override any variable in your playbook, inventory, or via `--extra-vars`.

---

## Usage

### Quick Start

```bash
ansible-playbook -i inventory playbook.yml
```

### Run Specific Phases

```bash
# Install binaries only
ansible-playbook -i inventory playbook.yml --tags install

# Configure service only
ansible-playbook -i inventory playbook.yml --tags configure

# Restart service only
ansible-playbook -i inventory playbook.yml --tags service
```

### Example Playbook

```yaml
---
- name: Deploy Node Exporter
  hosts: monitoring_targets
  become: true
  roles:
    - role: node_exporter
      vars:
        node_exporter_version: "1.7.0"
        node_exporter_port: 9100
```

---

## Tags

| Tag | Description |
|-----|-------------|
| `install` | Download and install the Node Exporter binary |
| `configure` | Render and deploy the systemd service unit |
| `service` | Start, stop, or restart the service |

---

## Handlers

| Handler | Trigger Condition | Action |
|---------|------------------|--------|
| `restart node_exporter` | Config file or binary change | Restarts the Node Exporter service |
| `reload systemd` | Service unit file updated | Reloads the systemd daemon |

---

## Templates

| Template | Destination | Description |
|----------|-------------|-------------|
| `node_exporter.service.j2` | `/etc/systemd/system/node_exporter.service` | Systemd service definition |

---

## Verification

After running the playbook, confirm Node Exporter is running correctly.

### Check service status

```bash
systemctl status node_exporter
```

### Verify metrics endpoint

```bash
curl -s http://localhost:9100/metrics | head -20
```

Expected metrics include:

```
node_cpu_seconds_total
node_memory_MemAvailable_bytes
node_filesystem_size_bytes
node_network_receive_bytes_total
```

### Confirm listening port

```bash
ss -tulpn | grep 9100
```

Expected output:

```
LISTEN   0   4096   *:9100   *:*   users:(("node_exporter",pid=XXXX,fd=3))
```

---

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Service fails to start | Port 9100 already in use | Change `node_exporter_port` or free the port |
| Binary not found | Wrong install dir | Verify `node_exporter_install_dir` is in `$PATH` |
| Metrics endpoint unreachable | Firewall blocking port | Allow TCP 9100 in your firewall rules |
| `systemctl` not found | Non-systemd system | This role requires systemd — not supported on older init systems |

---

## References

| Resource | Link |
|----------|------|
| Node Exporter GitHub | https://github.com/prometheus/node_exporter |
| Prometheus Node Exporter Guide | https://prometheus.io/docs/guides/node-exporter/ |
| Ansible Roles Documentation | https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html |
| Ansible Handlers Documentation | https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_handlers.html |

---

## Authors

| Name | Email | Organization |
|------|-------|-------------|
| Abhishek Vishwakarma | abhishek.vishwakarma@opstree.com | Opstree Solutions |
| Shubham Rathi | shubham.rathi@mygurukulam.co | MyGurukulam |

---

## License

This project is licensed under the [Apache 2.0 License](LICENSE).
