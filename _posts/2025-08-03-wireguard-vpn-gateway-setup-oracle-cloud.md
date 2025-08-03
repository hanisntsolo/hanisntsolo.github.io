---
layout: post
title: WireGuard VPN Gateway Setup on Oracle Cloud Infrastructure
subtitle: Complete guide for setting up WireGuard VPN gateway with port forwarding on Oracle Cloud
cover-img: /assets/img/wireguard/wiregurard.png
thumbnail-img: /assets/img/wireguard/wiregurard.png
share-img: /assets/img/wireguard/wiregurard.png
tags: [wireguard, vpn, oracle-cloud, networking, iptables, port-forwarding, infrastructure]
author: hanisntsolo
---

This guide documents the complete setup of a WireGuard VPN gateway on Oracle Cloud Infrastructure that provides secure VPN access for clients, forwards external traffic (ports 80, 81, 443) to connected clients, maintains Oracle Cloud security while enabling custom routing, and survives reboots with persistent configuration.

## Overview

Setting up a VPN gateway on Oracle Cloud Infrastructure can be tricky due to the platform's default security configurations. This comprehensive guide will walk you through creating a robust WireGuard VPN setup that not only provides secure remote access but also enables port forwarding for hosting services accessible from the internet.

## Architecture


```
Internet Traffic → Oracle Cloud (80.XXX.2XX.XX8) → WireGuard Gateway → Client Laptop
    ↓                           ↓                      ↓               ↓
Port 80/81/443 →        iptables DNAT →        VPN Tunnel →    Nginx Proxy Manager
```

## Prerequisites

Before we begin, ensure you have:

- **Complete Oracle Cloud Infrastructure setup** - If you haven't set up your Oracle Cloud free tier VM yet, follow our comprehensive guide: [Oracle Cloud Free Tier VM Setup for VPN Server]({{ site.baseurl }}/2025-08-03-oracle-cloud-free-tier-vm-setup-for-vpn/)
- Oracle Cloud Infrastructure account with properly configured VM instance
- Ubuntu/Debian-based Oracle Cloud instance with public IP
- VCN with correct security list rules (ports 22, 80, 81, 443, 51820 open)
- SSH access to your server with root/sudo privileges
- Basic understanding of iptables and networking
- **Note**: Replace `80.XXX.2XX.XX8` with your actual public IP throughout this guide

## Server Configuration

### 1. Initial Oracle Cloud Setup

**Important**: Always backup your existing configuration before making changes:

```bash
sudo cp /etc/iptables/rules.v4 /etc/iptables/rules.v4.backup-original
sudo iptables-save > /tmp/oracle-original-rules.txt
```

### 2. Install WireGuard

```bash
sudo apt update
sudo apt install wireguard wireguard-tools -y
```

### 3. Generate WireGuard Keys

```bash
# Generate server private key
wg genkey | sudo tee /etc/wireguard/private.key
sudo chmod 600 /etc/wireguard/private.key

# Generate server public key
sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key

# Generate client keys (repeat for each client)
wg genkey | tee client1-private.key | wg pubkey > client1-public.key
```

### 4. Create WireGuard Server Configuration

Create the main WireGuard configuration file at `/etc/wireguard/wg0.conf`:

```ini
[Interface]
PrivateKey = <SERVER_PRIVATE_KEY>
Address = 10.8.0.1/24
ListenPort = 51820
SaveConfig = false

# Oracle Cloud compatible iptables rules
# Insert rules BEFORE Oracle's default REJECT rules
PostUp = iptables -I INPUT 1 -i wg0 -j ACCEPT
PostUp = iptables -I INPUT 2 -p udp --dport 51820 -j ACCEPT
PostUp = iptables -I INPUT 3 -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
PostUp = iptables -I INPUT 4 -p tcp -m state --state NEW -m tcp --dport 81 -j ACCEPT
PostUp = iptables -I INPUT 5 -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT
PostUp = iptables -I FORWARD 1 -p tcp -d 10.8.0.2 --dport 80 -j ACCEPT
PostUp = iptables -I FORWARD 2 -p tcp -d 10.8.0.2 --dport 81 -j ACCEPT
PostUp = iptables -I FORWARD 3 -p tcp -d 10.8.0.2 --dport 443 -j ACCEPT
PostUp = iptables -I FORWARD 4 -i ens3 -o wg0 -j ACCEPT
PostUp = iptables -I FORWARD 5 -i wg0 -o ens3 -j ACCEPT
PostUp = iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 10.8.0.2:80
PostUp = iptables -t nat -A PREROUTING -p tcp --dport 81 -j DNAT --to-destination 10.8.0.2:81
PostUp = iptables -t nat -A PREROUTING -p tcp --dport 443 -j DNAT --to-destination 10.8.0.2:443
PostUp = iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -j MASQUERADE
PostUp = echo 1 > /proc/sys/net/ipv4/ip_forward

# Cleanup rules on shutdown (reverse order)
PostDown = iptables -t nat -D POSTROUTING -s 10.8.0.0/24 -j MASQUERADE
PostDown = iptables -t nat -D PREROUTING -p tcp --dport 443 -j DNAT --to-destination 10.8.0.2:443
PostDown = iptables -t nat -D PREROUTING -p tcp --dport 81 -j DNAT --to-destination 10.8.0.2:81
PostDown = iptables -t nat -D PREROUTING -p tcp --dport 80 -j DNAT --to-destination 10.8.0.2:80
PostDown = iptables -D FORWARD -i wg0 -o ens3 -j ACCEPT
PostDown = iptables -D FORWARD -i ens3 -o wg0 -j ACCEPT
PostDown = iptables -D FORWARD -p tcp -d 10.8.0.2 --dport 443 -j ACCEPT
PostDown = iptables -D FORWARD -p tcp -d 10.8.0.2 --dport 81 -j ACCEPT
PostDown = iptables -D FORWARD -p tcp -d 10.8.0.2 --dport 80 -j ACCEPT
PostDown = iptables -D INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT
PostDown = iptables -D INPUT -p tcp -m state --state NEW -m tcp --dport 81 -j ACCEPT
PostDown = iptables -D INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
PostDown = iptables -D INPUT -p udp --dport 51820 -j ACCEPT
PostDown = iptables -D INPUT -i wg0 -j ACCEPT

# Client configurations
[Peer]
PublicKey = <CLIENT1_PUBLIC_KEY>
AllowedIPs = 10.8.0.2/32

[Peer]
PublicKey = <CLIENT2_PUBLIC_KEY>
AllowedIPs = 10.8.0.3/32
```

### 5. Modify Oracle Cloud iptables Rules

This is **critical** - Oracle Cloud's default REJECT rules block everything. We need to modify `/etc/iptables/rules.v4`:

```bash
sudo nano /etc/iptables/rules.v4
```

**Key Changes:**

1. **Remove or comment out these lines:**
```bash
# -A INPUT -j REJECT --reject-with icmp-host-prohibited
# -A FORWARD -j REJECT --reject-with icmp-host-prohibited
```

2. **Add explicit ACCEPT rules before any REJECT rules:**
```bash
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 81 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT
-A INPUT -p udp --dport 51820 -j ACCEPT
-A INPUT -i wg0 -j ACCEPT

-A FORWARD -i ens3 -o wg0 -j ACCEPT
-A FORWARD -i wg0 -o ens3 -j ACCEPT
-A FORWARD -p tcp -d 10.8.0.2 --dport 80 -j ACCEPT
-A FORWARD -p tcp -d 10.8.0.2 --dport 81 -j ACCEPT
-A FORWARD -p tcp -d 10.8.0.2 --dport 443 -j ACCEPT
```

**Apply the changes:**
```bash
sudo iptables-restore < /etc/iptables/rules.v4
sudo netfilter-persistent save
```

### 6. Enable IP Forwarding

```bash
echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### 7. Start WireGuard Service

```bash
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
sudo systemctl status wg-quick@wg0
```

## Client Configuration

### Client Configuration File

Create `/etc/wireguard/wg0.conf` on your client device:

```ini
[Interface]
PrivateKey = <CLIENT_PRIVATE_KEY>
Address = 10.8.0.2/24
DNS = 1.1.1.1, 8.8.8.8

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
AllowedIPs = 0.0.0.0/0  # Routes ALL traffic through VPN
Endpoint = 80.XXX.2XX.XX8:51820
PersistentKeepalive = 25
```

**Note**: `AllowedIPs = 0.0.0.0/0` ensures full internet access through the VPN.

### Start Client VPN

```bash
sudo wg-quick up wg0
```

## Verification and Testing

### 1. Server-side Verification

```bash
# Check WireGuard status
sudo wg show

# Verify iptables rules order
sudo iptables -L INPUT -n --line-numbers | head -10
sudo iptables -L FORWARD -n --line-numbers | head -10

# Check NAT rules
sudo iptables -t nat -L PREROUTING -n --line-numbers
sudo iptables -t nat -L POSTROUTING -n --line-numbers

# Check listening ports
ss -tuln | grep -E ':(80|81|443|51820)'
```

### 2. Client-side Verification

```bash
# Check VPN connection
sudo wg show
ip addr show wg0

# Test server connectivity
ping -c 3 10.8.0.1

# Test internet access
curl -I google.com
```

### 3. External Access Testing

From an external network (mobile data, different location):

```bash
# Test HTTP access
curl http://80.XXX.2XX.XX8
curl http://80.XXX.2XX.XX8:81

# Or open in browser
# http://80.XXX.2XX.XX8
```

## Troubleshooting

### Common Issues and Solutions

**1. No internet access from client:**
- Ensure `AllowedIPs = 0.0.0.0/0` in client config
- Verify MASQUERADE rule is active: `sudo iptables -t nat -L POSTROUTING`

**2. External traffic not reaching client:**
- Check iptables rule ordering: Custom rules must be BEFORE REJECT rules
- Verify DNAT rules: `sudo iptables -t nat -L PREROUTING`
- Ensure Oracle Cloud security groups allow ports 80, 81, 443

**3. WireGuard won't start:**
- Check config syntax: `sudo wg-quick up wg0`
- Verify interface doesn't already exist: `ip link show wg0`
- Check permissions on private key: `ls -la /etc/wireguard/private.key`

**4. Rules not persistent after reboot:**
- Ensure rules.v4 is properly modified
- Run: `sudo netfilter-persistent save`
- Check: `sudo systemctl status netfilter-persistent`

### Debugging Commands

```bash
# Monitor traffic in real-time
sudo tcpdump -i wg0 -n

# Check connection tracking
sudo conntrack -L

# View detailed iptables with packet counts
sudo iptables -L -v -n

# Check routing table
ip route show table all
```

## Security Considerations

1. **Firewall Rules**: Only necessary ports are opened
2. **Oracle Cloud Security Groups**: Configure to allow required ports
3. **Key Management**: Private keys have restricted permissions (600)
4. **Client Isolation**: Each client has unique IP in /32 subnet
5. **Regular Updates**: Keep WireGuard and system packages updated

## Performance Optimization

For better performance on Oracle Cloud, consider these optimizations:

```bash
# Optimize for Oracle Cloud network
echo 'net.core.default_qdisc = fq' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_congestion_control = bbr' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Maintenance

### Adding New Clients

1. Generate new key pair
2. Add [Peer] section to server config
3. Reload WireGuard: `sudo systemctl reload wg-quick@wg0`
4. Create client config with new IP (10.8.0.x/24)

### Backup and Restore

```bash
# Backup WireGuard configs
sudo tar -czf wireguard-backup-$(date +%Y%m%d).tar.gz /etc/wireguard/

# Backup iptables rules
sudo iptables-save > iptables-backup-$(date +%Y%m%d).txt
```

## Conclusion

This setup provides a robust VPN gateway solution on Oracle Cloud Infrastructure that maintains Oracle's security model while enabling custom routing, provides secure remote access to local services, allows hosting local applications with global accessibility, and survives reboots with persistent configuration.

The key to success was understanding Oracle Cloud's default firewall behavior and properly ordering iptables rules to work with existing security policies.

## Additional Resources

- [WireGuard Official Documentation](https://www.wireguard.com/)
- [Oracle Cloud Networking](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
- [iptables Tutorial](https://www.netfilter.org/documentation/HOWTO/packet-filtering-HOWTO.html)

---

**Setup completed**: 2025-08-03  
**Tested and verified**: External access working ✅  
**Status**: Production ready 🚀
