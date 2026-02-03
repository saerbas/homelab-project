# Phase 1: Base Infrastructure & Security Hardening

## 1. Project Overview
**Goal:** Transform a basic laptop to a "Headless Server" which will be fully controlled and configured by **Infrastructure as Code (IaC)**

**Roles:**
* **Control Node:** Main PC (MacBook Air) – runs ansible
* **Managed Node:** Notebook (Ubuntu Server) – target system

---

## 2. Manual (Bootstrap)
Since a "Bare Metal" device has no API, minimal manual steps were necessary.

* **OS:** Ubuntu Server 24.04 LTS.
* **Network:** Static IP (via DHCP reservation) or known IP in the LAN.
* **SSH:** OpenSSH Server enabled during installation.
* **User:** Initial admin user created (no root).

---

## 3. SSH Trust Relationship (Authentication)
To enable automation, password authentication was replaced with SSH keys.

**Commands on the Control Node:**
```bash
# 1. Generate key pair
ssh-keygen -t ed25519 -C "ansible-control"

# 2. Copy public key to the server
ssh-copy-id user@<SERVER_IP>
```
**Result**
Login via ssh user@<SERVER_IP> is performed without password entry.

## 4. Ansible Configuration
Setting up the automation environment on the Control Node.

**Folder Structure**
* homelab-project/
* └── ansible/
    * ├── ansible.cfg    # Ansible behavior configuration
    * ├── inventory.ini  # List of target servers
    * └── setup.yml      # Playbook for hardening & setup

**Files**
**inventory.ini**
```toml
[masters]
192.168.x.x  ansible_user=<YOUR_USER>

[defaults]
inventory = inventory.ini
host_key_checking = False
```

## 5. System Hardening (Playbook: setup.yml)
The heart of Phase 1. This playbook defines the desired security state.

**Tasks**
1. System Update: apt update & apt dist-upgrade.
2. Tooling: Installation of curl, git, htop, fail2ban, ufw.
3. Firewall (UFW):
* Default Policy: Deny Incoming (block everything)
* Allow: Port 22 (SSH) and Port 6443 (Kubernetes API).
4. SSH Hardening:
* PasswordAuthentication no: Only keys allowed
* PermitRootLogin no: No direct root access

## 6. Deployment & Verification
Command: 
```bash
ansible-playbook setup.yml -K
```
Verification (Tests):

Connectivity: ansible all -m ping -> returns "pong".

Security: Login attempt without key fails.

Ports: Firewall blocks everything except Port 22 (and later 6443).

## 7. Lessons Learned & Key Concepts
Idempotency: The playbook can be executed multiple times, it only changes deviations.

Headless Management: The server is managed exclusively remotely.

Sequence: Open SSH port before activating the firewall.

