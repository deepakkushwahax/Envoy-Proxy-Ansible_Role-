# Envoy Proxy on Debian/Ubuntu

## Supported Versions

- **Ubuntu**: 20.04 (Focal), 22.04 (Jammy), 24.04 (Noble)
- **Debian**: 11 (Bullseye), 12 (Bookworm)

## Installation Method

This role uses the official **Envoy Proxy APT Repository**.

### Automatic Logic

The role (`tasks/install_debian.yml`) automatically handles:

1.  **GPG Key Management**: Downloads the official signing key and de-armors it for security.
2.  **Repository Mapping**:
    - Standard releases use their own codename (e.g., `jammy`).
    - **Ubuntu 24.04 (Noble)**: Automatically maps to `jammy` as official Noble repos are not yet published.

## Manual Steps (Reference)

If you were to run this manually, the steps performed are:

```bash
# 1. Install prerequisites
sudo apt-get update
sudo apt-get install -y apt-transport-https gnupg2 curl lsb-release

# 2. Add Key
curl -sL 'https://apt.envoyproxy.io/signing.key' | sudo gpg --dearmor -o /usr/share/keyrings/envoy-keyring.gpg

# 3. Add Repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/envoy-keyring.gpg] https://apt.envoyproxy.io jammy main" | sudo tee /etc/apt/sources.list.d/envoy.list

# 4. Install
sudo apt-get update
sudo apt-get install -y envoy
```
