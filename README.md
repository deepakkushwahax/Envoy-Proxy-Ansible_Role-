# Envoy Proxy Ansible Role

---

## Table of Contents

1. [Overview](#1-overview)
2. [Supported Operating Systems](#2-supported-operating-systems)
3. [Prerequisites & Known Limitations](#3-prerequisites--known-limitations)
4. [Role Structure](#4-role-structure)
5. [Configuration Overview](#5-configuration-overview)
6. [Installation Flow](#6-installation-flow)
7. [Running the Playbook](#7-running-the-playbook)
8. [Validation & Testing](#8-validation--testing)
9. [Best Practices Followed](#9-best-practices-followed)
10. [Troubleshooting](#10-troubleshooting)
11. [Conclusion](#11-conclusion)
12. [References](#12-references)
13. [Author](#13-author)

---

## 1. Overview

Envoy Proxy is an open-source high-performance **Edge & Service Proxy** designed for cloud-native applications.

This Ansible role automates the installation and configuration of Envoy Proxy, providing:

- Automated repository setup with secure GPG key handling
- Systemd service configuration for process management
- Dedicated system user for security
- Basic configuration for immediate usability

Official documentation:
[https://www.envoyproxy.io/docs/envoy/latest/](https://www.envoyproxy.io/docs/envoy/latest/)

---

## 2. Supported Operating Systems

| OS Family | Versions                                                |
| :-------- | :------------------------------------------------------ |
| Debian    | Ubuntu 22.04+ (Jammy), Debian 11/12 (Bullseye/Bookworm) |
| RedHat    | Amazon Linux 2/2023, RedHat EL 7/8/9, CentOS 7/8/9      |

- **[Debian / Ubuntu Instructions](Debian.md)**
- **[RedHat / Amazon Linux Instructions](RedHat.md)**

_Note: Verified specifically on Ubuntu 24.04 LTS (Noble) and Amazon Linux 2023._

---

## 3. Prerequisites & Known Limitations

### System Requirements

| Requirement | Description                           |
| :---------- | :------------------------------------ |
| RAM         | 2 GB+ recommended                     |
| CPU         | 1 vCPU or more                        |
| Disk        | 10 GB+                                |
| Network     | Outbound access for package downloads |

### Ansible Requirements

| Package | Purpose                         |
| :------ | :------------------------------ |
| ansible | Playbook execution (Core 2.12+) |

### Known Limitations

- **Package Source**:
  - Ubuntu/Debian: Official Envoy Proxy APT repository.
  - RedHat/Amazon Linux: Tetrate GetEnvoy RPM repository.
- Default configuration is a basic starter template; complex routing/filtering logic must be applied via `envoy.yaml.j2` or variable overrides.

---

## 4. Role Structure

```
roles/
└── envoy/
    ├── defaults/
    │   └── main.yml           # Default variables
    ├── handlers/
    │   └── main.yml           # Service restart handlers
    ├── tasks/
    │   └── main.yml           # Main installation tasks
    ├── templates/
    │   ├── envoy.service.j2   # Systemd service unit
    │   └── envoy.yaml.j2      # Envoy configuration template
```

---

## 5. Configuration Overview

Key variables are defined in `roles/envoy/defaults/main.yml`.

| Variable            | Default Value           | Description                                              |
| :------------------ | :---------------------- | :------------------------------------------------------- |
| `envoy_config_path` | `/etc/envoy/envoy.yaml` | Path where the main configuration file will be deployed. |

Systemd service settings (customizable via template editing if needed):

- **User/Group**: `envoy` / `envoy`
- **ExecStart**: `/usr/bin/envoy -c /etc/envoy/envoy.yaml`

---

## 6. Installation Flow

High-level execution flow of the role:

1.  **Prerequisites**: Installs basic utilities (`curl`, `gnupg2`, `yum-utils`, etc.).
2.  **Repository Setup**:
    - **Debian/Ubuntu**:
      - Removes broken keys, downloads official Envoy GPG key (dearmored).
      - Adds official `apt.envoyproxy.io` repository dynamically based on release.
    - **RedHat/Amazon Linux**:
      - Installs `func-e` binary manager to fetch Envoy.
3.  **Package Installation**:
    - **Debian/Ubuntu**: Installs `envoy` via APT.
    - **RedHat/Amazon Linux**: Downloads official Envoy binary via `func-e` and installs to `/usr/bin/envoy`.
4.  **User Creation**: Creates a dedicated system user (`envoy`) and group.
5.  **Configuration**:
    - Creates `/etc/envoy`.
    - Deploys `envoy.yaml` from template.
6.  **Service Setup**:
    - Deploys custom `envoy.service` to `/etc/systemd/system/`.
    - Reloads systemd daemon.
7.  **Startup**: Enables and starts the `envoy` service.

---

## 7. Running the Playbook

To deploy Envoy Proxy to your target inventory:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

**Example playbook.yml:**

```yaml
- hosts: all
  become: true
  roles:
    - envoy
```

---

## 8. Validation & Testing

### Service Status

Verify the service is active:

```bash
sudo systemctl status envoy
```

### Connectivity Check

Check if Envoy is listening (default config listens on port 10000):

```bash
curl -v localhost:10000
```

_Expected Output:_ `Hello from Envoy!` (based on default static response config).

### Port listener check

```bash
ss -lntp | grep envoy
```

---

## 9. Best Practices Followed

| Practice                   | Description                                                        |
| :------------------------- | :----------------------------------------------------------------- |
| **GPG Formatting**         | Handles binary de-armoring explicitly for robust `apt` usage.      |
| **Security**               | Runs as a dedicated `envoy` system user, not root.                 |
| **Idempotency**            | Tasks are written to be safe for re-runs (e.g., `creates` checks). |
| **Separation of Concerns** | Config parsing and service management are decoupled.               |
| **Systemd Integration**    | logging configured for `journald` compatibility (stdout/stderr).   |

---

## 10. Troubleshooting

| Issue                      | Fix                                                                                                 |
| :------------------------- | :-------------------------------------------------------------------------------------------------- |
| **APT Error: NO_PUBKEY**   | The role automatically handles GPG keys. Ensure outbound traffic to `apt.envoyproxy.io` is allowed. |
| **Service fails to start** | Check logs: `journalctl -u envoy -n 50`. Common issues include config syntax errors.                |
| **Port Conflict**          | Ensure port 10000 (default) is not in use, or modify `envoy.yaml.j2`.                               |

---

## 11. Conclusion

This role provides a reliable, automated method to deploy Envoy Proxy on Ubuntu infrastructure. It addresses common pain points like GPG key formatting and missing systemd units in the default package, ensuring a production-ready baseline.

---

## 12. References

| Purpose             | Link                                                     |
| :------------------ | :------------------------------------------------------- |
| Envoy Official Site | [https://www.envoyproxy.io/](https://www.envoyproxy.io/) |
| Ansible Docs        | [https://docs.ansible.com/](https://docs.ansible.com/)   |

---

## 13. Author

**Author:** Deepak Kushwaha
**Last Updated:** 2026-01-14
