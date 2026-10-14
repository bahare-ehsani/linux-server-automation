# Ansible Server Automation

## Objective

Use Ansible to verify security settings and practice configuration management on an AlmaLinux managed server.

The security verification playbook is read-only. The lab and SSH hardening playbooks are intended for controlled practice and configuration management.

## Architecture

The project uses a separate Ansible Control Node to manage the AlmaLinux server over SSH.

```text
Ansible Control Node
        |
        | SSH
        v
AlmaLinux Managed Node
```

Ansible uses SSH key authentication to connect to the managed server. Privileged tasks use `become`.

## Repository Structure

```text
ansible/
├── inventory/
│   ├── hosts.example
│   └── hosts                 # Local file; excluded from Git
├── lab/
│   └── lab-config.yml
├── playbooks/
│   ├── security-verification.yml
│   └── ssh-hardening.yml
└── README.md
```

## Inventory Configuration

The inventory defines the managed AlmaLinux server.

Example inventory:

```ini
[managed]
almalinux ansible_host=192.0.2.10 ansible_user=devops
```

The address `192.0.2.10` is reserved for documentation. Replace it with the IP address of your own managed server in your local inventory.

From the repository root, create the local inventory:

```bash
cp ansible/inventory/hosts.example ansible/inventory/hosts
```

Edit `ansible/inventory/hosts` and set the correct server address and SSH user.

The local `hosts` file is excluded from Git to avoid publishing environment-specific connection details. Do not store passwords, tokens, or private SSH keys in the inventory.

The `devops` account should have SSH key access and the required sudo privileges on the managed server.

## Playbooks

### 1. Security Verification

File: `playbooks/security-verification.yml`

This read-only playbook verifies:

- SSH root login is disabled.
- SSH password authentication is disabled.
- Keyboard-interactive authentication is disabled.
- Maximum authentication attempts are set to 3.
- SSH login grace time is set to 30 seconds.
- X11 forwarding is disabled.
- Firewalld is active and the public zone allows the expected SSH service.
- SELinux is in `Enforcing` mode.
- Auditd is active.
- Selected IPv4 redirect settings are set to `0`.
- Journald is active and the configured persistence and integrity checks pass.

### 2. SSH Hardening

File: `playbooks/ssh-hardening.yml`

This playbook manages an SSH configuration drop-in file. SSH configuration changes can affect remote access.

Before applying changes to a server, verify the current configuration, ensure console access or another recovery method is available, and review the playbook. Do not run it against a production server without an appropriate backup and rollback plan.

### 3. Lab Configuration

File: `lab/lab-config.yml`

This playbook practices managing a lab configuration file with Ansible. It uses backup support when the target file changes.

## Running the Playbooks

Run commands from the repository root.

Check playbook syntax:

```bash
ansible-playbook -i ansible/inventory/hosts ansible/playbooks/security-verification.yml --syntax-check
```

Run the read-only security verification:

```bash
ansible-playbook -i ansible/inventory/hosts ansible/playbooks/security-verification.yml -K
```

The `-K` option prompts for the sudo password required by Ansible `become`.

Practice the lab configuration only on the intended lab host:

```bash
ansible-playbook -i ansible/inventory/hosts ansible/lab/lab-config.yml -K
```

Do not run the SSH hardening playbook until its impact has been reviewed and a recovery path is available.

## Expected Result

For the read-only security verification, a successful run should normally report:

```text
changed=0
unreachable=0
failed=0
```

The exact number of successful and skipped tasks can vary with the managed server's state.

## Design Notes

The verification playbook separates checking security state from changing configuration.

The project currently uses straightforward playbooks. Ansible roles can be introduced later if the project grows and reusable task groups become necessary.

## Safety Principles

- Check the current state before making changes.
- Review the impact of configuration changes.
- Back up relevant configuration before sensitive changes.
- Keep a recovery method available before changing SSH.
- Test changes in a lab environment before using them in production.
- Keep real inventory files, credentials, and private keys out of Git.
