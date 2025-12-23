# Wazuh SIEM Deployment on Ubuntu Server (Proxmox Lab)

## Purpose

This project documents the intentional deployment of a **Wazuh SIEM platform** using an **Ubuntu Server (Live ISO)** virtual machine hosted on **Proxmox**.  
The goal was to design a **centralized security monitoring system** capable of ingesting logs from multiple lab assets (firewalls, endpoints, and infrastructure services) while maintaining strong network segmentation and operational clarity.

This build prioritizes:
- Realistic enterprise-style deployment patterns
- Clear separation of management, ingestion, and visualization components
- Repeatable, well-documented steps suitable for portfolio review

All sensitive details such as IP addresses, hostnames, and credentials have been sanitized.

---

## Table of Contents
1. Goals
2. Lab Topology
3. Platform & Versioning
4. Proxmox VM Design
5. Ubuntu Server Installation
6. Wazuh All-in-One Deployment
7. Service Validation
8. Accessing the Wazuh Dashboard
9. Post-Deployment Validation
10. Outcome Summary

---

## Goals

- Deploy a fully functional **Wazuh SIEM** using official installation methods
- Centralize log ingestion for future integrations (firewalls, Linux agents, VPN services)
- Maintain a clean and modular lab architecture
- Produce documentation suitable for GitHub portfolio use

---

## Lab Topology

### Simplified View

```
[   OPNsense   ]
       |
[ Proxmox Host ]
       |
[ Ubuntu Server VM ]
       |
[ Wazuh Manager | Indexer | Dashboard ]
```

### Detailed Logical View

```
[ Proxmox Bridge ]
        |
[ Ubuntu Server VM ]
        |
 ├─ wazuh-manager
 ├─ wazuh-indexer
 └─ wazuh-dashboard
```

> All internal IP addressing uses RFC1918 ranges and is intentionally censored.

---

## Platform & Versioning

| Component | Version |
|---------|--------|
| Proxmox VE | Latest Stable |
| Ubuntu Server | 22.04 LTS (Live ISO) |
| Wazuh | 4.x |
| Installation Type | Single-node (All-in-One) |

---

## Proxmox VM Design

**VM Configuration**
- CPU: 4 vCPUs
- Memory: 8 GB RAM
- Storage: 80 GB (VirtIO)
- Network: VirtIO bridge
- Boot Media: Ubuntu Server Live ISO

**Design Rationale**
- Single-node deployment simplifies lab operations
- Sufficient resources allocated to support indexing and dashboard services
- VirtIO drivers chosen for performance

---

## Ubuntu Server Installation

1. Boot VM from Ubuntu Server Live ISO
2. Select *Install Ubuntu Server*
3. Configure:
   - Language & keyboard
   - Network via DHCP (later static)
   - Storage using guided LVM
4. Create administrative user
5. Enable OpenSSH server
6. Complete installation and reboot

After reboot:

```bash
sudo apt update && sudo apt upgrade -y
```

(Optional) Assign a descriptive hostname:

```bash
sudo hostnamectl set-hostname wazuh-server
```

---

## Wazuh All-in-One Deployment

This deployment uses the **official Wazuh installer**, which installs:

- Wazuh Manager
- Wazuh Indexer (OpenSearch)
- Wazuh Dashboard

### Installation Steps

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

**Why this method**
- Officially supported
- Ensures component compatibility
- Ideal for lab and proof-of-concept environments

---

## Retrieving Dashboard Credentials

After installation, credentials are stored locally.

```bash
sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
```

Store these credentials securely.

---

## Service Validation

Verify that all core services are active:

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

Expected state:

```
Active: active (running)
```

---

## Verifying Listening Ports

Wazuh requires the following ports:

| Port | Purpose |
|----|-------|
| 1514/TCP | Agent communication |
| 1515/TCP | Agent enrollment |
| 443/TCP | Dashboard access |

Validate:

```bash
sudo ss -lntp | egrep '1514|1515|443'
```

---

## Accessing the Wazuh Dashboard

Open a browser and navigate to:

```
https://<WAZUH_SERVER_IP>
```

Notes:
- Self-signed certificate warning is expected
- Authenticate using retrieved credentials

---

## Post-Deployment Validation

Inside the Wazuh Dashboard:

- Confirm **Manager status = Running**
- Navigate to **Index Management**
- Verify indices are being created:
  - `wazuh-alerts-*`
  - `wazuh-monitoring-*`
- Confirm no service errors are reported

---

## Outcome Summary

This deployment resulted in:

- A fully operational Wazuh SIEM
- Centralized management, indexing, and visualization
- A scalable foundation for integrating:
  - Firewall logs (OPNsense)
  - Linux agents
  - VPN and infrastructure services

The environment is now prepared for advanced detection engineering, alert tuning, and SOC-style workflows.

---
