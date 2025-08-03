---
layout: post
title: Oracle Cloud Free Tier VM Setup for VPN Server
subtitle: Complete guide to setting up Oracle Cloud free tier infrastructure for hosting VPN services
cover-img: /assets/img/oracle-cloud/oracle.png
thumbnail-img: /assets/img/oracle-cloud/oracle.png
share-img: /assets/img/oracle-cloud/oracle.png
tags: [oracle-cloud, free-tier, vm, vcn, security-lists, networking, infrastructure, vpn-prerequisite]
author: hanisntsolo
---

This comprehensive guide walks you through setting up Oracle Cloud Infrastructure's free tier to host VPN services. This setup serves as a prerequisite for more advanced networking configurations like WireGuard VPN gateways.

## Overview

Oracle Cloud Infrastructure (OCI) offers one of the most generous free tiers in the cloud computing space, including always-free VM instances that are perfect for hosting VPN servers. This guide will help you set up the complete infrastructure foundation needed for VPN services.

## What You'll Get on Oracle Cloud Free Tier

- **2 AMD-based Compute VMs** (1/8 OCPU, 1 GB memory each)
- **1 Arm-based Ampere A1 Compute VM** (up to 4 OCPUs, 24 GB memory)
- **Two Block Volumes** (up to 200 GB total)
- **10 GB Object Storage**
- **10 GB Archive Storage**
- **1 Virtual Cloud Network (VCN)**
- **2 Subnets**
- **1 Internet Gateway**
- **1 NAT Gateway**
- **2 Load Balancers**

For VPN purposes, we'll focus on the Arm-based VM as it provides the best performance.

## Prerequisites

- Oracle Cloud Infrastructure account (sign up at [oracle.com/cloud/free](https://www.oracle.com/cloud/free/))
- Valid email address and phone number
- Credit card for verification (won't be charged for free tier resources)

## Step-by-Step Setup

### 1. Create Oracle Cloud Account

1. Visit [Oracle Cloud Free Tier](https://www.oracle.com/cloud/free/)
2. Click "Start for free"
3. Fill out the registration form:
   - Choose your country/territory
   - Enter personal information
   - Provide a valid phone number
   - Add credit card for verification
4. Verify your email address
5. Complete identity verification

**Important**: Oracle may take up to 24 hours to verify new accounts.

### 2. Access Oracle Cloud Console

1. Sign in to [cloud.oracle.com](https://cloud.oracle.com/)
2. Enter your Cloud Account Name (tenancy)
3. Click "Continue"
4. Sign in with your credentials

### 3. Create a Virtual Cloud Network (VCN)

A VCN is Oracle's version of a virtual private network where your resources will reside.

#### 3.1 Navigate to VCN Creation

1. In the Oracle Cloud Console, click the **hamburger menu** (☰)
2. Go to **Networking** → **Virtual Cloud Networks**
3. Ensure you're in the correct compartment (usually "root" for new accounts)
4. Click **"Start VCN Wizard"**

#### 3.2 Configure VCN Settings

1. Select **"Create VCN with Internet Connectivity"**
2. Click **"Start VCN Wizard"**
3. Configure the following:
   - **VCN Name**: `vpn-vcn`
   - **Compartment**: Select your compartment (usually root)
   - **VCN CIDR Block**: `10.0.0.0/16` (default is fine)
   - **Public Subnet CIDR Block**: `10.0.0.0/24`
   - **Private Subnet CIDR Block**: `10.0.1.0/24`
4. Click **"Next"**
5. Review the configuration and click **"Create"**

#### 3.3 VCN Creation Results

The wizard will create:
- 1 VCN (`vpn-vcn`)
- 1 Public subnet (for your VPN server)
- 1 Private subnet (for internal resources)
- 1 Internet Gateway (for public internet access)
- 1 NAT Gateway (for private subnet internet access)
- 1 Service Gateway (for Oracle services)
- Route tables and security lists

### 4. Configure Security Lists

Security lists act as virtual firewalls for your subnets. We need to open specific ports for VPN and web traffic.

#### 4.1 Access Security Lists

1. From your VCN details page, click **"Security Lists"** in the left menu
2. Click on **"Default Security List for vpn-vcn"**

#### 4.2 Add Ingress Rules

Click **"Add Ingress Rules"** and add the following rules one by one:

**Rule 1: SSH Access**
- **Source Type**: CIDR
- **Source CIDR**: `0.0.0.0/0`
- **IP Protocol**: TCP
- **Destination Port Range**: `22`
- **Description**: `SSH access`

**Rule 2: HTTP Traffic**
- **Source Type**: CIDR
- **Source CIDR**: `0.0.0.0/0`
- **IP Protocol**: TCP
- **Destination Port Range**: `80`
- **Description**: `HTTP traffic`

**Rule 3: HTTPS Traffic**
- **Source Type**: CIDR
- **Source CIDR**: `0.0.0.0/0`
- **IP Protocol**: TCP
- **Destination Port Range**: `443`
- **Description**: `HTTPS traffic`

**Rule 4: Custom HTTP Port**
- **Source Type**: CIDR
- **Source CIDR**: `0.0.0.0/0`
- **IP Protocol**: TCP
- **Destination Port Range**: `81`
- **Description**: `Custom HTTP port`

**Rule 5: WireGuard VPN**
- **Source Type**: CIDR
- **Source CIDR**: `0.0.0.0/0`
- **IP Protocol**: UDP
- **Destination Port Range**: `51820`
- **Description**: `WireGuard VPN`

**Rule 6: OpenVPN (Optional)**
- **Source Type**: CIDR
- **Source CIDR**: `0.0.0.0/0`
- **IP Protocol**: UDP
- **Destination Port Range**: `1194`
- **Description**: `OpenVPN`

#### 4.3 Verify Default Egress Rules

Ensure the following egress rule exists (it should be there by default):
- **Destination Type**: CIDR
- **Destination CIDR**: `0.0.0.0/0`
- **IP Protocol**: All Protocols
- **Description**: `All traffic outbound`

### 5. Create Compute Instance (VM)

Now we'll create the actual virtual machine that will host your VPN server.

#### 5.1 Navigate to Compute Instances

1. In the Oracle Cloud Console, click the **hamburger menu** (☰)
2. Go to **Compute** → **Instances**
3. Click **"Create Instance"**

#### 5.2 Configure Instance Details

**Basic Information:**
- **Name**: `vpn-server`
- **Compartment**: Select your compartment

**Placement:**
- **Availability Domain**: Select any available AD
- **Fault Domain**: Leave as default

**Security:**
- **Shielded Instance**: Leave enabled (recommended)

#### 5.3 Configure Image and Shape

**Image:**
1. Click **"Change Image"**
2. Select **"Canonical Ubuntu"**
3. Choose **"22.04"** (latest LTS version)
4. Click **"Select Image"**

**Shape:**
1. Click **"Change Shape"**
2. Select **"Ampere"** (Arm-based processors)
3. Choose **"VM.Standard.A1.Flex"**
4. Configure resources:
   - **OCPUs**: `2` (you can use up to 4 on free tier)
   - **Memory (GB)**: `12` (you can use up to 24 GB on free tier)
5. Click **"Select Shape"**

#### 5.4 Configure Networking

**Primary VNIC:**
- **Virtual Cloud Network**: Select `vpn-vcn`
- **Subnet**: Select the **public subnet** (should end with `-public-subnet`)
- **Use network security groups**: Leave unchecked
- **Assign a public IPv4 address**: **Checked**
- **IPv6**: Leave unchecked

#### 5.5 Add SSH Keys

**SSH Keys:**
1. Select **"Paste public keys"**
2. If you have an SSH key, paste your public key here
3. If you don't have SSH keys, generate them:

**On Linux/Mac:**
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
cat ~/.ssh/id_rsa.pub
```

**On Windows (PowerShell):**
```powershell
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
Get-Content ~/.ssh/id_rsa.pub
```

4. Copy the public key content and paste it in the SSH Keys field

#### 5.6 Configure Boot Volume

**Boot Volume:**
- **Boot Volume Size (GB)**: `50` (free tier allows up to 200 GB total)
- **Use in-transit encryption**: Checked (recommended)

#### 5.7 Create the Instance

1. Review all configurations
2. Click **"Create"**
3. Wait for the instance to be in **"Running"** state (usually takes 2-3 minutes)

### 6. Connect to Your Instance

#### 6.1 Get Connection Details

1. From the Instance details page, note down:
   - **Public IP Address**: (e.g., `132.145.XXX.XXX`)
   - **Username**: `ubuntu` (for Ubuntu images)

#### 6.2 SSH Connection

**From Linux/Mac Terminal:**
```bash
ssh -i ~/.ssh/id_rsa ubuntu@YOUR_PUBLIC_IP
```

**From Windows (PowerShell or cmd):**
```bash
ssh -i %USERPROFILE%\.ssh\id_rsa ubuntu@YOUR_PUBLIC_IP
```

#### 6.3 First-time Setup

Once connected, update your system:

```bash
# Update package lists
sudo apt update

# Upgrade all packages
sudo apt upgrade -y

# Install essential tools
sudo apt install -y curl wget git htop nano net-tools

# Check system information
hostnamectl
free -h
df -h
```

### 7. Configure Firewall (UFW)

Ubuntu comes with UFW (Uncomplicated Firewall). Let's configure it:

```bash
# Check UFW status
sudo ufw status

# Allow SSH (important - don't lock yourself out!)
sudo ufw allow 22/tcp

# Allow HTTP and HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 81/tcp

# Allow WireGuard
sudo ufw allow 51820/udp

# Allow OpenVPN (optional)
sudo ufw allow 1194/udp

# Enable UFW
sudo ufw enable

# Verify rules
sudo ufw status numbered
```

### 8. Verify Network Configuration

Test your network setup:

```bash
# Check network interfaces
ip addr show

# Test internet connectivity
ping -c 4 google.com

# Check listening ports
sudo ss -tuln

# Test HTTP access from outside
# (try accessing http://YOUR_PUBLIC_IP from your browser)
```

### 9. Set Up Basic Web Server (Optional Test)

To verify everything is working, let's set up a simple web server:

```bash
# Install nginx
sudo apt install nginx -y

# Start and enable nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Check status
sudo systemctl status nginx

# Create a simple test page
echo "<h1>Oracle Cloud VPN Server Ready!</h1>" | sudo tee /var/www/html/index.html

# Test locally
curl localhost

# Test externally by visiting http://YOUR_PUBLIC_IP in your browser
```

### 10. Optimize for VPN Usage

Apply some optimizations for VPN server usage:

```bash
# Enable IP forwarding (required for VPN)
echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv6.conf.all.forwarding=1' | sudo tee -a /etc/sysctl.conf

# Apply sysctl changes
sudo sysctl -p

# Install iptables-persistent for rule persistence
sudo apt install iptables-persistent -y

# Increase file descriptor limits
echo '* soft nofile 65536' | sudo tee -a /etc/security/limits.conf
echo '* hard nofile 65536' | sudo tee -a /etc/security/limits.conf

# Optimize network performance
echo 'net.core.default_qdisc = fq' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_congestion_control = bbr' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Verification Checklist

Before proceeding to VPN setup, verify the following:

### ✅ Infrastructure Checklist

- [ ] Oracle Cloud account is active and verified
- [ ] VCN (`vpn-vcn`) is created with public and private subnets
- [ ] Internet Gateway is attached and configured
- [ ] Security List allows required ports (22, 80, 81, 443, 51820, 1194)
- [ ] VM instance is running with public IP address
- [ ] SSH access is working
- [ ] Basic firewall (UFW) is configured
- [ ] IP forwarding is enabled
- [ ] Internet connectivity is working from the VM

### ✅ Network Connectivity Tests

```bash
# Test from your VM
ping -c 4 8.8.8.8           # Internet connectivity
curl -I google.com          # HTTP connectivity
sudo ss -tuln | grep :80    # Web server listening
sudo ss -tuln | grep :22    # SSH listening

# Test from external network
# SSH: ssh ubuntu@YOUR_PUBLIC_IP
# HTTP: curl http://YOUR_PUBLIC_IP
# Browser: http://YOUR_PUBLIC_IP
```

## Cost Monitoring

Even though we're using free tier resources, it's good practice to monitor usage:

### Set Up Budget Alerts

1. Go to **Governance & Administration** → **Budgets**
2. Click **"Create Budget"**
3. Configure:
   - **Name**: `Free Tier Monitor`
   - **Target**: `$1.00` (to catch any accidental charges)
   - **Alert Rule**: 80% of budget
   - **Email Recipients**: Your email

### Monitor Resource Usage

1. Go to **Governance & Administration** → **Usage Reports**
2. Check your usage against free tier limits
3. Key metrics to watch:
   - Compute hours
   - Block storage GB-months
   - Bandwidth usage

## Security Best Practices

### 1. SSH Key Security

```bash
# Change SSH key permissions (if needed)
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub

# Disable password authentication (optional but recommended)
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

### 2. Automatic Security Updates

```bash
# Enable automatic security updates
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure -plow unattended-upgrades
```

### 3. Fail2Ban (Optional)

```bash
# Install fail2ban for SSH protection
sudo apt install fail2ban -y

# Configure fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

## Troubleshooting Common Issues

### Issue 1: Can't Connect via SSH

**Symptoms**: Connection timeout or "connection refused"

**Solutions**:
1. Check security list rules allow port 22
2. Verify public IP address is correct
3. Ensure SSH key is properly formatted
4. Check if UFW is blocking the connection:
   ```bash
   sudo ufw status
   sudo ufw allow 22/tcp
   ```

### Issue 2: Web Server Not Accessible

**Symptoms**: Can't access HTTP/HTTPS from internet

**Solutions**:
1. Verify security list has ingress rules for ports 80/443
2. Check if nginx is running: `sudo systemctl status nginx`
3. Test locally first: `curl localhost`
4. Check UFW rules: `sudo ufw status`

### Issue 3: Instance Won't Start

**Symptoms**: Instance stuck in "Provisioning" or "Starting" state

**Solutions**:
1. Check service limits in your tenancy
2. Try different availability domain
3. Reduce resource allocation (OCPUs/memory)
4. Contact Oracle Support if using free tier resources

### Issue 4: Out of Credits/Billing Issues

**Symptoms**: Resources being terminated, billing notifications

**Solutions**:
1. Check usage reports for overages
2. Verify you're using "Always Free" eligible resources
3. Set up budget alerts
4. Review and terminate unused resources

## Next Steps

With your Oracle Cloud infrastructure properly set up, you're now ready to:

1. **Install and configure VPN software** like WireGuard or OpenVPN
2. **Set up reverse proxy** with Nginx Proxy Manager
3. **Configure domain and SSL certificates**
4. **Implement monitoring and logging**

For the next step in setting up WireGuard VPN on this infrastructure, check out: [WireGuard VPN Gateway Setup on Oracle Cloud Infrastructure]({{ site.baseurl }}/2025-08-03-wireguard-vpn-gateway-setup-oracle-cloud/).

## Backup and Disaster Recovery

### Create Boot Volume Backup

```bash
# From Oracle Cloud Console:
# 1. Go to Compute → Instances
# 2. Click your instance name
# 3. Under "Boot Volume", click the boot volume name
# 4. Click "Create Manual Backup"
# 5. Name it: vpn-server-backup-YYYY-MM-DD
```

### Configuration Backup Script

Create a backup script for your configurations:

```bash
#!/bin/bash
# backup-config.sh

BACKUP_DIR="/home/ubuntu/backups"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

# Backup important configs
sudo tar -czf $BACKUP_DIR/system-config-$DATE.tar.gz \
    /etc/ssh/sshd_config \
    /etc/ufw/ \
    /etc/nginx/ \
    /etc/wireguard/ \
    /etc/sysctl.conf \
    /etc/iptables/

echo "Backup created: $BACKUP_DIR/system-config-$DATE.tar.gz"
```

## Conclusion

You now have a fully configured Oracle Cloud Infrastructure setup that's ready for VPN services. This foundation provides:

- **Robust networking** with VCN, subnets, and security lists
- **Properly configured firewall** rules at both OCI and OS levels
- **Optimized system settings** for VPN performance
- **Security best practices** implemented
- **Monitoring and alerting** configured

The infrastructure is production-ready and can handle multiple VPN clients while remaining within Oracle's generous free tier limits.

## Additional Resources

- [Oracle Cloud Free Tier](https://www.oracle.com/cloud/free/)
- [Oracle Cloud Documentation](https://docs.oracle.com/en-us/iaas/)
- [Ubuntu Server Guide](https://ubuntu.com/server/docs)
- [UFW Documentation](https://help.ubuntu.com/community/UFW)

---

**Infrastructure Setup Completed**: 2025-08-03  
**Tested and Verified**: All services accessible ✅  
**Status**: Ready for VPN deployment 🚀
