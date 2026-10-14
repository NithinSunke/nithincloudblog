---
title: "OCI ↔ Homelab Private Connectivity Using Tailscale Subnet Routers"
author: "Sunke Nithin"
date: 2026-06-10
categories: ["CLOUD", "MY LAB SETUP"]
tags: ["Network"]
---

# OCI ↔ Homelab Private Connectivity Using Tailscale Subnet Routers

This document captures the working setup used to connect an OCI VCN to a Proxmox homelab using Tailscale subnet routers. It includes the exact commands for the OCI router, homelab router, Windows and Linux clients, plus the fixes that resolved routing conflicts.

# 1. Final Architecture

The deployed topology is:

``` wp-block-code
OCI VCN: 10.0.0.0/16
OCI Router VM (tsrtr): 10.0.30.254
OCI Tailscale IP: 100.88.125.4

Homelab LAN: 192.168.1.0/24
Proxmox LXC Tailscale Router: 192.168.1.250
Homelab Tailscale IP: 100.x.x.x

Traffic flow:
Windows/Linux client -> 192.168.1.250 -> Tailscale -> 10.0.30.254 -> OCI private VM
OCI VM -> 10.0.30.254 -> Tailscale -> 192.168.1.250 -> homelab server
```

The OCI router and the homelab router both run Tailscale and advertise their local subnets as routes.

# 2. Address Plan

- OCI VCN CIDR: 10.0.0.0/16
- OCI subnet containing the router VM: 10.0.16.0/20
- OCI router VM private IP: 10.0.30.254
- Homelab LAN: 192.168.1.0/24
- Proxmox LXC router IP: 192.168.1.250
- Conflicting Windows route that was removed: 10.0.10.0/24 via 192.168.1.6

# 3. OCI Router (tsrtr) Commands

Install Tailscale on AlmaLinux / Oracle Linux / other minimal OCI images:

``` wp-block-code
sudo dnf update -y
sudo dnf install -y curl
curl -fsSL https://tailscale.com/install.sh | sh
sudo systemctl enable --now tailscaled
sudo tailscale up
```

Verify installation and connectivity:

``` wp-block-code
tailscale version
tailscale status
tailscale debug prefs
tailscale debug prefs | grep -A5 AdvertiseRoutes
ip addr
ip route
cat /proc/sys/net/ipv4/ip_forward

Enable forwarding permanently and immediately:

sudo tee /etc/sysctl.d/99-tailscale.conf <<EOF
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
EOF
sudo sysctl --system

Advertise the OCI VCN route and allow return routes if needed:

sudo tailscale up --advertise-routes=10.0.0.0/16
sudo tailscale up --accept-routes --advertise-routes=10.0.0.0/16
```

After this, approve 10.0.0.0/16 in the Tailscale admin console for tsrtr.

# 4. Homelab Router (Proxmox LXC) Commands

Example LXC setup: Debian 12 container with static IP 192.168.1.250/24.

On the Proxmox host, enable the TUN device for the container. Replace 201 with your container ID:

``` wp-block-code
pct stop 201
nano /etc/pve/lxc/201.conf

# Add these lines:
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file

pct start 201

Inside the LXC, install Tailscale and start the daemon:

apt update
apt install -y curl
curl -fsSL https://tailscale.com/install.sh | sh
systemctl enable --now tailscaled
tailscale up

Enable forwarding inside the LXC:

cat >/etc/sysctl.d/99-tailscale.conf <<EOF
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
EOF
sysctl --system
cat /proc/sys/net/ipv4/ip_forward

Advertise the homelab subnet:

tailscale set --advertise-routes=192.168.1.0/24
# or
sudo tailscale up --advertise-routes=192.168.1.0/24

Verify the route advertisement:

tailscale status
tailscale debug prefs | grep -A5 AdvertiseRoutes
```

Approve 192.168.1.0/24 in the Tailscale admin console for the container.

# 5. OCI Console Steps

- Enable Skip Source/Destination Check on the OCI router VNIC.
- In the OCI VCN route table attached to the OCI subnet, add a route to the homelab subnet: 192.168.1.0/24 -\> private IP of tsrtr (10.0.30.254).
- Attach that route table to the OCI subnet that contains your OCI workloads.
- Allow traffic in the OCI Security Lists / NSGs for the flows you want to test (ICMP, SSH, HTTPS, etc.).

# 6. Client Route Commands

Windows client: keep only the correct route to OCI through the homelab router.

``` wp-block-code
route print
Get-NetRoute -DestinationPrefix 10.0.0.0/16 | Sort-Object RouteMetric

route -p add 10.0.0.0 mask 255.255.0.0 192.168.1.250
route delete 10.0.10.0 mask 255.255.255.0 192.168.1.6
route -p delete 10.0.10.0 mask 255.255.255.0 192.168.1.6
route print 10.0.*
```

Linux client: add a temporary route through the homelab router.

``` wp-block-code
ip route add 10.0.0.0/16 via 192.168.1.250
ip route
ip route del 10.0.0.0/16 via 192.168.1.250
```

If you want the route to be persistent on a Proxmox host, you can place it in /etc/network/interfaces under vmbr0 using post-up / post-down or up / down lines. Example:

``` wp-block-code
post-up /sbin/ip route replace 10.0.0.0/16 via 192.168.1.250 dev vmbr0
post-down /sbin/ip route del 10.0.0.0/16 via 192.168.1.250 dev vmbr0 || true
```

# 7. Validation and Testing Commands

Use these to verify the tunnel and the routes end-to-end:

``` wp-block-code
# OCI router checks
tailscale status
cat /proc/sys/net/ipv4/ip_forward
ip route

# Homelab router checks
tailscale status
tailscale debug prefs | grep -A5 AdvertiseRoutes
cat /proc/sys/net/ipv4/ip_forward
ip route

# Basic reachability tests
ping 192.168.1.250
ping 10.0.30.254
ping 10.0.10.147
tracert 10.0.30.254   # Windows
tracert 10.0.10.147   # Windows
traceroute 10.0.30.254 # Linux
traceroute 10.0.10.147 # Linux

# Service-level test
Test-NetConnection 10.0.10.147 -Port 22
```

Expected behavior: Windows and Linux clients should reach the OCI router at 10.0.30.254, and then OCI private VMs such as 10.0.10.147, once route conflicts are removed.

# 8. Troubleshooting Commands and Fixes

If Tailscale says "Some peers are advertising routes but --accept-routes is false", run:

``` wp-block-code
sudo tailscale up --accept-routes --advertise-routes=10.0.0.0/16
```

This happened because the OCI machine was seeing another Tailscale peer advertise routes. The fix was to enable accept-routes on tsrtr.

If the route exists but a specific subnet is failing from Windows, check for a more specific conflicting route. The case that caused the issue here was:

``` wp-block-code
Get-NetRoute -DestinationPrefix 10.0.0.0/16 | Sort-Object RouteMetric

#Problem route that had to be removed:
route delete 10.0.10.0 mask 255.255.255.0 192.168.1.6
```

If the homelab Linux VM can reach OCI but Windows cannot, the most likely cause is a stale or more specific Windows route, not Tailscale itself.

If Windows can reach 10.0.30.254 but not 10.0.10.147, that was the exact symptom of the conflicting /24 route overriding the /16 route.

# 9. Clean Command Checklist

This is the short checklist of commands used in the final working setup.

``` wp-block-code
# OCI router
sudo dnf update -y
sudo dnf install -y curl
curl -fsSL https://tailscale.com/install.sh | sh
sudo systemctl enable --now tailscaled
sudo tailscale up
sudo tee /etc/sysctl.d/99-tailscale.conf <<EOF
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
EOF
sudo sysctl --system
sudo tailscale up --advertise-routes=10.0.0.0/16
sudo tailscale up --accept-routes --advertise-routes=10.0.0.0/16

# Homelab LXC router
pct stop 201
nano /etc/pve/lxc/201.conf
pct start 201
apt update
apt install -y curl
curl -fsSL https://tailscale.com/install.sh | sh
systemctl enable --now tailscaled
tailscale up
cat >/etc/sysctl.d/99-tailscale.conf <<EOF
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
EOF
sysctl --system
tailscale set --advertise-routes=192.168.1.0/24

# Windows
route -p add 10.0.0.0 mask 255.255.0.0 192.168.1.250
route delete 10.0.10.0 mask 255.255.255.0 192.168.1.6
route -p delete 10.0.10.0 mask 255.255.255.0 192.168.1.6
route print

# Linux client
ip route add 10.0.0.0/16 via 192.168.1.250
ip route del 10.0.0.0/16 via 192.168.1.250
```
