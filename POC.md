# Proof of Concept: Envoy Proxy Installation on Ubuntu

## 1. Objective

The objective of this POC is to demonstrate the automated installation and configuration of Envoy Proxy on an Ubuntu 24.04 (Noble) EC2 instance using Ansible.

## 2. Environment

- **Target OS**: Ubuntu 24.04 LTS (Noble Numbat)
- **Infrastructure**: AWS EC2 Instance
- **Tooling**: Ansible (Core 2.16+)

## 3. Implementation Details

### Ansible Role Structure

The solution is implemented as a reusable Ansible role (`roles/envoy`) with the following components:

- **`tasks/main.yml`**: Handles package prerequisites, GPG key setup, repository configuration, package installation, and service deployment.
- **`handlers/main.yml`**: Manages the `envoy` service restart.
- **`templates/`**: Contains the `envoy.yaml.j2` configuration and `envoy.service.j2` systemd unit file.
- **`defaults/main.yml`**: Defines default variables.

### Key Challenges & Solutions

#### A. GPG Key Format (Binary vs. Armored)

**Issue**: The stable Envoy repository uses a GPG key that caused compatibility issues with `apt` when not properly de-armored.
**Solution**: Implemented an explicit step to download the key and convert it to a binary format using `gpg --dearmor` before placing it in `/usr/share/keyrings/`.

#### B. Missing Systemd Service

**Issue**: The default `envoy` package from the official repository does not include a systemd service file, preventing service management via `systemctl`.
**Solution**: Created a custom `envoy.service` systemd unit file and added tasks to create a dedicated `envoy` system user and group.

#### C. Logging Configuration

**Issue**: The default Envoy configuration attempted to log to `/dev/stdout`, which caused the systemd service to fail startup.
**Solution**: Modified the Envoy configuration template (`envoy.yaml.j2`) to remove the explicit `StdoutAccessLog` instruction, allowing systemd to capture standard output natively.

## 4. Verification

The deployment was verified on the target instance `100.27.204.250`.

### Service Status

The `envoy` service is active and running:

```bash
systemctl status envoy
# Status: Active (running)
```

### Connectivity Check

A curl request to the Envoy listener on port 10000 returns the configured response:

```bash
curl -v localhost:10000
# Output: Hello from Envoy!
```

## 5. Conclusion

This POC successfully validates the automated deployment of Envoy Proxy on Ubuntu. The Ansible role is robust, handling repository quirks and service configuration requirements to ensure a reliable installation.
