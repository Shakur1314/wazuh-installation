# wazuh-installation
complete guide for installing wazuh siem on ubuntu
# Wazuh Installation Guide - Simple Steps

A straightforward guide for installing Wazuh SIEM on Ubuntu using the automated all-in-one method.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Accessing the Dashboard](#accessing-the-dashboard)
- [Setting Up Agents](#setting-up-agents)

---

## Prerequisites

### System Requirements
- Ubuntu 20.04 LTS or newer
- Minimum 4GB RAM (8GB recommended)
- Minimum 50GB disk space
- Root or sudo access

### Check System Resources
```bash
free -h
df -h
```

---

## Installation

### Step 1: Download the Installation Script
```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
```

### Step 2: Run Automated All-in-One Installation
```bash
sudo bash wazuh-install.sh -a
```

This single command will:
- Install Wazuh Indexer (data storage)
- Install Wazuh Manager (security engine)
- Install Wazuh Dashboard (web interface)
- Generate SSL certificates automatically
- Configure all components
- Start all services

**Important:** The script will display admin credentials at the end. Save these credentials - you'll need them to log into the web interface!

Output will look like:
```
INFO: --- Summary ---
INFO: You can access the web interface https://<your-ip>
    User: admin
    Password: <generated-password>
```

### Step 3: Verify Installation
```bash
# Check Wazuh Manager status
sudo systemctl status wazuh-manager

# Check Wazuh Indexer status
sudo systemctl status wazuh-indexer

# Check Wazuh Dashboard status
sudo systemctl status wazuh-dashboard
```

All services should show "active (running)".

---

## Accessing the Dashboard

### 1. Get Your Server IP
```bash
hostname -I
```

### 2. Open Web Browser
Navigate to:
```
https://YOUR_SERVER_IP
```

Example: `https://10.0.2.15`

### 3. Login
- **Username:** `admin`
- **Password:** (the password displayed during installation)

**Note:** You'll see a security warning about the SSL certificate. This is normal for self-signed certificates. Click "Advanced" → "Proceed" to continue.

---

## Setting Up Agents

Agents are installed on the systems you want to monitor. They collect security data and send it to the Wazuh Manager.

### Method 1: Deploy via Dashboard (Easiest)

1. Log into the Wazuh dashboard
2. Navigate to **Agents** → **Deploy new agent**
3. Select your operating system (Windows, Linux, macOS)
4. The dashboard will provide ready-to-use commands
5. Copy and run the commands on the target system

### Method 2: Manual Linux Agent Installation

#### Ubuntu/Debian
```bash
# Download the agent package
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.1-1_amd64.deb

# Install with your manager's IP address
sudo WAZUH_MANAGER='YOUR_MANAGER_IP' dpkg -i ./wazuh-agent_4.14.1-1_amd64.deb

# Start the agent
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent

# Verify the agent is running
sudo systemctl status wazuh-agent
```

Replace `YOUR_MANAGER_IP` with your Wazuh server's IP address (e.g., `10.0.2.15`).

#### CentOS/RHEL
```bash
# Import GPG key
sudo rpm --import https://packages.wazuh.com/key/GPG-KEY-WAZUH

# Add repository
cat > /etc/yum.repos.d/wazuh.repo << EOF
[wazuh]
gpgcheck=1
gpgkey=https://packages.wazuh.com/key/GPG-KEY-WAZUH
enabled=1
name=EL-\$releasever - Wazuh
baseurl=https://packages.wazuh.com/4.x/yum/
protect=1
EOF

# Install agent with your manager's IP
sudo WAZUH_MANAGER='YOUR_MANAGER_IP' yum install wazuh-agent -y

# Start the agent
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

### Method 3: Windows Agent Installation

#### Option A: Via MSI Installer

1. Download the installer:
   ```
   https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.1-1.msi
   ```

2. Run PowerShell as Administrator:
   ```powershell
   # Install with manager IP
   .\wazuh-agent-4.14.1-1.msi /q WAZUH_MANAGER="YOUR_MANAGER_IP"
   
   # Start the service
   NET START WazuhSvc
   ```

#### Option B: Via Dashboard
Use the "Deploy new agent" wizard in the dashboard for Windows-specific instructions.

### Verify Agent Connection

#### From Dashboard:
1. Go to **Agents** in the dashboard
2. You should see your new agent listed with status "Active"

#### From Command Line (on Manager):
```bash
sudo /var/ossec/bin/agent_control -l
```

---

## Useful Commands

### Check Service Status
```bash
# Wazuh Manager
sudo systemctl status wazuh-manager

# Wazuh Indexer
sudo systemctl status wazuh-indexer

# Wazuh Dashboard
sudo systemctl status wazuh-dashboard
```

### Restart Services
```bash
sudo systemctl restart wazuh-manager
sudo systemctl restart wazuh-indexer
sudo systemctl restart wazuh-dashboard
```

### View Logs
```bash
# Wazuh Manager logs
sudo tail -f /var/ossec/logs/ossec.log

# System logs for Wazuh Manager
sudo journalctl -u wazuh-manager -f
```

### Agent Management
```bash
# List all agents
sudo /var/ossec/bin/agent_control -l

# Get specific agent information
sudo /var/ossec/bin/agent_control -i AGENT_ID

# Restart all agents
sudo /var/ossec/bin/agent_control -R -a
```

---

## Quick Troubleshooting

### Agent Not Showing Up in Dashboard

**Check agent status:**
```bash
sudo systemctl status wazuh-agent
```

**Check agent logs:**
```bash
sudo tail -f /var/ossec/logs/ossec.log
```

**Verify manager IP in agent config:**
```bash
sudo cat /var/ossec/etc/ossec.conf | grep address
```

**Restart agent:**
```bash
sudo systemctl restart wazuh-agent
```

### Can't Access Dashboard

**Check if dashboard is running:**
```bash
sudo systemctl status wazuh-dashboard
```

**Restart dashboard:**
```bash
sudo systemctl restart wazuh-dashboard
```

**Check firewall (if applicable):**
```bash
sudo ufw status
# If port 443 is blocked, allow it:
sudo ufw allow 443/tcp
```

---

## What to Monitor

Once your agents are connected, Wazuh automatically monitors:

- **File Integrity Monitoring** - Detects unauthorized file changes
- **Log Analysis** - Analyzes system and application logs
- **Rootkit Detection** - Scans for rootkits and malware
- **Vulnerability Detection** - Identifies security vulnerabilities
- **Security Configuration Assessment** - Checks for misconfigurations
- **Regulatory Compliance** - PCI DSS, GDPR, HIPAA compliance monitoring

Access these features through the dashboard under:
- **Security Events** - Real-time alerts
- **Integrity Monitoring** - File changes
- **Vulnerabilities** - CVE detection
- **Compliance** - Regulatory standards

---

## Additional Resources

- **Official Documentation:** https://documentation.wazuh.com/
- **Wazuh API Docs:** https://documentation.wazuh.com/current/user-manual/api/
- **Community Forum:** https://groups.google.com/forum/#!forum/wazuh
- **GitHub:** https://github.com/wazuh/wazuh

---

## Summary

That's it! With just two commands, you have a fully functional Wazuh SIEM:

```bash
# Download installer
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh

# Install everything
sudo bash wazuh-install.sh -a
```

Then deploy agents on your systems to start monitoring security events.

---

**Last Updated:** December 2025  
**Wazuh Version:** 4.14.1
