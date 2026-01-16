# Known Issues & Resolutions

This document tracks the issues encountered while running the Ansible role on different operating systems and their applied solutions.

## 1. Ubuntu 24.04 (Noble) - Repository Missing

### Symptom

When running on Ubuntu 24.04, the apt task fails with:

```
E: The repository 'https://apt.envoyproxy.io noble Release' does not have a Release file.
```

### Cause

The official Envoy Proxy APT repository does not yet publish packages specifically under the `noble` codename.

### Resolution

- **Fix Applied**: Updated `roles/envoy/tasks/install_debian.yml` to contain logic that detects if the release is `noble` and forces the usage of the `jammy` (Ubuntu 22.04) repository instead.
- **Status**: Fixed.

---

## 2. Amazon Linux / RedHat - RPM Repository Failure

### Symptom

When running on Amazon Linux 2023, the `yum` installation fails with:

```
Failure downloading https://getenvoy.io/.../tetrate-getenvoy.repo
HTTP Error 404: Not Found
```

OR `yum install getenvoy-envoy` fails because the package cannot be found even if the repo file is added (due to redirects/broken paths).

### Cause

The third-party Tetrate GetEnvoy RPM repositories (previously the standard for RHEL/CentOS) appear to be deprecated, moved, or unstable, returning 404s or redirects that `yum` cannot handle gracefully.

### Resolution

- **Fix Applied**: Switched the installation strategy in `roles/envoy/tasks/install_redhat.yml` from using RPMs to using **func-e** (Envoy Version Manager).
  - The role now installs `tar` and `gzip`.
  - Downloads the standard `func-e` binary manager from GitHub.
  - Uses `func-e` to fetch the official Envoy static binary (v1.30.1) and installs it to `/usr/bin/envoy`.
- **Status**: Fixed.

---

## 3. SSH Connection on Amazon Linux

### Symptom

```
ubuntu@34.202.205.151: Permission denied (publickey)
```

### Cause

Amazon Linux instances use `ec2-user` by default, whereas Ubuntu instances use `ubuntu`.

### Resolution

- **Fix Applied**: Updated `inventory.ini` to use `ansible_user=ec2-user` for the Amazon Linux host.
- **Status**: Configured.
