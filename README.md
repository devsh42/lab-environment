# Ansible Playbooks for Splunk and Cribl Installation

This repository contains Ansible playbooks for installing and configuring Splunk Enterprise and Cribl Stream on separate VMs/servers.

## Prerequisites

- Ansible 2.9 or higher
- SSH access to target servers (initially as root or user with sudo privileges)
- Sudo/root privileges on target servers for initial bootstrap
- Internet access on target servers for downloading software
- SSH public key for the ansible user (default: `~/.ssh/id_rsa.pub`)

## Project Structure

```
.
├── ansible.cfg              # Ansible configuration
├── inventory.yml            # Inventory file with server definitions
├── inventory.example.yml    # Example inventory file
├── site.yml                 # Main entry point (orchestrates both)
├── group_vars/
│   └── all.yml             # Default variables for all hosts
├── playbooks/
│   ├── bootstrap.yml       # Initial host setup playbook
│   ├── main.yml            # Alternative main playbook
│   ├── splunk.yml          # Splunk installation playbook
│   ├── cribl.yml           # Cribl installation playbook
│   └── templates/
│       ├── splunk.service.j2  # Systemd service template for Splunk
│       └── cribl.service.j2  # Systemd service template for Cribl
└── README.md
```

## Configuration

### 1. Initial Setup - Bootstrap Hosts

**IMPORTANT**: Before running the Splunk/Cribl playbooks, you need to bootstrap your hosts.

#### Step 1: Update Inventory for Bootstrap

Edit `inventory.yml` and set:
- `ansible_host`: IP addresses or hostnames of your servers
- `ansible_user`: Use `root` or another user with sudo privileges for initial connection
- Optionally specify `ansible_ssh_private_key_file` if using SSH keys

#### Step 2: Configure SSH Keys

You have two options:

**Option A: Use default SSH key file** (recommended for single key)
- The playbook will automatically read `~/.ssh/id_rsa.pub` from your control node
- No additional configuration needed

**Option B: Specify multiple SSH keys**
- Edit `group_vars/all.yml` and add your SSH public keys:
  ```yaml
  ansible_ssh_public_keys:
    - "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC... user@host"
    - "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... user@host"
  ```

#### Step 3: Run Bootstrap Playbook

```bash
ansible-playbook playbooks/bootstrap.yml
```

This will:
- Create the `ansible` user on all hosts
- Install sudo package
- Configure SSH authorized keys
- Set up sudo nopasswd access for the ansible user

#### Step 4: Update Inventory After Bootstrap

After successful bootstrap, update `inventory.yml` to use the `ansible` user:
```yaml
ansible_user: ansible  # Changed from 'root'
```

### 2. Configure Variables

#### Splunk Variables (in `playbooks/splunk.yml`)
- `splunk_version`: Splunk version to install (default: "9.1.2")
- `splunk_admin_password`: Admin password (default: "changeme" - **CHANGE THIS**)
- `splunk_build_type`: "linux" for RHEL/CentOS, "linux2" for older versions

#### Cribl Variables (in `playbooks/cribl.yml`)
- `cribl_version`: Cribl version to install (default: "4.3.0")

## Installation

### Complete Installation Workflow

1. **Bootstrap hosts** (first time only):
   ```bash
   ansible-playbook playbooks/bootstrap.yml
   ```

2. **Update inventory** to use `ansible` user (if not already done)

3. **Install services**:
   ```bash
   ansible-playbook site.yml
   ```

### Install Both Services

To install both Splunk and Cribl on their respective servers:

```bash
ansible-playbook site.yml
```

Or using the main playbook:

```bash
ansible-playbook playbooks/main.yml
```

### Install Splunk Only

To install only Splunk:

```bash
ansible-playbook playbooks/splunk.yml
```

### Install Cribl Only

To install only Cribl:

```bash
ansible-playbook playbooks/cribl.yml
```

## What the Playbooks Do

### Bootstrap (`bootstrap.yml`)
1. Creates `ansible` user and group
2. Installs sudo package (if not present)
3. Configures SSH authorized keys for the ansible user
4. Sets up sudo nopasswd access (or password-based if configured)
5. Ensures proper permissions on SSH and sudoers files

### Splunk Installation (`splunk.yml`)
1. Creates `splunk` user and group
2. Downloads Splunk tarball
3. Extracts Splunk to `/opt/splunk`
4. Sets proper ownership and permissions
5. Accepts Splunk license
6. Enables Splunk to start at boot
7. Sets admin password
8. Creates and enables systemd service

### Cribl Installation (`cribl.yml`)
1. Creates `cribl` user and group
2. Downloads Cribl tarball
3. Extracts Cribl to `/opt/cribl`
4. Sets proper ownership and permissions
5. Creates data directory
6. Creates and enables systemd service

## Default Settings

- **Users/Groups**: `splunk/splunk` and `cribl/cribl`
- **Installation Directory**: `/opt/`
- **Splunk Home**: `/opt/splunk`
- **Cribl Home**: `/opt/cribl`
- **Services**: Both services are configured to start automatically on boot

## Accessing the Services

### Splunk
- Web UI: `http://<splunk-server-ip>:8000`
- Default username: `admin`
- Default password: Set via `splunk_admin_password` variable

### Cribl
- Web UI: `http://<cribl-server-ip>:9000`
- Default username: `admin`
- Default password: Set during first login (check Cribl documentation)

## Troubleshooting

### Check Service Status
```bash
# On Splunk server
sudo systemctl status splunk

# On Cribl server
sudo systemctl status cribl
```

### View Logs
```bash
# Splunk logs
sudo journalctl -u splunk -f

# Cribl logs
sudo journalctl -u cribl -f
```

### Manual Service Control
```bash
# Splunk
sudo systemctl start|stop|restart splunk

# Cribl
sudo systemctl start|stop|restart cribl
```

## Quick Reference

### First Time Setup (New Servers)

1. **Prepare SSH access**: Ensure you can SSH to servers as root or a user with sudo privileges
2. **Update inventory**: Edit `inventory.yml` with your server IPs and set `ansible_user: root` (or your sudo user)
3. **Run bootstrap**:
   ```bash
   ansible-playbook playbooks/bootstrap.yml -k  # -k prompts for SSH password if needed
   ```
4. **Update inventory**: Change `ansible_user: root` to `ansible_user: ansible` in `inventory.yml`
5. **Install services**:
   ```bash
   ansible-playbook site.yml
   ```

### Using Password Authentication for Bootstrap

If you don't have SSH keys set up yet, you can use password authentication:

```bash
ansible-playbook playbooks/bootstrap.yml -k -K
# -k prompts for SSH password
# -K prompts for sudo password
```

After bootstrap, SSH keys will be configured and you won't need passwords anymore.

## Notes

- The playbooks are idempotent - you can run them multiple times safely
- If software is already installed, the playbooks will skip installation steps
- Make sure to update the admin passwords in the playbooks before running
- Ensure your servers have sufficient disk space in `/opt/`
- Firewall rules may need to be configured separately for web access
- The bootstrap playbook can be run multiple times safely - it will skip steps if already configured

## Security Considerations

- Change default passwords immediately after installation
- Configure firewall rules to restrict access
- Use SSH keys instead of passwords for Ansible connections
- Review and harden system configurations as needed

