# Envoy Proxy on RedHat/CentOS/Amazon Linux

## Supported Versions

- **Amazon Linux**: AL2, AL2023
- **RedHat Enterprise Linux (RHEL)**: 7, 8, 9
- **CentOS**: 7, 8, 9

## Installation Method

This role uses **func-e** (Tetrate's Envoy Version Manager) to install Envoy.

### Why not RPMs?

Official Envoy RPMs are not consistently maintained, and third-party repositories (like Tetrate GetEnvoy RPMs) have been found to be unreliable or deprecated. Using `func-e` provides a robust, distribution-agnostic way to fetch the official Envoy binary.

### Key Logic

The role (`tasks/install_redhat.yml`) performs the following:

1.  **Dependencies**: Installs `tar` and `gzip`.
2.  **Manager Setup**: Downloads the `func-e` binary manager to `/usr/local/bin`.
3.  **Fetch Binary**: Runs `func-e use <version>` to download the official Envoy static binary.
4.  **Install**: Copies the fetched binary to `/usr/bin/envoy`.

## Manual Steps (Reference)

If you were to run this manually:

```bash
# 1. Install prerequisites
sudo yum install -y tar gzip

# 2. Install func-e
wget https://github.com/tetratelabs/func-e/releases/download/v1.3.0/func-e_1.3.0_linux_amd64.tar.gz
tar -xzf func-e_1.3.0_linux_amd64.tar.gz
sudo mv func-e /usr/local/bin/
sudo chmod +x /usr/local/bin/func-e

# 3. Fetch Envoy
sudo func-e use 1.30.1

# 4. Install to path
sudo cp ~/.func-e/versions/1.30.1/envoy /usr/bin/envoy
```
