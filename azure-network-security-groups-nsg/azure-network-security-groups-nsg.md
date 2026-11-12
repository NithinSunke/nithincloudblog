<div class="wrap">

<a href="../index.html" class="back-home">← All articles</a>

<div>

<div class="eyebrow">

Azure fundamentals · AZ-900 · Hands-on lab

</div>

# Azure Network Security Groups (NSG): Why SSH Timed Out and How One Rule Fixed It

I deployed an Ubuntu VM with a public IP into `SUBNET1`, tried to SSH
in, and the connection timed out. This post explains why, then walks
through creating an NSG, attaching it to the subnet and adding an SSH
rule. Along the way it covers how NSG rules are evaluated, the default
rules, service tags, application security groups and how to
troubleshoot.

<div class="meta">

Sunke NithinNovember 2026~15 min read + 15 min lab

</div>

</div>

[1. The problem: SSH timed out](#problem) [2. What is an NSG?](#what)
[3. Anatomy of a rule](#rule) [4. The six default rules](#defaults) [5.
How rules are evaluated](#order) [6. Subnet NSG or NIC NSG?](#where) [7.
Service tags and ASGs](#tags) [8. Lab: fix SSH with an NSG](#lab) [9.
The same lab with Azure CLI](#cli) [10. Lock it down properly](#harden)
[11. Troubleshooting toolkit](#troubleshoot) [12. Quiz](#quiz) [13. Key
takeaways](#takeaways)

## 1. The problem: SSH timed out

The setup was simple: VM `vnet1srv01` (Ubuntu 26.04 LTS) in
`VNET-SP1 / SUBNET1` with private IP `10.0.0.4` and a public IP. The
VM's **Network settings** page showed *Network security group: –* and
*Effective security rules: 0*. With no NSG anywhere, you might expect
everything to be open. Instead, SSH to the public IP on port 22 failed
with *"connection timed out"*.

The reason: the VM's public IP uses the **Standard SKU**, and Standard
public IPs are **secure by default**. Inbound traffic is blocked unless
a network security group explicitly allows it. With no NSG there was
nothing to allow port 22, so packets were silently dropped. A *timeout*
(rather than "connection refused") is the typical sign of traffic being
dropped by a filter before it reaches the VM.

<figure>
<div class="diagram">
<img
src="data:image/svg+xml;base64,PHN2ZyB2aWV3Ym94PSIwIDAgNzIwIDIzMCIgcm9sZT0iaW1nIiBhcmlhLWxhYmVsbGVkYnk9ImYxdCI+CiAgPHRpdGxlIGlkPSJmMXQiPkJlZm9yZTogU1NIIGZyb20gdGhlIGxhcHRvcCB0byB0aGUgU3RhbmRhcmQgcHVibGljIElQIGlzIGRyb3BwZWQgYmVjYXVzZSBubyBOU0cgYWxsb3dzIGl0LiBBZnRlcjogbnNnMSBvbiBTVUJORVQxIGFsbG93cyBUQ1AgMjIsIHNvIFNTSCByZWFjaGVzIHRoZSBWTS48L3RpdGxlPgogIDxkZWZzPgogICAgPG1hcmtlciBpZD0ib2siIHZpZXdib3g9IjAgMCAxMCAxMCIgcmVmeD0iOSIgcmVmeT0iNSIgbWFya2Vyd2lkdGg9IjciIG1hcmtlcmhlaWdodD0iNyIgb3JpZW50PSJhdXRvIj48cGF0aCBkPSJNMCwwIEwxMCw1IEwwLDEwIHoiIHN0eWxlPSJmaWxsOnZhcigtLWMtYWxsb3cpIj48L3BhdGg+PC9tYXJrZXI+CiAgPC9kZWZzPgogIDx0ZXh0IHg9IjIwIiB5PSIzMCIgY2xhc3M9InN2LXQiIHN0eWxlPSJmb250LXNpemU6MTRweCI+QmVmb3JlPC90ZXh0PgogIDxyZWN0IHg9IjIwIiB5PSI0NCIgd2lkdGg9IjExMCIgaGVpZ2h0PSI0NCIgcng9IjgiIGNsYXNzPSJzdi1ib3giIHN0eWxlPSJzdHJva2U6dmFyKC0tbXV0ZWQpIj48L3JlY3Q+PHRleHQgeD0iNzUiIHk9IjcxIiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBjbGFzcz0ic3YtdCIgc3R5bGU9ImZvbnQtc2l6ZToxM3B4Ij5NeSBsYXB0b3A8L3RleHQ+CiAgPHBhdGggZD0iTTEzMiA2NiBIMjcwIiBzdHlsZT0ic3Ryb2tlOnZhcigtLWMtZGVueSk7c3Ryb2tlLXdpZHRoOjI7ZmlsbDpub25lO3N0cm9rZS1kYXNoYXJyYXk6NiA0Ij48L3BhdGg+CiAgPHRleHQgeD0iMjAwIiB5PSI1OCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgY2xhc3M9InN2LW0iIHN0eWxlPSJmb250LXNpemU6MTFweCI+c3NoIDoyMjwvdGV4dD4KICA8cmVjdCB4PSIyNzIiIHk9IjQ0IiB3aWR0aD0iMTUwIiBoZWlnaHQ9IjQ0IiByeD0iOCIgY2xhc3M9InN2LWJveCIgc3R5bGU9InN0cm9rZTp2YXIoLS1jLXBpcCkiPjwvcmVjdD4KICA8dGV4dCB4PSIzNDciIHk9IjY0IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBjbGFzcz0ic3YtdCIgc3R5bGU9ImZvbnQtc2l6ZToxM3B4Ij5TdGFuZGFyZCBwdWJsaWMgSVA8L3RleHQ+CiAgPHRleHQgeD0iMzQ3IiB5PSI4MCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgY2xhc3M9InN2LXMiPmNsb3NlZCBieSBkZWZhdWx0PC90ZXh0PgogIDx0ZXh0IHg9IjQ0MCIgeT0iNzEiIGNsYXNzPSJzdi10IiBzdHlsZT0iZmlsbDp2YXIoLS1jLWRlbnkpO2ZvbnQtc2l6ZToyMnB4Ij7inJU8L3RleHQ+CiAgPHRleHQgeD0iNDcwIiB5PSI2NCIgY2xhc3M9InN2LXMiPm5vIE5TRyDihpIgbm8gYWxsb3cgcnVsZTwvdGV4dD4KICA8dGV4dCB4PSI0NzAiIHk9IjgwIiBjbGFzcz0ic3YtcyI+4oaSIGRyb3BwZWQg4oaSICZxdW90O3RpbWVkIG91dCZxdW90OzwvdGV4dD4KCiAgPHRleHQgeD0iMjAiIHk9IjE0MCIgY2xhc3M9InN2LXQiIHN0eWxlPSJmb250LXNpemU6MTRweCI+QWZ0ZXI8L3RleHQ+CiAgPHJlY3QgeD0iMjAiIHk9IjE1NCIgd2lkdGg9IjExMCIgaGVpZ2h0PSI0NCIgcng9IjgiIGNsYXNzPSJzdi1ib3giIHN0eWxlPSJzdHJva2U6dmFyKC0tbXV0ZWQpIj48L3JlY3Q+PHRleHQgeD0iNzUiIHk9IjE4MSIgdGV4dC1hbmNob3I9Im1pZGRsZSIgY2xhc3M9InN2LXQiIHN0eWxlPSJmb250LXNpemU6MTNweCI+TXkgbGFwdG9wPC90ZXh0PgogIDxwYXRoIGQ9Ik0xMzIgMTc2IEgxOTgiIHN0eWxlPSJzdHJva2U6dmFyKC0tYy1hbGxvdyk7c3Ryb2tlLXdpZHRoOjI7ZmlsbDpub25lIiBtYXJrZXItZW5kPSJ1cmwoI29rKSI+PC9wYXRoPgogIDxyZWN0IHg9IjIwMCIgeT0iMTU0IiB3aWR0aD0iMTMwIiBoZWlnaHQ9IjQ0IiByeD0iOCIgY2xhc3M9InN2LWJveCIgc3R5bGU9InN0cm9rZTp2YXIoLS1jLXBpcCkiPjwvcmVjdD4KICA8dGV4dCB4PSIyNjUiIHk9IjE4MSIgdGV4dC1hbmNob3I9Im1pZGRsZSIgY2xhc3M9InN2LXQiIHN0eWxlPSJmb250LXNpemU6MTNweCI+UHVibGljIElQPC90ZXh0PgogIDxwYXRoIGQ9Ik0zMzIgMTc2IEgzNjgiIHN0eWxlPSJzdHJva2U6dmFyKC0tYy1hbGxvdyk7c3Ryb2tlLXdpZHRoOjI7ZmlsbDpub25lIiBtYXJrZXItZW5kPSJ1cmwoI29rKSI+PC9wYXRoPgogIDxyZWN0IHg9IjM3MCIgeT0iMTQ2IiB3aWR0aD0iMTcwIiBoZWlnaHQ9IjYwIiByeD0iOCIgY2xhc3M9InN2LWJveCIgc3R5bGU9InN0cm9rZTp2YXIoLS1jLWluKTtzdHJva2Utd2lkdGg6MiI+PC9yZWN0PgogIDx0ZXh0IHg9IjQ1NSIgeT0iMTcwIiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBjbGFzcz0ic3YtdCIgc3R5bGU9ImZvbnQtc2l6ZToxM3B4Ij5uc2cxIG9uIFNVQk5FVDE8L3RleHQ+CiAgPHRleHQgeD0iNDU1IiB5PSIxOTAiIHRleHQtYW5jaG9yPSJtaWRkbGUiIGNsYXNzPSJzdi1tIiBzdHlsZT0iZm9udC1zaXplOjExcHgiPjEwMCBzc2ggQWxsb3cgVENQIDIyPC90ZXh0PgogIDxwYXRoIGQ9Ik01NDIgMTc2IEg1NzgiIHN0eWxlPSJzdHJva2U6dmFyKC0tYy1hbGxvdyk7c3Ryb2tlLXdpZHRoOjI7ZmlsbDpub25lIiBtYXJrZXItZW5kPSJ1cmwoI29rKSI+PC9wYXRoPgogIDxyZWN0IHg9IjU4MCIgeT0iMTU0IiB3aWR0aD0iMTIwIiBoZWlnaHQ9IjQ0IiByeD0iOCIgY2xhc3M9InN2LWJveCIgc3R5bGU9InN0cm9rZTp2YXIoLS1tdXRlZCkiPjwvcmVjdD4KICA8dGV4dCB4PSI2NDAiIHk9IjE3NCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgY2xhc3M9InN2LXQiIHN0eWxlPSJmb250LXNpemU6MTNweCI+dm5ldDFzcnYwMTwvdGV4dD4KICA8dGV4dCB4PSI2NDAiIHk9IjE5MCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgY2xhc3M9InN2LW0iIHN0eWxlPSJmb250LXNpemU6MTFweCI+MTAuMC4wLjQ8L3RleHQ+Cjwvc3ZnPg==" />
</div>
<figcaption>Figure 1. A Standard public IP needs an NSG that allows the
traffic. Adding nsg1 with one rule opened SSH.</figcaption>
</figure>

<div class="note">

**Two things that are easy to mix up.** The *private subnet* setting on
SUBNET1 (from the [VNet
post](../azure-vnet-private-subnet-lab/azure-vnet-private-subnet-lab.html))
controls *outbound* internet access for VMs *without* a public IP. It
has nothing to do with this problem. Inbound SSH was blocked by the
Standard public IP's secure-by-default behaviour, and an NSG is what
opens it.

</div>

## 2. What is an NSG?

A **network security group (NSG)** is Azure's built-in, distributed
firewall for traffic in and out of a virtual network. It is a list of
**allow and deny rules** that Azure enforces at the network interface of
each VM, so there is no appliance to size, patch or pay for. NSGs are
free.

- An NSG is a regional resource in a resource group (here `nsg1` in
  `Network-RG`, West US 3). It can only be attached to subnets and NICs
  in the same region.
- You attach it to a **subnet**, a **network interface (NIC)**, or both.
  One NSG can be reused on many subnets and NICs.
- NSGs are **stateful**: if an inbound rule allows a connection in, the
  reply traffic is allowed back out automatically, and vice versa. You
  never write a rule for return traffic.
- They filter at layers 3 and 4 (IP addresses, ports, protocols). For
  URL filtering, TLS inspection or threat intelligence you need Azure
  Firewall or a third-party appliance.

<div class="note">

**Coming from OCI?** An Azure NSG combines what OCI splits into security
lists (attached to a subnet) and network security groups (attached to
VNICs). Like OCI rules, Azure rules are stateful by default; unlike OCI,
Azure rules have a **priority** and can explicitly **deny**.

</div>

## 3. Anatomy of a rule

This is the rule from the lab, with every field explained:

<div class="tbl">

| Field                          | Lab value    | What it means                                                                                         |
|--------------------------------|--------------|-------------------------------------------------------------------------------------------------------|
| **Priority**                   | `100`        | 100 to 4096. Lower numbers are checked first. Must be unique per direction within the NSG.            |
| **Name**                       | `ssh`        | Unique within the NSG. Name rules by purpose, for example `allow-ssh-from-home`.                      |
| **Source**                     | Any          | Any, an IP or CIDR range, "My IP address", a **service tag** or an **application security group**.    |
| **Source port ranges**         | `*`          | Almost always `*`: clients pick a random source port.                                                 |
| **Destination**                | Any          | Any, IP/CIDR, service tag or ASG. "Any" here means any address the NSG protects.                      |
| **Service / destination port** | Custom, `22` | A single port, a range (`8000-8100`) or a list (`80,443`). Presets like SSH, RDP, HTTPS fill this in. |
| **Protocol**                   | Any          | TCP, UDP, ICMPv4, ICMPv6 or Any. SSH only needs **TCP**, so TCP would be tighter.                     |
| **Action**                     | Allow        | Allow or Deny.                                                                                        |

</div>

The portal showed a **warning triangle** next to the `ssh` rule. That's
Azure flagging that port 22 is open to the whole internet. Bots scan
every Azure public IP for SSH and RDP within minutes, so section 10
tightens this rule.

## 4. The six default rules

A new NSG isn't empty. It has six default rules at priorities 65000 and
above. You can't delete them, but you can override them with your own
rules at lower numbers. This is exactly what `nsg1` showed straight
after creation:

<div class="tbl">

| Dir. | Priority | Name                          | Source → Destination            | Action |
|------|----------|-------------------------------|---------------------------------|--------|
| In   | 65000    | AllowVnetInBound              | VirtualNetwork → VirtualNetwork | Allow  |
| In   | 65001    | AllowAzureLoadBalancerInBound | AzureLoadBalancer → Any         | Allow  |
| In   | 65500    | DenyAllInBound                | Any → Any                       | Deny   |
| Out  | 65000    | AllowVnetOutBound             | VirtualNetwork → VirtualNetwork | Allow  |
| Out  | 65001    | AllowInternetOutBound         | Any → Internet                  | Allow  |
| Out  | 65500    | DenyAllOutBound               | Any → Any                       | Deny   |

</div>

In plain words: **inside the VNet, everything can talk to everything**;
Azure load balancer health probes are allowed in; **nothing from the
internet gets in**; and **outbound to the internet is allowed**. The
`VirtualNetwork` tag covers the VNet's address space plus peered VNets
and connected on-premises ranges.

So attaching `nsg1` with only the defaults would still have blocked SSH,
because *DenyAllInBound* would catch it. The custom rule at priority 100
is what let it through.

## 5. How rules are evaluated

For each new connection, Azure checks the rules for that direction **in
priority order, lowest number first**. The **first rule that matches**
decides the result and evaluation stops. Rules with higher numbers are
never looked at.

<figure>
<div class="diagram">
<img
src="data:image/svg+xml;base64,PHN2ZyB2aWV3Ym94PSIwIDAgNzIwIDI3MCIgcm9sZT0iaW1nIiBhcmlhLWxhYmVsbGVkYnk9ImYydCI+CiAgPHRpdGxlIGlkPSJmMnQiPkFuIGluYm91bmQgU1NIIHBhY2tldCBpcyBjaGVja2VkIGFnYWluc3QgcnVsZXMgaW4gcHJpb3JpdHkgb3JkZXIuIFJ1bGUgMTAwIHNzaCBtYXRjaGVzIGZpcnN0IGFuZCBhbGxvd3MgaXQ7IHRoZSBsYXRlciBydWxlcyBhcmUgbmV2ZXIgZXZhbHVhdGVkLjwvdGl0bGU+CiAgPGRlZnM+PG1hcmtlciBpZD0iYTIiIHZpZXdib3g9IjAgMCAxMCAxMCIgcmVmeD0iOSIgcmVmeT0iNSIgbWFya2Vyd2lkdGg9IjciIG1hcmtlcmhlaWdodD0iNyIgb3JpZW50PSJhdXRvIj48cGF0aCBkPSJNMCwwIEwxMCw1IEwwLDEwIHoiIHN0eWxlPSJmaWxsOnZhcigtLW11dGVkKSI+PC9wYXRoPjwvbWFya2VyPjwvZGVmcz4KICA8cmVjdCB4PSIyMCIgeT0iMjAiIHdpZHRoPSIyMDAiIGhlaWdodD0iNjAiIHJ4PSI4IiBjbGFzcz0ic3YtYm94IiBzdHlsZT0ic3Ryb2tlOnZhcigtLWMtaW4pIj48L3JlY3Q+CiAgPHRleHQgeD0iMTIwIiB5PSI0NCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgY2xhc3M9InN2LXQiIHN0eWxlPSJmb250LXNpemU6MTNweCI+SW5ib3VuZCBwYWNrZXQ8L3RleHQ+CiAgPHRleHQgeD0iMTIwIiB5PSI2NCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgY2xhc3M9InN2LW0iIHN0eWxlPSJmb250LXNpemU6MTFweCI+aW50ZXJuZXQg4oaSIDEwLjAuMC40IFRDUCAyMjwvdGV4dD4KICA8cGF0aCBkPSJNMjIyIDUwIEgyNjgiIGNsYXNzPSJzdi1saW5lIiBtYXJrZXItZW5kPSJ1cmwoI2EyKSI+PC9wYXRoPgogIDxnIGNsYXNzPSJzdi1tIiBzdHlsZT0iZm9udC1zaXplOjEycHgiPgogICAgPHJlY3QgeD0iMjcwIiB5PSIyMCIgd2lkdGg9IjQzMCIgaGVpZ2h0PSI0NCIgcng9IjYiIHN0eWxlPSJmaWxsOnZhcigtLWMtYWxsb3cpO29wYWNpdHk6LjE4Ij48L3JlY3Q+CiAgICA8dGV4dCB4PSIyODQiIHk9IjQ3Ij4xMDA8L3RleHQ+PHRleHQgeD0iMzQwIiB5PSI0NyI+c3NoIMK3IEFueSDihpIgQW55IMK3IDIyPC90ZXh0Pjx0ZXh0IHg9IjYxMCIgeT0iNDciIGNsYXNzPSJhbGxvdyI+QWxsb3cg4pyTIG1hdGNoPC90ZXh0PgogICAgPHJlY3QgeD0iMjcwIiB5PSI3NCIgd2lkdGg9IjQzMCIgaGVpZ2h0PSI0MCIgcng9IjYiIGNsYXNzPSJzdi1ib3giIHN0eWxlPSJzdHJva2U6dmFyKC0tbGluZSk7b3BhY2l0eTouNTUiPjwvcmVjdD4KICAgIDx0ZXh0IHg9IjI4NCIgeT0iOTkiIHN0eWxlPSJvcGFjaXR5Oi41NSI+NjUwMDA8L3RleHQ+PHRleHQgeD0iMzQwIiB5PSI5OSIgc3R5bGU9Im9wYWNpdHk6LjU1Ij5BbGxvd1ZuZXRJbkJvdW5kPC90ZXh0Pjx0ZXh0IHg9IjYxMCIgeT0iOTkiIHN0eWxlPSJvcGFjaXR5Oi41NSI+bm90IGNoZWNrZWQ8L3RleHQ+CiAgICA8cmVjdCB4PSIyNzAiIHk9IjEyMiIgd2lkdGg9IjQzMCIgaGVpZ2h0PSI0MCIgcng9IjYiIGNsYXNzPSJzdi1ib3giIHN0eWxlPSJzdHJva2U6dmFyKC0tbGluZSk7b3BhY2l0eTouNTUiPjwvcmVjdD4KICAgIDx0ZXh0IHg9IjI4NCIgeT0iMTQ3IiBzdHlsZT0ib3BhY2l0eTouNTUiPjY1MDAxPC90ZXh0Pjx0ZXh0IHg9IjM0MCIgeT0iMTQ3IiBzdHlsZT0ib3BhY2l0eTouNTUiPkFsbG93QXp1cmVMb2FkQmFsYW5jZXJJbkJvdW5kPC90ZXh0Pjx0ZXh0IHg9IjYxMCIgeT0iMTQ3IiBzdHlsZT0ib3BhY2l0eTouNTUiPm5vdCBjaGVja2VkPC90ZXh0PgogICAgPHJlY3QgeD0iMjcwIiB5PSIxNzAiIHdpZHRoPSI0MzAiIGhlaWdodD0iNDAiIHJ4PSI2IiBjbGFzcz0ic3YtYm94IiBzdHlsZT0ic3Ryb2tlOnZhcigtLWxpbmUpO29wYWNpdHk6LjU1Ij48L3JlY3Q+CiAgICA8dGV4dCB4PSIyODQiIHk9IjE5NSIgc3R5bGU9Im9wYWNpdHk6LjU1Ij42NTUwMDwvdGV4dD48dGV4dCB4PSIzNDAiIHk9IjE5NSIgc3R5bGU9Im9wYWNpdHk6LjU1Ij5EZW55QWxsSW5Cb3VuZDwvdGV4dD48dGV4dCB4PSI2MTAiIHk9IjE5NSIgc3R5bGU9Im9wYWNpdHk6LjU1Ij5ub3QgY2hlY2tlZDwvdGV4dD4KICA8L2c+CiAgPHRleHQgeD0iMjcwIiB5PSIyNDAiIGNsYXNzPSJzdi1zIj5BIHBhY2tldCB0byBwb3J0IDMzODkgd291bGQgc2tpcCBydWxlIDEwMCAobm8gbWF0Y2gpLCBza2lwIDY1MDAwIGFuZCA2NTAwMSw8L3RleHQ+CiAgPHRleHQgeD0iMjcwIiB5PSIyNTYiIGNsYXNzPSJzdi1zIj5hbmQgYmUgZGVuaWVkIGJ5IDY1NTAwLjwvdGV4dD4KPC9zdmc+" />
</div>
<figcaption>Figure 2. First match wins. Leave gaps between priorities
(100, 110, 120…) so you can insert rules later.</figcaption>
</figure>

A common mistake is a broad Deny at a low number above a narrow Allow at
a higher number. For example, *Deny Any at 100* and *Allow 443 at 200*
blocks HTTPS, because rule 100 matches first.

## 6. Subnet NSG or NIC NSG?

You can attach NSGs at two levels, and when both exist, traffic must be
allowed by **both**:

<figure>
<div class="diagram">
<img
src="data:image/svg+xml;base64,PHN2ZyB2aWV3Ym94PSIwIDAgNzIwIDIwMCIgcm9sZT0iaW1nIiBhcmlhLWxhYmVsbGVkYnk9ImYzdCI+CiAgPHRpdGxlIGlkPSJmM3QiPkluYm91bmQgdHJhZmZpYyBwYXNzZXMgdGhlIHN1Ym5ldCBOU0cgZmlyc3QsIHRoZW4gdGhlIE5JQyBOU0cuIE91dGJvdW5kIHRyYWZmaWMgcGFzc2VzIHRoZSBOSUMgTlNHIGZpcnN0LCB0aGVuIHRoZSBzdWJuZXQgTlNHLjwvdGl0bGU+CiAgPGRlZnM+PG1hcmtlciBpZD0iYTMiIHZpZXdib3g9IjAgMCAxMCAxMCIgcmVmeD0iOSIgcmVmeT0iNSIgbWFya2Vyd2lkdGg9IjciIG1hcmtlcmhlaWdodD0iNyIgb3JpZW50PSJhdXRvIj48cGF0aCBkPSJNMCwwIEwxMCw1IEwwLDEwIHoiIHN0eWxlPSJmaWxsOnZhcigtLWMtaW4pIj48L3BhdGg+PC9tYXJrZXI+CiAgPG1hcmtlciBpZD0iYTQiIHZpZXdib3g9IjAgMCAxMCAxMCIgcmVmeD0iOSIgcmVmeT0iNSIgbWFya2Vyd2lkdGg9IjciIG1hcmtlcmhlaWdodD0iNyIgb3JpZW50PSJhdXRvIj48cGF0aCBkPSJNMCwwIEwxMCw1IEwwLDEwIHoiIHN0eWxlPSJmaWxsOnZhcigtLWMtb3V0KSI+PC9wYXRoPjwvbWFya2VyPjwvZGVmcz4KICA8dGV4dCB4PSI0MCIgeT0iOTYiIHRleHQtYW5jaG9yPSJtaWRkbGUiIGNsYXNzPSJzdi10IiBzdHlsZT0iZm9udC1zaXplOjEzcHgiPkludGVybmV0PC90ZXh0PgogIDxyZWN0IHg9IjE2MCIgeT0iMzAiIHdpZHRoPSIxOTAiIGhlaWdodD0iMTMwIiByeD0iMTAiIGNsYXNzPSJzdi1ib3giIHN0eWxlPSJzdHJva2U6dmFyKC0tYy1pbik7c3Ryb2tlLXdpZHRoOjIiPjwvcmVjdD4KICA8dGV4dCB4PSIyNTUiIHk9IjU2IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBjbGFzcz0ic3YtdCIgc3R5bGU9ImZvbnQtc2l6ZToxM3B4Ij5TdWJuZXQgTlNHPC90ZXh0PgogIDx0ZXh0IHg9IjI1NSIgeT0iNzQiIHRleHQtYW5jaG9yPSJtaWRkbGUiIGNsYXNzPSJzdi1zIj5uc2cxIG9uIFNVQk5FVDE8L3RleHQ+CiAgPHJlY3QgeD0iMzkwIiB5PSIzMCIgd2lkdGg9IjE3MCIgaGVpZ2h0PSIxMzAiIHJ4PSIxMCIgY2xhc3M9InN2LWJveCIgc3R5bGU9InN0cm9rZTp2YXIoLS1tdXRlZCkiPjwvcmVjdD4KICA8dGV4dCB4PSI0NzUiIHk9IjU2IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBjbGFzcz0ic3YtdCIgc3R5bGU9ImZvbnQtc2l6ZToxM3B4Ij5OSUMgTlNHPC90ZXh0PgogIDx0ZXh0IHg9IjQ3NSIgeT0iNzQiIHRleHQtYW5jaG9yPSJtaWRkbGUiIGNsYXNzPSJzdi1zIj4obm9uZSBpbiB0aGlzIGxhYik8L3RleHQ+CiAgPHJlY3QgeD0iNjAwIiB5PSI3MCIgd2lkdGg9IjEwMCIgaGVpZ2h0PSI1MCIgcng9IjgiIGNsYXNzPSJzdi1ib3giIHN0eWxlPSJzdHJva2U6dmFyKC0tbXV0ZWQpIj48L3JlY3Q+CiAgPHRleHQgeD0iNjUwIiB5PSIxMDAiIHRleHQtYW5jaG9yPSJtaWRkbGUiIGNsYXNzPSJzdi10IiBzdHlsZT0iZm9udC1zaXplOjEzcHgiPlZNPC90ZXh0PgogIDxwYXRoIGQ9Ik03MCAxMDAgSDU5OCIgc3R5bGU9InN0cm9rZTp2YXIoLS1jLWluKTtzdHJva2Utd2lkdGg6MjtmaWxsOm5vbmUiIG1hcmtlci1lbmQ9InVybCgjYTMpIj48L3BhdGg+CiAgPHRleHQgeD0iOTAiIHk9IjkyIiBjbGFzcz0ic3YtcyIgc3R5bGU9ImZpbGw6dmFyKC0tYy1pbikiPmluYm91bmQ6IHN1Ym5ldCDihpIgTklDPC90ZXh0PgogIDxwYXRoIGQ9Ik01OTggMTM4IEg3MCIgc3R5bGU9InN0cm9rZTp2YXIoLS1jLW91dCk7c3Ryb2tlLXdpZHRoOjI7ZmlsbDpub25lIiBtYXJrZXItZW5kPSJ1cmwoI2E0KSI+PC9wYXRoPgogIDx0ZXh0IHg9IjkwIiB5PSIxNTYiIGNsYXNzPSJzdi1zIiBzdHlsZT0iZmlsbDp2YXIoLS1jLW91dCkiPm91dGJvdW5kOiBOSUMg4oaSIHN1Ym5ldDwvdGV4dD4KICA8dGV4dCB4PSIxNjAiIHk9IjE5MCIgY2xhc3M9InN2LXMiPkJvdGggbXVzdCBhbGxvdyB0aGUgdHJhZmZpYy4gTm8gTlNHIGF0IGEgbGV2ZWwgbWVhbnMgdGhhdCBsZXZlbCBkb2VzbiYjMzk7dCBmaWx0ZXIuPC90ZXh0Pgo8L3N2Zz4=" />
</div>
<figcaption>Figure 3. Inbound traffic meets the subnet NSG first;
outbound traffic meets the NIC NSG first.</figcaption>
</figure>

- **Prefer subnet-level NSGs.** One NSG protects every VM in the subnet,
  which is easier to audit. That's what the lab does: `nsg1` on
  `SUBNET1`.
- **Use NIC-level NSGs sparingly**, for a VM that needs an exception.
  Having both levels on every VM is the most common reason "I allowed it
  but it still doesn't work".
- When you create a VM in the portal with "Basic" NIC security, Azure
  quietly creates a NIC NSG for it. If you manage security at the
  subnet, remove that NIC NSG or keep it in sync.

## 7. Service tags and application security groups

**Service tags** are Microsoft-maintained names for groups of IP ranges,
so you never hard-code Azure addresses. Useful ones:

<div class="tbl">

| Tag                              | Covers                                                                  |
|----------------------------------|-------------------------------------------------------------------------|
| `VirtualNetwork`                 | Your VNet, peered VNets and connected on-premises ranges                |
| `Internet`                       | Public IP space outside Azure's VNets                                   |
| `AzureLoadBalancer`              | The source of load balancer health probes (168.63.129.16)               |
| `Storage`, `Sql`, `AzureMonitor` | Azure service endpoints; regional variants like `Storage.WestUS3` exist |
| `AzureCloud`                     | All Azure public IP ranges                                              |

</div>

**Application security groups (ASGs)** let you write rules by *role*
instead of by IP. Put the web servers' NICs in `asg-web` and the
database NICs in `asg-db`, then write one rule: *allow asg-web → asg-db
on 1521*. When you add a new web server, you only add its NIC to
`asg-web`; the rule doesn't change.

<figure>
<div class="diagram">
<img
src="data:image/svg+xml;base64,PHN2ZyB2aWV3Ym94PSIwIDAgNzIwIDE3MCIgcm9sZT0iaW1nIiBhcmlhLWxhYmVsbGVkYnk9ImY0dCI+CiAgPHRpdGxlIGlkPSJmNHQiPkFwcGxpY2F0aW9uIHNlY3VyaXR5IGdyb3Vwczogd2ViIFZNcyBpbiBhc2ctd2ViIG1heSByZWFjaCBkYXRhYmFzZSBWTXMgaW4gYXNnLWRiIG9uIHBvcnQgMTUyMTsgbm90aGluZyBlbHNlIG1heS48L3RpdGxlPgogIDxkZWZzPjxtYXJrZXIgaWQ9ImE1IiB2aWV3Ym94PSIwIDAgMTAgMTAiIHJlZng9IjkiIHJlZnk9IjUiIG1hcmtlcndpZHRoPSI3IiBtYXJrZXJoZWlnaHQ9IjciIG9yaWVudD0iYXV0byI+PHBhdGggZD0iTTAsMCBMMTAsNSBMMCwxMCB6IiBzdHlsZT0iZmlsbDp2YXIoLS1jLWFsbG93KSI+PC9wYXRoPjwvbWFya2VyPjwvZGVmcz4KICA8cmVjdCB4PSIyMCIgeT0iMjAiIHdpZHRoPSIyNDAiIGhlaWdodD0iMTIwIiByeD0iMTIiIHN0eWxlPSJmaWxsOm5vbmU7c3Ryb2tlOnZhcigtLWMtaW4pO3N0cm9rZS13aWR0aDoyO3N0cm9rZS1kYXNoYXJyYXk6NiA0Ij48L3JlY3Q+CiAgPHRleHQgeD0iMzYiIHk9IjQ0IiBjbGFzcz0ic3YtdCIgc3R5bGU9ImZpbGw6dmFyKC0tYy1pbik7Zm9udC1zaXplOjEzcHgiPmFzZy13ZWI8L3RleHQ+CiAgPHJlY3QgeD0iNDAiIHk9IjYwIiB3aWR0aD0iOTAiIGhlaWdodD0iNDAiIHJ4PSI2IiBjbGFzcz0ic3YtYm94IiBzdHlsZT0ic3Ryb2tlOnZhcigtLW11dGVkKSI+PC9yZWN0Pjx0ZXh0IHg9Ijg1IiB5PSI4NSIgdGV4dC1hbmNob3I9Im1pZGRsZSIgY2xhc3M9InN2LW0iPndlYjAxPC90ZXh0PgogIDxyZWN0IHg9IjE1MCIgeT0iNjAiIHdpZHRoPSI5MCIgaGVpZ2h0PSI0MCIgcng9IjYiIGNsYXNzPSJzdi1ib3giIHN0eWxlPSJzdHJva2U6dmFyKC0tbXV0ZWQpIj48L3JlY3Q+PHRleHQgeD0iMTk1IiB5PSI4NSIgdGV4dC1hbmNob3I9Im1pZGRsZSIgY2xhc3M9InN2LW0iPndlYjAyPC90ZXh0PgogIDxyZWN0IHg9IjQ2MCIgeT0iMjAiIHdpZHRoPSIyNDAiIGhlaWdodD0iMTIwIiByeD0iMTIiIHN0eWxlPSJmaWxsOm5vbmU7c3Ryb2tlOnZhcigtLWMtb3V0KTtzdHJva2Utd2lkdGg6MjtzdHJva2UtZGFzaGFycmF5OjYgNCI+PC9yZWN0PgogIDx0ZXh0IHg9IjQ3NiIgeT0iNDQiIGNsYXNzPSJzdi10IiBzdHlsZT0iZmlsbDp2YXIoLS1jLW91dCk7Zm9udC1zaXplOjEzcHgiPmFzZy1kYjwvdGV4dD4KICA8cmVjdCB4PSI1MzUiIHk9IjYwIiB3aWR0aD0iOTAiIGhlaWdodD0iNDAiIHJ4PSI2IiBjbGFzcz0ic3YtYm94IiBzdHlsZT0ic3Ryb2tlOnZhcigtLW11dGVkKSI+PC9yZWN0Pjx0ZXh0IHg9IjU4MCIgeT0iODUiIHRleHQtYW5jaG9yPSJtaWRkbGUiIGNsYXNzPSJzdi1tIj5vcmFkYjAxPC90ZXh0PgogIDxwYXRoIGQ9Ik0yNjIgODAgSDQ1OCIgc3R5bGU9InN0cm9rZTp2YXIoLS1jLWFsbG93KTtzdHJva2Utd2lkdGg6Mi41O2ZpbGw6bm9uZSIgbWFya2VyLWVuZD0idXJsKCNhNSkiPjwvcGF0aD4KICA8dGV4dCB4PSIzNjAiIHk9IjcwIiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBjbGFzcz0ic3YtbSIgc3R5bGU9ImZvbnQtc2l6ZToxMXB4Ij5BbGxvdyBUQ1AgMTUyMTwvdGV4dD4KICA8dGV4dCB4PSIzNjAiIHk9IjE2MCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgY2xhc3M9InN2LXMiPk9uZSBydWxlOiBzb3VyY2UgYXNnLXdlYiwgZGVzdGluYXRpb24gYXNnLWRiLCBwb3J0IDE1MjEuIE5ldyB3ZWIgVk1zIGp1c3Qgam9pbiBhc2ctd2ViLjwvdGV4dD4KPC9zdmc+" />
</div>
<figcaption>Figure 4. ASGs group NICs by role, so rules survive scaling
and IP changes.</figcaption>
</figure>

## 8. Lab: fix SSH with an NSG

**Starting point:** `VNET-SP1` with private subnet `SUBNET1` (from the
VNet post), and an Ubuntu VM `vnet1srv01` in that subnet with a Standard
public IP and no NSG. **Goal:** create `nsg1`, attach it to SUBNET1,
allow SSH, and log in. NSGs are free.

<div class="step">

<div class="step-h">

<span class="step-n">STEP 1</span>

### Check the subnet: no NSG

</div>

Open VNET-SP1 → **Subnets** → **SUBNET1**. The address prefix is now
greyed out with *"cannot be updated because the subnet is in use"*,
since the VM's NIC lives here. Private subnet is enabled, and **Network
security group** is *None*.

<figure class="shot">
<img src="images/step1.png" loading="lazy"
alt="Edit subnet panel for SUBNET1 showing prefix locked because the subnet is in use and network security group set to None" />
<figcaption>A subnet in use can't change its range, and it has no NSG
yet.</figcaption>
</figure>

</div>

<div class="step">

<div class="step-h">

<span class="step-n">STEP 2</span>

### Check the VM's network settings

</div>

Open the VM → **Networking → Network settings**. It sits in
`VNET-SP1 / SUBNET1` with private IP `10.0.0.4` (the first usable
address, as the VNet post predicted) and a public IP. **Network security
group** shows *–*, **Effective security rules** shows *0*, and the page
says *No port rules to display*.

<figure class="shot">
<img src="images/step2.png" loading="lazy"
alt="VM network settings showing VNET-SP1/SUBNET1, private IP 10.0.0.4, no network security group and zero effective security rules" />
<figcaption>No NSG on the subnet or the NIC.</figcaption>
</figure>

</div>

<div class="step">

<div class="step-h">

<span class="step-n">STEP 3</span>

### Try SSH: it times out

</div>

Connect with your SSH client (Termius here) to `<public-ip>:22`. Address
resolution works, then *"Connection failed: connection timed out"*. The
Standard public IP drops inbound traffic that no NSG allows (section 1).

<figure class="shot">
<img src="images/step3.png" loading="lazy"
alt="SSH client log ending with connection timed out on port 22" />
<figcaption>A timeout means the packets were dropped, not
refused.</figcaption>
</figure>

</div>

<div class="step">

<div class="step-h">

<span class="step-n">STEP 4</span>

### Create the NSG

</div>

Search **Network security groups** → **+ Create**. Choose `Network-RG`,
name it `nsg1`, and pick **West US 3**, which must match the VNet's
region. Then **Review + create → Create**.

<figure class="shot">
<img src="images/step4.png" loading="lazy"
alt="Create network security group form with name nsg1 in Network-RG, West US 3" />
<figcaption>Only a name and a region are required.</figcaption>
</figure>

</div>

<div class="step">

<div class="step-h">

<span class="step-n">STEP 5</span>

### Look at the default rules

</div>

Open `nsg1`. **Custom security rules: 0 inbound, 0 outbound** and
**Associated with: 0 subnets, 0 network interfaces**. Below are the six
default rules from section 4. An NSG that isn't associated with anything
does nothing.

<figure class="shot">
<img src="images/step5.png" loading="lazy"
alt="nsg1 overview with six default rules and no associations" />
<figcaption>Three inbound and three outbound defaults.</figcaption>
</figure>

</div>

<div class="step">

<div class="step-h">

<span class="step-n">STEP 6</span>

### Associate the NSG with SUBNET1

</div>

In `nsg1` go to **Settings → Subnets** → **+ Associate**. Choose virtual
network `VNET-SP1 (Network-RG)` and subnet `SUBNET1`, then click **OK**.
You can do the same from the subnet's edit panel in step 1.

<figure class="shot">
<img src="images/step6.png" loading="lazy"
alt="Associate subnet panel with VNET-SP1 and SUBNET1 selected" />
<figcaption>One NSG can be associated with many subnets.</figcaption>
</figure>

</div>

<div class="step">

<div class="step-h">

<span class="step-n">STEP 7</span>

### Confirm the association

</div>

The Subnets list now shows `SUBNET1 · 10.0.0.0/24 · VNET-SP1`.

<figure class="shot">
<img src="images/step7.png" loading="lazy"
alt="nsg1 Subnets list showing SUBNET1 10.0.0.0/24 in VNET-SP1" />
<figcaption>nsg1 now protects every NIC in SUBNET1.</figcaption>
</figure>

</div>

<div class="step">

<div class="step-h">

<span class="step-n">STEP 8</span>

### See the NSG from the VM's side

</div>

Back on the VM's **Network settings**, a new panel reads *Network
security group nsg1 (attached to subnet: SUBNET1) · Impacts 1 subnets, 0
network interfaces*, with the three inbound default rules. SSH would
still fail now, because *DenyAllInBound* (65500) catches port 22.

<figure class="shot">
<img src="images/step8.png" loading="lazy"
alt="VM network settings now showing nsg1 attached to SUBNET1 with three default inbound rules" />
<figcaption>The defaults alone still deny internet traffic.</figcaption>
</figure>

</div>

<div class="step">

<div class="step-h">

<span class="step-n">STEP 9</span>

### Start a new inbound rule

</div>

Click **Create port rule → Inbound port rule**. Creating it from the VM
page adds it to whichever NSG applies, here the subnet's `nsg1`.

<figure class="shot">
<img src="images/step9.png" loading="lazy"
alt="Create port rule menu with Inbound port rule and Outbound port rule options" />
<figcaption>Rules can be added from the VM or from the NSG
itself.</figcaption>
</figure>

</div>

<div class="step">

<div class="step-h">

<span class="step-n">STEP 10</span>

### Allow SSH on port 22

</div>

Fill in **Source** Any, **Source port ranges** `*`, **Destination** Any,
**Service** Custom, **Destination port ranges** `22`, **Protocol** Any,
**Action** Allow, **Priority** `100`, **Name** `ssh`, then **Add**.
Section 10 shows how to tighten Source and Protocol, which you should do
for anything beyond a short lab.

<figure class="shot">
<img src="images/step10.png" loading="lazy"
alt="Add inbound security rule form allowing destination port 22 with priority 100 named ssh" />
<figcaption>One rule at priority 100 sits above all the
defaults.</figcaption>
</figure>

</div>

<div class="step">

<div class="step-h">

<span class="step-n">STEP 11</span>

### Check the rule list

</div>

Inbound port rules now show four entries, with
`100 ssh 22 Any Any Any Allow` on top. Note the **warning triangle**:
Azure flags any rule that opens a management port such as 22 or 3389 to
the whole internet.

<figure class="shot">
<img src="images/step11.png" loading="lazy"
alt="Inbound rules with ssh at priority 100 above the three default rules, with a warning icon" />
<figcaption>The custom rule is evaluated first.</figcaption>
</figure>

</div>

<div class="step">

<div class="step-h">

<span class="step-n">STEP 12</span>

### SSH again: it works

</div>

Retry the connection. The Ubuntu 26.04.1 LTS welcome banner appears.
`hostname -f` returns
`vnet1srv01.<unique-id>.phxx.internal.cloudapp.net`, the VM's name in
Azure's internal DNS (provided by 168.63.129.16), which other VMs in the
VNet can resolve.

<figure class="shot">
<img src="images/step12.png" loading="lazy"
alt="Successful SSH session showing Ubuntu 26.04.1 LTS banner and hostname -f output" />
<figcaption>One allow rule was all it took.</figcaption>
</figure>

</div>

## 9. The same lab with Azure CLI

    RG=Network-RG

    # 1. Create the NSG
    az network nsg create -g $RG -n nsg1 -l westus3

    # 2. Associate it with SUBNET1
    az network vnet subnet update -g $RG --vnet-name VNET-SP1 -n SUBNET1 \
      --network-security-group nsg1

    # 3. Allow SSH, but only from your own public IP and only over TCP
    MYIP=$(curl -s https://ifconfig.me)
    az network nsg rule create -g $RG --nsg-name nsg1 -n allow-ssh-from-home \
      --priority 100 --direction Inbound --access Allow \
      --protocol Tcp --source-address-prefixes $MYIP/32 \
      --source-port-ranges '*' --destination-port-ranges 22

    # 4. List the rules, including the defaults
    az network nsg rule list -g $RG --nsg-name nsg1 --include-default \
      --query "[].{prio:priority, name:name, dir:direction, port:destinationPortRange, src:sourceAddressPrefix, access:access}" -o table

    # 5. Effective rules on the VM's NIC (the VM must be running)
    NIC=$(az vm show -g $RG -n vnet1srv01 --query "networkProfile.networkInterfaces[0].id" -o tsv)
    az network nic list-effective-nsg --ids $NIC -o table

If you run this in Cloud Shell, `ifconfig.me` returns Cloud Shell's IP,
not your laptop's. Run step 3 from your own terminal, or look up your IP
in a browser and paste it in.

## 10. Lock it down properly

The lab rule opens SSH to the entire internet. That's fine for a few
minutes of testing, but do one of these before you leave the VM running:

<div class="lab">

### Option A: restrict the source to your IP (quick)

Edit the `ssh` rule: set **Source** to *My IP address* (the portal fills
it in) and **Protocol** to *TCP*. The warning triangle disappears. If
your home IP changes, update the rule.

    az network nsg rule update -g Network-RG --nsg-name nsg1 -n ssh \
      --source-address-prefixes <your-ip>/32 --protocol Tcp

</div>

<div class="lab">

### Option B: no public SSH at all (best)

- **Azure Bastion** gives browser-based SSH and RDP through the portal,
  so the VM needs no public IP and no inbound 22 rule. It's paid, and
  the Developer SKU is the cheapest way to try it.
- **Just-in-time VM access** (Microsoft Defender for Servers) adds a
  temporary allow rule for your IP when you request access and removes
  it afterwards.
- **Site-to-site VPN or a mesh VPN** such as Tailscale lets you SSH to
  the private IP `10.0.0.4` directly.

</div>

When you're done with the lab, either delete the `ssh` rule or stop
(deallocate) the VM so it isn't sitting on the internet with port 22
open.

## 11. Troubleshooting toolkit

<div class="tbl">

| Tool                         | Where                                                    | What it tells you                                                                                                                                                |
|------------------------------|----------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Effective security rules** | VM → Network settings, or NIC → Effective security rules | The combined result of subnet and NIC NSGs for that NIC                                                                                                          |
| **IP flow verify**           | Network Watcher                                          | Whether a given packet (IP, port, protocol, direction) is allowed or denied, and by *which rule*                                                                 |
| **Connection troubleshoot**  | VM → Troubleshoot, or Network Watcher                    | Tests an actual connection end to end and reports where it fails                                                                                                 |
| **VNet flow logs**           | Network Watcher                                          | Logs every allowed and denied flow to a storage account. Microsoft is retiring the older NSG flow logs in favour of VNet flow logs, so use those for new setups. |

</div>

**Quick checklist when a port won't open:**

1.  Is there an NSG on the subnet *and* on the NIC? Both must allow it.
2.  Does a lower-numbered Deny rule match first?
3.  Is the protocol right (TCP vs UDP)?
4.  Is the service actually listening inside the VM (`sudo ss -tlnp`)
    and is the OS firewall (`ufw`, `firewalld`, Windows Firewall) open?
5.  Timeout usually means a filter dropped it; "connection refused"
    usually means it reached the VM and nothing was listening.

## 12. Quiz: test yourself

Pick an answer to see whether it's right and why.

<div id="quiz" class="quiz">

</div>

Reset quiz

## 13. Key takeaways

- A **Standard public IP is closed by default**; with no NSG allowing
  it, inbound traffic is dropped and SSH times out.
- An **NSG** is a free, stateful layer 3–4 firewall attached to subnets
  and/or NICs in the same region.
- Rules are checked **lowest priority number first**, and the **first
  match wins**.
- The **defaults** allow traffic inside the VNet and outbound to the
  internet, and deny inbound from the internet.
- With NSGs on both subnet and NIC, traffic must pass **both**. Prefer
  subnet-level NSGs.
- Use **service tags** and **ASGs** instead of hard-coded IPs, and never
  leave SSH or RDP open to Any; use your IP, Bastion or JIT.

**Next in the series:** Azure Bastion, so we can remove the public IP
and the SSH rule altogether.

Written by Sunke Nithin while learning Azure for AZ-900. Lab performed
on an Azure free account; the subscription ID, directory name and public
IP in the screenshots are blurred.

</div>
