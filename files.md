# Envoy Proxy Project Files

This document provides a detailed explanation of each file in the Envoy Proxy Ansible project, including the working code.

## 1. Inventory File (`inventory.ini`)

**Path**: `inventory.ini`
**Purpose**: Defines the target hosts for Ansible. It lists the EC2 instance IP and the connection details (user, private key, SSH options).

```ini
[envoy_nodes]
3.88.12.69 ansible_user=ubuntu ansible_ssh_private_key_file=/home/deepak/Downloads/Envoy_Proxy.pem ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

## 2. Playbook (`playbook.yml`)

**Path**: `playbook.yml`
**Purpose**: The entry point for Ansible execution. It maps the hosts from the inventory to the roles that should be applied to them.

```yaml
---
- hosts: all
  become: yes
  roles:
    - role: roles/envoy
```

## 3. Role Tasks (`roles/envoy/tasks/main.yml`)

**Path**: `roles/envoy/tasks/main.yml`
**Purpose**: The core logic of the role. It steps through the installation process:

1.  Installs prerequisites (`apt-transport-https`, etc.).
2.  Sets up the Envoy GPG key (critical step: downloads and de-armors it).
3.  Adds the Envoy repository (using `jammy` codename for compatibility).
4.  Installs the `envoy` package.
5.  Creates a dedicated `envoy` user and group.
6.  Deploys the systemd service file.
7.  Deploys the Envoy configuration file.
8.  Starts and enables the service.

```yaml
---
- name: Install prerequisites
  ansible.builtin.apt:
    name:
      - apt-transport-https
      - gnupg2
      - curl
      - lsb-release
    state: present
    update_cache: yes

- name: Remove broken Envoy GPG key
  ansible.builtin.file:
    path: /usr/share/keyrings/envoy-keyring.gpg
    state: absent

- name: Download Envoy GPG key
  ansible.builtin.get_url:
    url: https://apt.envoyproxy.io/signing.key
    dest: /tmp/envoy-signing.key
    mode: "0644"

- name: Dearmor Envoy GPG key
  ansible.builtin.shell: cat /tmp/envoy-signing.key | gpg --dearmor -o /usr/share/keyrings/envoy-keyring.gpg
  args:
    creates: /usr/share/keyrings/envoy-keyring.gpg

- name: Add Envoy repository
  ansible.builtin.apt_repository:
    repo: "deb [arch=amd64 signed-by=/usr/share/keyrings/envoy-keyring.gpg] https://apt.envoyproxy.io jammy main"
    state: present
    filename: envoy

- name: Install Envoy
  ansible.builtin.apt:
    name: envoy
    state: present
    update_cache: yes
  notify: Restart envoy

- name: Create Envoy group
  ansible.builtin.group:
    name: envoy
    state: present

- name: Create Envoy user
  ansible.builtin.user:
    name: envoy
    group: envoy
    system: yes
    shell: /usr/sbin/nologin
    state: present

- name: Deploy Envoy service file
  ansible.builtin.template:
    src: envoy.service.j2
    dest: /etc/systemd/system/envoy.service
    mode: "0644"
  notify: Restart envoy

- name: Reload systemd
  ansible.builtin.systemd:
    daemon_reload: yes

- name: Create Envoy configuration directory
  ansible.builtin.file:
    path: /etc/envoy
    state: directory
    owner: envoy
    group: envoy
    mode: "0755"

- name: Deploy Envoy configuration
  ansible.builtin.template:
    src: envoy.yaml.j2
    dest: "{{ envoy_config_path }}"
    owner: envoy
    group: envoy
    mode: "0644"
  notify: Restart envoy

- name: Ensure Envoy service is started and enabled
  ansible.builtin.service:
    name: envoy
    state: started
    enabled: yes
```

## 4. Role Handlers (`roles/envoy/handlers/main.yml`)

**Path**: `roles/envoy/handlers/main.yml`
**Purpose**: Defines actions that verify triggered by `notify` in tasks. Here, it restarts the Envoy service when the configuration or package changes.

```yaml
---
- name: Restart envoy
  ansible.builtin.service:
    name: envoy
    state: restarted
```

## 5. Role Defaults (`roles/envoy/defaults/main.yml`)

**Path**: `roles/envoy/defaults/main.yml`
**Purpose**: Sets default values for variables. This allows the configuration path to be overridden if needed without changing the code.

```yaml
---
envoy_config_path: /etc/envoy/envoy.yaml
```

## 6. Envoy Configuration Template (`roles/envoy/templates/envoy.yaml.j2`)

**Path**: `roles/envoy/templates/envoy.yaml.j2`
**Purpose**: The actual configuration file for the Envoy Proxy.

- **Listener**: Listens on port `10000`.
- **Route**: Matches any domain (`*`) and any prefix (`/`).
- **Response**: Returns a direct "Hello from Envoy!" 200 OK response.
- **Admin**: Admin interface on port `9901`.
- _Note_: The standard `stdout` access logger was removed to be compatible with systemd logging.

```yaml
admin:
  address:
    socket_address:
      protocol: TCP
      address: 0.0.0.0
      port_value: 9901

static_resources:
  listeners:
    - name: listener_0
      address:
        socket_address:
          protocol: TCP
          address: 0.0.0.0
          port_value: 10000
      filter_chains:
        - filters:
            - name: envoy.filters.network.http_connection_manager
              typed_config:
                "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
                stat_prefix: ingress_http
                route_config:
                  name: local_route
                  virtual_hosts:
                    - name: local_service
                      domains: ["*"]
                      routes:
                        - match:
                            prefix: "/"
                          direct_response:
                            status: 200
                            body:
                              inline_string: "Hello from Envoy!\n"
                http_filters:
                  - name: envoy.filters.http.router
                    typed_config:
                      "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router
```

## 7. Service Template (`roles/envoy/templates/envoy.service.j2`)

**Path**: `roles/envoy/templates/envoy.service.j2`
**Purpose**: A systemd unit file to manage Envoy as a background service. It specifies the user `envoy` and the command to start the binary with the config file.

```ini
[Unit]
Description=Envoy Proxy
Documentation=https://www.envoyproxy.io/
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/bin/envoy -c {{ envoy_config_path }}
Restart=on-failure
User=envoy
Group=envoy

[Install]
WantedBy=multi-user.target
```
