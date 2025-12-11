# Ansible Playbooks for Splunk and Cribl Installation

This repository contains Ansible playbooks for installing and configuring Splunk Enterprise and Cribl Stream on separate VMs/servers.

## Prerequisites

- Ansible 2.9 or higher
- SSH access to target servers
- Sudo/root privileges on target servers
- Internet access on target servers for downloading software

## Project Structure

```
.
├── ansible.cfg              # Ansible configuration
├── inventory.yml            # Inventory file with server definitions
├── site.yml                 # Main entry point (orchestrates both)
├── playbooks/
│   ├── main.yml            # Alternative main playbook
│   ├── splunk.yml          # Splunk installation playbook
│   ├── cribl.yml           # Cribl installation playbook
│   └── templates/
│       ├── splunk.service.j2  # Systemd service template for Splunk
│       └── cribl.service.j2  # Systemd service template for Cribl
└── README.md
```

## Configuration

### 1. Update Inventory

Edit `inventory.yml` and update the following:
- `ansible_host`: IP addresses or hostnames of your servers
- `ansible_user`: SSH username for connecting to servers

### 2. Configure Variables

#### Splunk Variables (in `playbooks/splunk.yml`)
- `splunk_version`: Splunk version to install (default: "9.1.2")
- `splunk_admin_password`: Admin password (default: "changeme" - **CHANGE THIS**)
- `splunk_build_type`: "linux" for RHEL/CentOS, "linux2" for older versions

#### Cribl Variables (in `playbooks/cribl.yml`)
- `cribl_version`: Cribl version to install (default: "4.3.0")

## Installation

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

## Notes

- The playbooks are idempotent - you can run them multiple times safely
- If software is already installed, the playbooks will skip installation steps
- Make sure to update the admin passwords in the playbooks before running
- Ensure your servers have sufficient disk space in `/opt/`
- Firewall rules may need to be configured separately for web access

## Security Considerations

- Change default passwords immediately after installation
- Configure firewall rules to restrict access
- Use SSH keys instead of passwords for Ansible connections
- Review and harden system configurations as needed

