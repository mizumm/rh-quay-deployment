# Red Hat Quay PoC Ansible Playbook

This Ansible Playbook constructs a Red Hat Quay 3.16 Proof of Concept (PoC) environment, based on [Proof of Concept - Deploying Red Hat Quay](https://docs.redhat.com/en/documentation/red_hat_quay/3.16/html-single/proof_of_concept_-_deploying_red_hat_quay/index).

**Note**: This Playbook is intended for building a Proof of Concept environment for Red Hat Quay. It is not recommended to use this Playbook for deploying to a production environment.

## Overview

The following components will be deployed:
- **Podman**: Container runtime
- **PostgreSQL 15**: Database for Quay (includes pg_trgm extension)
- **Redis 6**: For build logs and user events
- **Red Hat Quay 3.16**: Container registry

## Prerequisites

- **Ansible**: Ansible must be installed on the control node
- **RHEL 9 or RHEL 10**: The target node (or localhost) must be RHEL 9 or RHEL 10
- **Red Hat Subscription**: The target node must be registered with Red Hat
- **Registry Credentials**: Red Hat account information with access to `registry.redhat.io`

## Setup

### 1. Inventory Configuration

Edit the `inventory` file to specify the deployment target host. Default is set to `localhost`.

```ini
[quay_servers]
localhost ansible_connection=local
```

### 2. Variable Configuration

Edit `group_vars/all.yml` to set necessary variables. In particular, the following registry credentials are mandatory.

**Recommended**: Set as environment variables

```bash
export REDHAT_REGISTRY_USERNAME="your-rh-username"
export REDHAT_REGISTRY_PASSWORD="your-rh-password"
```

Alternatively, you can edit `group_vars/all.yml` directly (not recommended).

```yaml
registry_username: "your_username"
registry_password: "your_password"
```

Additionally, please review and update the following variables in `group_vars/all.yml` as needed:

- `quay_base_dir`
- `server_hostname`
- `server_ip`
- `quay_admin_password`
- `quay_admin_email`
- `secret_key` / `database_secret_key`

## Usage

Install the necessary Ansible collections:

```bash
ansible-galaxy collection install containers.podman
ansible-galaxy collection install ansible.posix
```

Run the Playbook:

```bash
ansible-playbook -i inventory playbook.yml
```

*Note: Since the playbook uses `become: true`, you generally do not need to run `ansible-playbook` with `sudo`. However, if your user requires a password for sudo privilege escalation, please use the `-K` (or `--ask-become-pass`) option.*

## Verification

After deployment is complete, you can verify the operation with the following steps.

1. **Check Containers**:
   ```bash
   sudo podman ps
   ```
   Ensure that `quay`, `redis`, and `postgresql-quay` containers are running.

2. **Browser Access**:
   Access `https://<server_hostname>` and confirm that the Quay login screen is displayed.
   *Note: You may see a warning about a self-signed certificate (or invalid certificate), but please ignore it as this is a PoC.*

3. **Login**:
   The administrator user configured in `group_vars/all.yml` (default: `quayadmin`) is added to the `SUPER_USERS` array. However, since the database is empty on first launch, you may need to first create an account with the same name via "Create Account" or you may see an initialization wizard depending on the configuration.

## Directory Structure

- `inventory`: Inventory file
- `group_vars/`: Variable definitions
- `roles/`:
  - `prerequisites`: Podman installation, Firewall settings, etc.
  - `database`: PostgreSQL setup
  - `redis`: Redis setup
  - `quay`: Quay configuration generation and deployment
