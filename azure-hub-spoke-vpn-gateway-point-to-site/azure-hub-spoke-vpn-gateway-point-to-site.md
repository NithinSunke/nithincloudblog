<div class="wrap">

<a href="../index.html" class="back-home">← All articles</a>

<div>

<div class="eyebrow">

Azure networking · AZ-900 / AZ-104 · Hands-on lab

</div>

# Azure Hub-and-Spoke with a VPN Gateway: Peering Two Spokes and Connecting from Windows with Point-to-Site VPN

This lab builds a small hub-and-spoke network: `VNET-HUB` with a VPN
gateway, peered to `VNET-SP1` and `VNET-SP2`. It then adds point-to-site
(P2S) VPN with certificate authentication so a Windows laptop can join
the network privately. Every step is a screenshot from the real build,
including the errors along the way: the first ping to a spoke VM timed
out, and section 6 shows how to diagnose and fix it.

<div class="meta">

Sunke NithinNovember 2026~20 min read + ~90 min lab (most of it waiting
for the gateway)

</div>

</div>

[1. What we're building](#design) [2. VPN gateway basics](#gateway) [3.
Point-to-site and certificates](#p2s) [4. Peering and gateway
transit](#transit) [5. Lab, phase by phase](#lab) [6. Reach the spoke:
troubleshooting](#verify) [7. Two more things to fix](#fixes) [8. CLI
reference](#cli) [9. Cost and clean-up](#cost) [10. Quiz](#quiz) [11.
Key takeaways](#takeaways)

## 1. What we're building

In a **hub-and-spoke** design, shared services such as VPN gateways,
firewalls and Bastion live once in a central **hub** VNet. Each workload
gets its own **spoke** VNet, peered to the hub. Spokes stay isolated
from each other, but all of them can use the hub's services. This saves
money because you pay for one gateway, not one per workload, and it
keeps security rules in one place.

<figure>
<div class="diagram">
<img
src="data:image/svg+xml;base64,PHN2ZyB2aWV3Ym94PSIwIDAgNzIwIDM2MCIgcm9sZT0iaW1nIiBhcmlhLWxhYmVsbGVkYnk9ImYxdCI+CiAgPHRpdGxlIGlkPSJmMXQiPkxhYiB0b3BvbG9neTogYSBXaW5kb3dzIGxhcHRvcCBjb25uZWN0cyBvdmVyIHBvaW50LXRvLXNpdGUgVlBOIHRvIHZwbi1nYXRld2F5IGluIEdhdGV3YXlTdWJuZXQgb2YgVk5FVC1IVUIgMTIuMC4wLjAvMTYuIFZORVQtSFVCIGlzIHBlZXJlZCB3aXRoIFZORVQtU1AxIDEwLjAuMC4wLzE2LCB3aGljaCBob3N0cyB2bmV0MXNydjAxIGF0IDEwLjAuMC40LCBhbmQgd2l0aCBWTkVULVNQMi48L3RpdGxlPgogIDxkZWZzPjxtYXJrZXIgaWQ9ImExIiB2aWV3Ym94PSIwIDAgMTAgMTAiIHJlZng9IjkiIHJlZnk9IjUiIG1hcmtlcndpZHRoPSI3IiBtYXJrZXJoZWlnaHQ9IjciIG9yaWVudD0iYXV0byI+PHBhdGggZD0iTTAsMCBMMTAsNSBMMCwxMCB6IiBzdHlsZT0iZmlsbDp2YXIoLS1jLXZwbikiPjwvcGF0aD48L21hcmtlcj48L2RlZnM+CiAgPHJlY3QgeD0iMjAiIHk9IjIwIiB3aWR0aD0iMTYwIiBoZWlnaHQ9IjgwIiByeD0iMTAiIGNsYXNzPSJzdi1ib3giIHN0eWxlPSJzdHJva2U6dmFyKC0tbXV0ZWQpIj48L3JlY3Q+CiAgPHRleHQgeD0iMTAwIiB5PSI0OCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgY2xhc3M9InN2LXQiIHN0eWxlPSJmb250LXNpemU6MTRweCI+V2luZG93cyBsYXB0b3A8L3RleHQ+CiAgPHRleHQgeD0iMTAwIiB5PSI2NiIgdGV4dC1hbmNob3I9Im1pZGRsZSIgY2xhc3M9InN2LXMiPlAyU0NoaWxkQ2VydDwvdGV4dD4KICA8dGV4dCB4PSIxMDAiIHk9Ijg0IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBjbGFzcz0ic3YtbSIgc3R5bGU9ImZvbnQtc2l6ZToxMXB4Ij5nZXRzIDE5Mi4xNjguMS54PC90ZXh0PgogIDxwYXRoIGQ9Ik0xODIgNjAgSDI3OCIgc3R5bGU9InN0cm9rZTp2YXIoLS1jLXZwbik7c3Ryb2tlLXdpZHRoOjM7ZmlsbDpub25lO3N0cm9rZS1kYXNoYXJyYXk6OCA1IiBtYXJrZXItZW5kPSJ1cmwoI2ExKSI+PC9wYXRoPgogIDx0ZXh0IHg9IjIzMCIgeT0iNTAiIHRleHQtYW5jaG9yPSJtaWRkbGUiIGNsYXNzPSJzdi1zIiBzdHlsZT0iZmlsbDp2YXIoLS1jLXZwbikiPlAyUyB0dW5uZWw8L3RleHQ+CiAgPHRleHQgeD0iMjMwIiB5PSI4MCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgY2xhc3M9InN2LXMiIHN0eWxlPSJmaWxsOnZhcigtLWMtdnBuKSI+b3ZlciBpbnRlcm5ldDwvdGV4dD4KICA8cmVjdCB4PSIyODAiIHk9IjIwIiB3aWR0aD0iNDIwIiBoZWlnaHQ9IjE0MCIgcng9IjEyIiBzdHlsZT0iZmlsbDpub25lO3N0cm9rZTp2YXIoLS1jLWh1Yik7c3Ryb2tlLXdpZHRoOjIuNSI+PC9yZWN0PgogIDx0ZXh0IHg9IjI5NiIgeT0iNDQiIGNsYXNzPSJzdi10IiBzdHlsZT0iZmlsbDp2YXIoLS1jLWh1YikiPlZORVQtSFVCPC90ZXh0PgogIDx0ZXh0IHg9IjI5NiIgeT0iNjIiIGNsYXNzPSJzdi1tIiBzdHlsZT0iZm9udC1zaXplOjExcHgiPjEyLjAuMC4wLzE2PC90ZXh0PgogIDxyZWN0IHg9IjMwMCIgeT0iNzYiIHdpZHRoPSIxOTAiIGhlaWdodD0iNzAiIHJ4PSI4IiBjbGFzcz0ic3YtYm94IiBzdHlsZT0ic3Ryb2tlOnZhcigtLWMtZ3cpO3N0cm9rZS13aWR0aDoyIj48L3JlY3Q+CiAgPHRleHQgeD0iMzk1IiB5PSI5OCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgY2xhc3M9InN2LXQiIHN0eWxlPSJmb250LXNpemU6MTNweCI+R2F0ZXdheVN1Ym5ldDwvdGV4dD4KICA8dGV4dCB4PSIzOTUiIHk9IjExNCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgY2xhc3M9InN2LW0iIHN0eWxlPSJmb250LXNpemU6MTFweCI+MTIuMC4xLjAvMjQ8L3RleHQ+CiAgPHRleHQgeD0iMzk1IiB5PSIxMzIiIHRleHQtYW5jaG9yPSJtaWRkbGUiIGNsYXNzPSJzdi1zIj52cG4tZ2F0ZXdheSAoVnBuR3cxQVopPC90ZXh0PgogIDxyZWN0IHg9IjUxMCIgeT0iNzYiIHdpZHRoPSIxNzAiIGhlaWdodD0iNzAiIHJ4PSI4IiBjbGFzcz0ic3YtYm94IiBzdHlsZT0ic3Ryb2tlOnZhcigtLWxpbmUpIj48L3JlY3Q+CiAgPHRleHQgeD0iNTk1IiB5PSIxMDIiIHRleHQtYW5jaG9yPSJtaWRkbGUiIGNsYXNzPSJzdi10IiBzdHlsZT0iZm9udC1zaXplOjEzcHgiPlNVQk5FVDE8L3RleHQ+CiAgPHRleHQgeD0iNTk1IiB5PSIxMjAiIHRleHQtYW5jaG9yPSJtaWRkbGUiIGNsYXNzPSJzdi1tIiBzdHlsZT0iZm9udC1zaXplOjExcHgiPjEyLjAuMC4wLzI0IMK3IG5zZzE8L3RleHQ+CiAgPHJlY3QgeD0iODAiIHk9IjIzMCIgd2lkdGg9IjI3MCIgaGVpZ2h0PSIxMTAiIHJ4PSIxMiIgc3R5bGU9ImZpbGw6bm9uZTtzdHJva2U6dmFyKC0tYy1zcCk7c3Ryb2tlLXdpZHRoOjIuNSI+PC9yZWN0PgogIDx0ZXh0IHg9Ijk2IiB5PSIyNTQiIGNsYXNzPSJzdi10IiBzdHlsZT0iZmlsbDp2YXIoLS1jLXNwKSI+Vk5FVC1TUDE8L3RleHQ+CiAgPHRleHQgeD0iOTYiIHk9IjI3MiIgY2xhc3M9InN2LW0iIHN0eWxlPSJmb250LXNpemU6MTFweCI+MTAuMC4wLjAvMTYgwrcgU1VCTkVUMSAxMC4wLjAuMC8yNDwvdGV4dD4KICA8cmVjdCB4PSI5NiIgeT0iMjg0IiB3aWR0aD0iMTUwIiBoZWlnaHQ9IjQwIiByeD0iNiIgY2xhc3M9InN2LWJveCIgc3R5bGU9InN0cm9rZTp2YXIoLS1tdXRlZCkiPjwvcmVjdD4KICA8dGV4dCB4PSIxNzEiIHk9IjMwOSIgdGV4dC1hbmNob3I9Im1pZGRsZSIgY2xhc3M9InN2LW0iPnZuZXQxc3J2MDEgLjQ8L3RleHQ+CiAgPHJlY3QgeD0iNDIwIiB5PSIyMzAiIHdpZHRoPSIyNzAiIGhlaWdodD0iMTEwIiByeD0iMTIiIHN0eWxlPSJmaWxsOm5vbmU7c3Ryb2tlOnZhcigtLWMtc3ApO3N0cm9rZS13aWR0aDoyLjUiPjwvcmVjdD4KICA8dGV4dCB4PSI0MzYiIHk9IjI1NCIgY2xhc3M9InN2LXQiIHN0eWxlPSJmaWxsOnZhcigtLWMtc3ApIj5WTkVULVNQMjwvdGV4dD4KICA8dGV4dCB4PSI0MzYiIHk9IjI3MiIgY2xhc3M9InN2LXMiPnNlY29uZCBzcG9rZTwvdGV4dD4KICA8cGF0aCBkPSJNNDAwIDE2MiBMMjUwIDIyOCIgc3R5bGU9InN0cm9rZTp2YXIoLS1hY2NlbnQpO3N0cm9rZS13aWR0aDoyLjU7ZmlsbDpub25lIj48L3BhdGg+CiAgPHBhdGggZD0iTTU2MCAxNjIgTDU2MCAyMjgiIHN0eWxlPSJzdHJva2U6dmFyKC0tYWNjZW50KTtzdHJva2Utd2lkdGg6Mi41O2ZpbGw6bm9uZSI+PC9wYXRoPgogIDx0ZXh0IHg9IjE5MCIgeT0iMTk2IiBjbGFzcz0ic3YtcyIgc3R5bGU9ImZpbGw6dmFyKC0tYWNjZW50KSI+U1AxLUhVQiBwZWVyaW5nPC90ZXh0PgogIDx0ZXh0IHg9IjU3MCIgeT0iMjAwIiBjbGFzcz0ic3YtcyIgc3R5bGU9ImZpbGw6dmFyKC0tYWNjZW50KSI+U1AyLUhVQiBwZWVyaW5nPC90ZXh0PgogIDxwYXRoIGQ9Ik0zNTIgMzAwIEg0MTgiIHN0eWxlPSJzdHJva2U6dmFyKC0tYy1iYWQpO3N0cm9rZS13aWR0aDoxLjU7ZmlsbDpub25lO3N0cm9rZS1kYXNoYXJyYXk6NSA1Ij48L3BhdGg+CiAgPHRleHQgeD0iMzg1IiB5PSIyOTIiIHRleHQtYW5jaG9yPSJtaWRkbGUiIGNsYXNzPSJzdi1zIiBzdHlsZT0iZmlsbDp2YXIoLS1jLWJhZCkiPm5vIHBhdGg8L3RleHQ+Cjwvc3ZnPg==" />
</div>
<figcaption>Figure 1. The lab network. All three VNets are in
Network-RG, West US 3.</figcaption>
</figure>

## 2. VPN gateway basics

An **Azure VPN gateway** (resource type *virtual network gateway*,
gateway type *VPN*) is a pair of managed VMs that Azure runs in a
special subnet of your VNet. It terminates encrypted tunnels over the
public internet. One gateway can serve three kinds of connection:

<div class="tbl">

| Connection              | Connects                                          | Typical use                                                  |
|-------------------------|---------------------------------------------------|--------------------------------------------------------------|
| **Site-to-site (S2S)**  | An on-premises VPN device ↔ Azure, over IPsec/IKE | Office or datacenter network to Azure                        |
| **Point-to-site (P2S)** | Individual computers ↔ Azure                      | Admins and remote workers; this lab                          |
| **VNet-to-VNet**        | Two Azure VPN gateways                            | Encrypted links between regions (peering is usually simpler) |

</div>

- **GatewaySubnet:** the gateway must live in a subnet named exactly
  `GatewaySubnet`, with nothing else in it. Microsoft recommends `/27`
  or larger; this lab used a `/24`. The portal's *Subnet purpose →
  Virtual Network Gateway* option fills in the name for you.
- **SKU:** sets throughput, tunnel counts and features. SKUs ending in
  **AZ**, such as `VpnGw1AZ`, are **zone-redundant**: the gateway
  instances are spread across availability zones. The Basic SKU is
  legacy, lacks most P2S options and isn't recommended.
- **VPN type:** **Route-based** is the default and the one you want; P2S
  needs it.
- **Public IP:** a Standard, static public IP is created with the
  gateway. Clients and on-premises devices connect to it.
- **Active-active** and **BGP** are optional. They matter for resilient
  site-to-site designs, and the lab leaves both disabled.
- **It takes time:** creating a gateway typically takes 20–45 minutes.

<div class="note">

**Coming from OCI?** An Azure VPN gateway does the job of OCI's DRG plus
its IPSec connection. A hub VNet with gateway transit is like a DRG hub
with spoke VCN attachments. Azure point-to-site has no direct OCI
equivalent; in OCI you'd usually run your own VPN server, such as
OpenVPN Access Server, from the Marketplace.

</div>

## 3. Point-to-site and certificates

Point-to-site lets a single computer dial into the VNet. Three settings
define it:

<div class="tbl">

| Setting            | Options                                                                                                                                                                           | Lab choice                              |
|--------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------|
| **Address pool**   | A private range that doesn't overlap any VNet *or* the client's own network. Each connected client gets one address from it.                                                      | `192.168.1.0/24` (see section 7, fix 1) |
| **Tunnel type**    | **OpenVPN** (TLS on 443, works through most firewalls; needs the Azure VPN Client or an OpenVPN client), **IKEv2** (native in Windows and macOS), **SSTP** (Windows only, legacy) | IKEv2 and OpenVPN                       |
| **Authentication** | **Azure certificate**, **Microsoft Entra ID** (OpenVPN only; supports MFA and Conditional Access), **RADIUS**                                                                     | Azure certificate                       |

</div>

With **certificate authentication**, you upload the *public* part of a
**root certificate** to the gateway. Any client presenting a **child
(client) certificate** signed by that root, together with its private
key, is allowed in. To cut off one device, add its child certificate's
thumbprint to the **revoked certificates** list.

<figure>
<div class="diagram">
<img
src="data:image/svg+xml;base64,PHN2ZyB2aWV3Ym94PSIwIDAgNzIwIDIyMCIgcm9sZT0iaW1nIiBhcmlhLWxhYmVsbGVkYnk9ImYydCI+CiAgPHRpdGxlIGlkPSJmMnQiPkNlcnRpZmljYXRlIGNoYWluOiBQMlNSb290Q2VydCBzaWducyBQMlNDaGlsZENlcnQuIFRoZSByb290JiMzOTtzIHB1YmxpYyBrZXkgKEJhc2UtNjQgLmNlcikgaXMgdXBsb2FkZWQgdG8gdGhlIGdhdGV3YXk7IHRoZSBjaGlsZCBjZXJ0aWZpY2F0ZSB3aXRoIGl0cyBwcml2YXRlIGtleSAoLnBmeCkgaXMgaW5zdGFsbGVkIG9uIGVhY2ggY2xpZW50LjwvdGl0bGU+CiAgPGRlZnM+PG1hcmtlciBpZD0iYTIiIHZpZXdib3g9IjAgMCAxMCAxMCIgcmVmeD0iOSIgcmVmeT0iNSIgbWFya2Vyd2lkdGg9IjciIG1hcmtlcmhlaWdodD0iNyIgb3JpZW50PSJhdXRvIj48cGF0aCBkPSJNMCwwIEwxMCw1IEwwLDEwIHoiIHN0eWxlPSJmaWxsOnZhcigtLW11dGVkKSI+PC9wYXRoPjwvbWFya2VyPjwvZGVmcz4KICA8cmVjdCB4PSIyMCIgeT0iMjAiIHdpZHRoPSIyMDAiIGhlaWdodD0iNzAiIHJ4PSIxMCIgY2xhc3M9InN2LWJveCIgc3R5bGU9InN0cm9rZTp2YXIoLS1jLWh1Yik7c3Ryb2tlLXdpZHRoOjIiPjwvcmVjdD4KICA8dGV4dCB4PSIxMjAiIHk9IjQ2IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBjbGFzcz0ic3YtdCIgc3R5bGU9ImZvbnQtc2l6ZToxNHB4Ij5QMlNSb290Q2VydDwvdGV4dD4KICA8dGV4dCB4PSIxMjAiIHk9IjY2IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBjbGFzcz0ic3YtcyI+c2VsZi1zaWduZWQgcm9vdCAoQ2VydFNpZ24pPC90ZXh0PgogIDx0ZXh0IHg9IjEyMCIgeT0iODIiIHRleHQtYW5jaG9yPSJtaWRkbGUiIGNsYXNzPSJzdi1zIj5rZWVwcyB0aGUgcHJpdmF0ZSBrZXkgc2FmZTwvdGV4dD4KICA8cGF0aCBkPSJNMTIwIDkyIFYxMjgiIGNsYXNzPSJzdi1saW5lIiBtYXJrZXItZW5kPSJ1cmwoI2EyKSI+PC9wYXRoPgogIDx0ZXh0IHg9IjEyOCIgeT0iMTE0IiBjbGFzcz0ic3YtcyI+c2lnbnM8L3RleHQ+CiAgPHJlY3QgeD0iMjAiIHk9IjEzMCIgd2lkdGg9IjIwMCIgaGVpZ2h0PSI3MCIgcng9IjEwIiBjbGFzcz0ic3YtYm94IiBzdHlsZT0ic3Ryb2tlOnZhcigtLWMtdnBuKTtzdHJva2Utd2lkdGg6MiI+PC9yZWN0PgogIDx0ZXh0IHg9IjEyMCIgeT0iMTU2IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBjbGFzcz0ic3YtdCIgc3R5bGU9ImZvbnQtc2l6ZToxNHB4Ij5QMlNDaGlsZENlcnQ8L3RleHQ+CiAgPHRleHQgeD0iMTIwIiB5PSIxNzYiIHRleHQtYW5jaG9yPSJtaWRkbGUiIGNsYXNzPSJzdi1zIj5jbGllbnQgYXV0aCBFS1U8L3RleHQ+CiAgPHRleHQgeD0iMTIwIiB5PSIxOTIiIHRleHQtYW5jaG9yPSJtaWRkbGUiIGNsYXNzPSJzdi1tIiBzdHlsZT0iZm9udC1zaXplOjExcHgiPjEuMy42LjEuNS41LjcuMy4yPC90ZXh0PgogIDxwYXRoIGQ9Ik0yMjIgNTUgSDQxOCIgY2xhc3M9InN2LWxpbmUiIG1hcmtlci1lbmQ9InVybCgjYTIpIj48L3BhdGg+CiAgPHRleHQgeD0iMzIwIiB5PSI0NiIgdGV4dC1hbmNob3I9Im1pZGRsZSIgY2xhc3M9InN2LXMiPmV4cG9ydCBwdWJsaWMga2V5IG9ubHk8L3RleHQ+CiAgPHRleHQgeD0iMzIwIiB5PSI3MiIgdGV4dC1hbmNob3I9Im1pZGRsZSIgY2xhc3M9InN2LW0iIHN0eWxlPSJmb250LXNpemU6MTFweCI+cm9vdGNlcnQuY2VyIChCYXNlLTY0KTwvdGV4dD4KICA8cmVjdCB4PSI0MjAiIHk9IjIwIiB3aWR0aD0iMjgwIiBoZWlnaHQ9IjcwIiByeD0iMTAiIGNsYXNzPSJzdi1ib3giIHN0eWxlPSJzdHJva2U6dmFyKC0tYy1ndyk7c3Ryb2tlLXdpZHRoOjIiPjwvcmVjdD4KICA8dGV4dCB4PSI1NjAiIHk9IjQ2IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBjbGFzcz0ic3YtdCIgc3R5bGU9ImZvbnQtc2l6ZToxNHB4Ij52cG4tZ2F0ZXdheTwvdGV4dD4KICA8dGV4dCB4PSI1NjAiIHk9IjY2IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBjbGFzcz0ic3YtcyI+Um9vdCBjZXJ0aWZpY2F0ZXMg4oaSIFB1YmxpYyBjZXJ0aWZpY2F0ZSBkYXRhPC90ZXh0PgogIDx0ZXh0IHg9IjU2MCIgeT0iODIiIHRleHQtYW5jaG9yPSJtaWRkbGUiIGNsYXNzPSJzdi1zIj5zdGF0dXM6IFZhbGlkPC90ZXh0PgogIDxwYXRoIGQ9Ik0yMjIgMTY1IEg0MTgiIGNsYXNzPSJzdi1saW5lIiBtYXJrZXItZW5kPSJ1cmwoI2EyKSI+PC9wYXRoPgogIDx0ZXh0IHg9IjMyMCIgeT0iMTU2IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBjbGFzcz0ic3YtcyI+ZXhwb3J0IHdpdGggcHJpdmF0ZSBrZXk8L3RleHQ+CiAgPHRleHQgeD0iMzIwIiB5PSIxODIiIHRleHQtYW5jaG9yPSJtaWRkbGUiIGNsYXNzPSJzdi1tIiBzdHlsZT0iZm9udC1zaXplOjExcHgiPmNoaWxkY2VydC5wZnggKHBhc3N3b3JkKTwvdGV4dD4KICA8cmVjdCB4PSI0MjAiIHk9IjEzMCIgd2lkdGg9IjI4MCIgaGVpZ2h0PSI3MCIgcng9IjEwIiBjbGFzcz0ic3YtYm94IiBzdHlsZT0ic3Ryb2tlOnZhcigtLW11dGVkKTtzdHJva2Utd2lkdGg6MiI+PC9yZWN0PgogIDx0ZXh0IHg9IjU2MCIgeT0iMTU2IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBjbGFzcz0ic3YtdCIgc3R5bGU9ImZvbnQtc2l6ZToxNHB4Ij5FYWNoIGNsaWVudCBjb21wdXRlcjwvdGV4dD4KICA8dGV4dCB4PSI1NjAiIHk9IjE3NiIgdGV4dC1hbmNob3I9Im1pZGRsZSIgY2xhc3M9InN2LXMiPkN1cnJlbnQgVXNlciDihpIgUGVyc29uYWwgc3RvcmU8L3RleHQ+CiAgPHRleHQgeD0iNTYwIiB5PSIxOTIiIHRleHQtYW5jaG9yPSJtaWRkbGUiIGNsYXNzPSJzdi1zIj5wcmVzZW50cyB0aGUgY2hpbGQgY2VydCBvbiBjb25uZWN0PC90ZXh0Pgo8L3N2Zz4=" />
</div>
<figcaption>Figure 2. The gateway trusts the root; clients prove
themselves with a child certificate signed by it.</figcaption>
</figure>

## 4. Peering and gateway transit

Peering connects two VNets, but it doesn't automatically let a spoke use
the hub's VPN gateway. That takes **gateway transit**, which is two
checkboxes on the peering:

- On the **hub side**: *Allow gateway or route server in 'VNET-HUB' to
  forward traffic to 'VNET-SPx'* (this is called `allowGatewayTransit`).
- On the **spoke side**: *Enable 'VNET-SPx' to use 'VNET-HUB's' remote
  gateway or route server* (`useRemoteGateways`).

<figure>
<div class="diagram">
<img
src="data:image/svg+xml;base64,PHN2ZyB2aWV3Ym94PSIwIDAgODAwIDIwMCIgcm9sZT0iaW1nIiBhcmlhLWxhYmVsbGVkYnk9ImYzdCI+CiAgPHRpdGxlIGlkPSJmM3QiPldpdGhvdXQgZ2F0ZXdheSB0cmFuc2l0LCBQMlMgY2xpZW50cyByZWFjaCBvbmx5IFZORVQtSFVCLiBXaXRoIGdhdGV3YXkgdHJhbnNpdCBlbmFibGVkIG9uIGJvdGggc2lkZXMsIHRoZSBnYXRld2F5IGFkdmVydGlzZXMgVk5FVC1TUDEmIzM5O3MgcmFuZ2UgdG8gY2xpZW50cyBhbmQgU1AxIHJvdXRlcyByZXR1cm4gdHJhZmZpYyBiYWNrIHRocm91Z2ggdGhlIGdhdGV3YXkuPC90aXRsZT4KICA8dGV4dCB4PSIyMCIgeT0iMjgiIGNsYXNzPSJzdi10IiBzdHlsZT0iZm9udC1zaXplOjE0cHgiPldpdGhvdXQgdHJhbnNpdCAoZmlyc3QgYXR0ZW1wdCk8L3RleHQ+CiAgPGcgY2xhc3M9InN2LW0iIHN0eWxlPSJmb250LXNpemU6MTJweCI+CiAgICA8cmVjdCB4PSIyMCIgeT0iNDAiIHdpZHRoPSIxMTAiIGhlaWdodD0iNDAiIHJ4PSI2IiBjbGFzcz0ic3YtYm94IiBzdHlsZT0ic3Ryb2tlOnZhcigtLW11dGVkKSI+PC9yZWN0Pjx0ZXh0IHg9Ijc1IiB5PSI2NSIgdGV4dC1hbmNob3I9Im1pZGRsZSI+TGFwdG9wPC90ZXh0PgogICAgPHJlY3QgeD0iMTgwIiB5PSI0MCIgd2lkdGg9IjEzMCIgaGVpZ2h0PSI0MCIgcng9IjYiIGNsYXNzPSJzdi1ib3giIHN0eWxlPSJzdHJva2U6dmFyKC0tYy1odWIpIj48L3JlY3Q+PHRleHQgeD0iMjQ1IiB5PSI2NSIgdGV4dC1hbmNob3I9Im1pZGRsZSI+Vk5FVC1IVUIg4pyTPC90ZXh0PgogICAgPHJlY3QgeD0iMzYwIiB5PSI0MCIgd2lkdGg9IjEzMCIgaGVpZ2h0PSI0MCIgcng9IjYiIGNsYXNzPSJzdi1ib3giIHN0eWxlPSJzdHJva2U6dmFyKC0tYy1zcCkiPjwvcmVjdD48dGV4dCB4PSI0MjUiIHk9IjY1IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIj5WTkVULVNQMSDinJU8L3RleHQ+CiAgPC9nPgogIDxwYXRoIGQ9Ik0xMzIgNjAgSDE3OCIgc3R5bGU9InN0cm9rZTp2YXIoLS1jLXZwbik7c3Ryb2tlLXdpZHRoOjI7ZmlsbDpub25lIj48L3BhdGg+CiAgPHBhdGggZD0iTTMxMiA2MCBIMzU4IiBzdHlsZT0ic3Ryb2tlOnZhcigtLWMtYmFkKTtzdHJva2Utd2lkdGg6MjtmaWxsOm5vbmU7c3Ryb2tlLWRhc2hhcnJheTo1IDQiPjwvcGF0aD4KICA8dGV4dCB4PSI1MTAiIHk9IjU2IiBjbGFzcz0ic3YtcyI+Z2F0ZXdheSBkb2VzbiYjMzk7dCBhZHZlcnRpc2UgMTAuMC4wLjAvMTY7PC90ZXh0PgogIDx0ZXh0IHg9IjUxMCIgeT0iNzIiIGNsYXNzPSJzdi1zIj5TUDEgaGFzIG5vIHJvdXRlIGJhY2sgdG8gMTkyLjE2OC4xLjAvMjQ8L3RleHQ+CiAgPHRleHQgeD0iMjAiIHk9IjEyOCIgY2xhc3M9InN2LXQiIHN0eWxlPSJmb250LXNpemU6MTRweCI+V2l0aCB0cmFuc2l0IChzZWN0aW9uIDYpPC90ZXh0PgogIDxnIGNsYXNzPSJzdi1tIiBzdHlsZT0iZm9udC1zaXplOjEycHgiPgogICAgPHJlY3QgeD0iMjAiIHk9IjE0MCIgd2lkdGg9IjExMCIgaGVpZ2h0PSI0MCIgcng9IjYiIGNsYXNzPSJzdi1ib3giIHN0eWxlPSJzdHJva2U6dmFyKC0tbXV0ZWQpIj48L3JlY3Q+PHRleHQgeD0iNzUiIHk9IjE2NSIgdGV4dC1hbmNob3I9Im1pZGRsZSI+TGFwdG9wPC90ZXh0PgogICAgPHJlY3QgeD0iMTgwIiB5PSIxNDAiIHdpZHRoPSIxMzAiIGhlaWdodD0iNDAiIHJ4PSI2IiBjbGFzcz0ic3YtYm94IiBzdHlsZT0ic3Ryb2tlOnZhcigtLWMtaHViKSI+PC9yZWN0Pjx0ZXh0IHg9IjI0NSIgeT0iMTY1IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIj5WTkVULUhVQiDinJM8L3RleHQ+CiAgICA8cmVjdCB4PSIzNjAiIHk9IjE0MCIgd2lkdGg9IjEzMCIgaGVpZ2h0PSI0MCIgcng9IjYiIGNsYXNzPSJzdi1ib3giIHN0eWxlPSJzdHJva2U6dmFyKC0tYy1zcCkiPjwvcmVjdD48dGV4dCB4PSI0MjUiIHk9IjE2NSIgdGV4dC1hbmNob3I9Im1pZGRsZSI+Vk5FVC1TUDEg4pyTPC90ZXh0PgogIDwvZz4KICA8cGF0aCBkPSJNMTMyIDE2MCBIMTc4IiBzdHlsZT0ic3Ryb2tlOnZhcigtLWMtdnBuKTtzdHJva2Utd2lkdGg6MjtmaWxsOm5vbmUiPjwvcGF0aD4KICA8cGF0aCBkPSJNMzEyIDE2MCBIMzU4IiBzdHlsZT0ic3Ryb2tlOnZhcigtLWdvb2QpO3N0cm9rZS13aWR0aDoyO2ZpbGw6bm9uZSI+PC9wYXRoPgogIDx0ZXh0IHg9IjUxMCIgeT0iMTU2IiBjbGFzcz0ic3YtcyI+aHViOiBhbGxvd0dhdGV3YXlUcmFuc2l0PC90ZXh0PgogIDx0ZXh0IHg9IjUxMCIgeT0iMTcyIiBjbGFzcz0ic3YtcyI+c3Bva2U6IHVzZVJlbW90ZUdhdGV3YXlzLCB0aGVuIHJlLWRvd25sb2FkPC90ZXh0PgogIDx0ZXh0IHg9IjUxMCIgeT0iMTg4IiBjbGFzcz0ic3YtcyI+dGhlIFZQTiBjbGllbnQgcGFja2FnZTwvdGV4dD4KPC9zdmc+" />
</div>
<figcaption>Figure 3. Gateway transit is what lets spokes share the
hub's VPN gateway.</figcaption>
</figure>

Even with transit, **SP1 still can't reach SP2**, because peering
doesn't pass traffic on to a third VNet. Spoke-to-spoke traffic needs a
router in the hub: Azure Firewall or a network virtual appliance, plus
route tables in the spokes. Alternatively, peer SP1 and SP2 directly.

## 5. Lab, phase by phase

**Starting point:** `VNET-SP1` with `SUBNET1`, `nsg1` and VM
`vnet1srv01`, built in the
[VNet](../azure-vnet-private-subnet-lab/azure-vnet-private-subnet-lab.html)
and
[NSG](../azure-network-security-groups-nsg/azure-network-security-groups-nsg.html)
posts. You also need a Windows 10/11 or Windows Server PC for the
certificates and the VPN client.

<div class="phase">

Phase 1 · The three VNets

</div>

### Create the hub and the second spoke

Create `VNET-HUB` (`12.0.0.0/16`, see section 7, fix 2) and `VNET-SP2`
in `Network-RG`, West US 3, the same way as VNET-SP1 in the VNet post.
Peering also works across regions, but keeping everything in one region
keeps the lab simple.

<figure class="shot">
<img src="images/step01.png" loading="lazy"
alt="Virtual networks list showing VNET-HUB, VNET-SP1 and VNET-SP2 in Network-RG, West US 3" />
<figcaption>Three VNets, one resource group.</figcaption>
</figure>

<div class="pair">

<figure class="shot">
<img src="images/step02.png" loading="lazy"
alt="VNET-HUB overview with address space 12.0.0.0/16 and one subnet" />
<figcaption>VNET-HUB: 12.0.0.0/16, Azure-provided DNS.</figcaption>
</figure>

<figure class="shot">
<img src="images/step03.png" loading="lazy"
alt="VNET-HUB subnets: SUBNET1 12.0.0.0/24 with 251 available IPs and nsg1" />
<figcaption>Hub SUBNET1 has 251 free IPs: a /24 minus 5
reserved.</figcaption>
</figure>

</div>

<div class="pair">

<figure class="shot">
<img src="images/step04.png" loading="lazy"
alt="VNET-SP1 overview, Capabilities tab showing DDoS, Firewall, Peerings, Defender and Private endpoints not configured" />
<figcaption>SP1's Capabilities tab: peerings not configured
yet.</figcaption>
</figure>

<figure class="shot">
<img src="images/step05.png" loading="lazy"
alt="VNET-SP1 subnets: SUBNET1 10.0.0.0/24 with 250 available IPs and nsg1" />
<figcaption>SP1 SUBNET1 shows 250 free IPs because the VM's NIC uses
one.</figcaption>
</figure>

</div>

<div class="phase">

Phase 2 · GatewaySubnet and the VPN gateway

</div>

### Add GatewaySubnet to the hub

VNET-HUB → **Subnets → + Subnet**. Set **Subnet purpose** to *Virtual
Network Gateway*, and the name becomes `GatewaySubnet` automatically.
Use starting address `12.0.1.0`, size `/24` (a `/27` is enough), and
leave the NSG and route table at None. Don't attach an NSG to
GatewaySubnet; it can break the gateway.

<figure class="shot">
<img src="images/step06.png" loading="lazy"
alt="Add a subnet panel with purpose Virtual Network Gateway, name GatewaySubnet, range 12.0.1.0/24" />
<figcaption>The purpose dropdown sets the required name.</figcaption>
</figure>

### Create the virtual network gateway

Search **VPN gateways → + Create**:

- **Name:** `vpn-gateway`, **Region:** West US 3
- **Gateway type:** VPN, **SKU:** `VpnGw1AZ`, **Virtual network:**
  `VNET-HUB`. The resource group and GatewaySubnet are picked up
  automatically.
- **Public IP:** create new, `vpn-ip` (Standard, Static)
- **Active-active:** Disabled, **BGP:** Disabled

Then **Review + create**, and wait 20–45 minutes.

<figure class="shot">
<img src="images/step07.png" loading="lazy"
alt="Create virtual network gateway form with name vpn-gateway, SKU VpnGw1AZ, VNET-HUB, GatewaySubnet 12.0.1.0/24 and new public IP vpn-ip" />
<figcaption>Only VNets in the selected subscription and region are
listed.</figcaption>
</figure>

<figure class="shot">
<img src="images/step08.png" loading="lazy"
alt="vpn-gateway overview: SKU VpnGw1AZ, gateway type VPN, VPN type Route-based, virtual network VNET-HUB" />
<figcaption>Deployed: route-based VPN on VNET-HUB with its public
IP.</figcaption>
</figure>

<div class="phase">

Phase 3 · Peer the spokes to the hub

</div>

### Add the SP1 peering

VNET-HUB → **Peerings → + Add**. One form creates **both directions**:
the "remote" link (from SP1's side) and the "local" link (from the hub's
side). The lab names were `hub-sp1` for the remote link and `SP1-HUB`
for the local one. *Allow access* and *Allow forwarded traffic* are
ticked on both sides. The two gateway checkboxes were left unticked
here. That's why the first ping to the spoke VM failed, and section 6
walks through the fix. If you're building this fresh, tick them now as
described there.

<div class="pair">

<figure class="shot">
<img src="images/step09.png" loading="lazy"
alt="VNET-HUB peerings list, empty" />
<figcaption>No peerings yet.</figcaption>
</figure>

<figure class="shot">
<img src="images/step10.png" loading="lazy"
alt="Add peering form for VNET-SP1 with access and forwarded traffic allowed, gateway options unticked" />
<figcaption>One form, two peering links.</figcaption>
</figure>

</div>

<figure class="shot">
<img src="images/step11.png" loading="lazy"
alt="Peering SP1-HUB fully synchronized and connected" />
<figcaption><strong>Fully Synchronized</strong> and
<strong>Connected</strong> mean both sides agree.</figcaption>
</figure>

### Add the SP2 peering

Repeat for `VNET-SP2` with the local link name `SP2-HUB`. The address
ranges of all three VNets must not overlap, or Azure rejects the
peering.

<div class="pair">

<figure class="shot">
<img src="images/step12.png" loading="lazy"
alt="Add peering form for VNET-SP2, local link SP2-HUB" />
<figcaption>Same settings for the second spoke.</figcaption>
</figure>

<figure class="shot">
<img src="images/step13.png" loading="lazy"
alt="Two peerings SP1-HUB and SP2-HUB, both connected" />
<figcaption>Both spokes peered to the hub.</figcaption>
</figure>

</div>

<div class="phase">

Phase 4 · Certificates

</div>

### Start point-to-site, and see why certificates come first

vpn-gateway → **Point-to-site configuration → Configure now**. Enter the
address pool and choose *Azure certificate*. The page refuses to save:
*"At least one valid root certificate must be configured if Azure
Certificate Authentication type is selected."* So, on to the
certificates.

<div class="pair">

<figure class="shot">
<img src="images/step14.png" loading="lazy"
alt="Point-to-site is not configured, Configure now link" />
<figcaption>P2S is off by default.</figcaption>
</figure>

<figure class="shot">
<img src="images/step15.png" loading="lazy"
alt="Point-to-site form with address pool 192.168.1.0/24 and error requiring a root certificate" />
<figcaption>Certificate auth needs a root certificate before
saving.</figcaption>
</figure>

</div>

### Create the root and child certificates in PowerShell

On the Windows PC, open **PowerShell as administrator** and run the two
commands from Microsoft's P2S certificate guide. The first creates a
self-signed root that can sign other certificates; the second creates a
client certificate signed by it, with the Client Authentication EKU
(`1.3.6.1.5.5.7.3.2`). Both land in *Current User → Personal*.

    # Root certificate
    $params = @{
        Type = 'Custom'
        Subject = 'CN=P2SRootCert'
        KeySpec = 'Signature'
        KeyExportPolicy = 'Exportable'
        KeyUsage = 'CertSign'
        KeyUsageProperty = 'Sign'
        KeyLength = 2048
        HashAlgorithm = 'sha256'
        NotAfter = (Get-Date).AddMonths(24)
        CertStoreLocation = 'Cert:\CurrentUser\My'
    }
    $cert = New-SelfSignedCertificate @params

    # Client (child) certificate, signed by the root
    $params = @{
        Type = 'Custom'
        Subject = 'CN=P2SChildCert'
        DnsName = 'P2SChildCert'
        KeySpec = 'Signature'
        KeyExportPolicy = 'Exportable'
        KeyLength = 2048
        HashAlgorithm = 'sha256'
        NotAfter = (Get-Date).AddMonths(18)
        CertStoreLocation = 'Cert:\CurrentUser\My'
        Signer = $cert
        TextExtension = @('2.5.29.37={text}1.3.6.1.5.5.7.3.2')
    }
    New-SelfSignedCertificate @params

<div class="pair">

<figure class="shot">
<img src="images/step16.png" loading="lazy"
alt="PowerShell creating P2SRootCert and P2SChildCert" />
<figcaption>Root, then child signed by <code>$cert</code>.</figcaption>
</figure>

<figure class="shot">
<img src="images/step17.png" loading="lazy"
alt="certmgr showing P2SChildCert (Client Authentication) and P2SRootCert in Personal certificates" />
<figcaption><code>certmgr.msc</code>: child issued by root, expiring 18
and 24 months out.</figcaption>
</figure>

</div>

**If you closed PowerShell** before creating the child certificate,
`$cert` is gone. Look up the root's thumbprint and load it again:

    Get-ChildItem -Path "Cert:\CurrentUser\My"
    $cert = Get-ChildItem -Path "Cert:\CurrentUser\My\<root-thumbprint>"

<figure class="shot">
<img src="images/step18.png" loading="lazy"
alt="Get-ChildItem listing thumbprints of P2SChildCert and P2SRootCert" />
<figcaption>Find the root by its subject, then reuse its
thumbprint.</figcaption>
</figure>

### Export the root (public key) and the child (with private key)

In `certmgr.msc`, right-click a certificate → **All Tasks → Export**:

- **P2SRootCert:** *No, do not export the private key* → **Base-64
  encoded X.509 (.CER)** → `rootcert.cer`. This file goes to Azure.
- **P2SChildCert:** *Yes, export the private key* → **PFX**, include all
  certificates in the path, set a password → `childcert.pfx`. You only
  need this to install the client certificate on *other* computers; keep
  it private.

<div class="pair">

<figure class="shot">
<img src="images/step19.png" loading="lazy"
alt="Export file format: Base-64 encoded X.509 (.CER) selected" />
<figcaption>Root: Base-64 .cer, no private key.</figcaption>
</figure>

<figure class="shot">
<img src="images/step20.png" loading="lazy"
alt="Export file name rootcert.cer in Documents" />
<figcaption>Saved as rootcert.cer.</figcaption>
</figure>

</div>

<div class="pair">

<figure class="shot">
<img src="images/step21.png" loading="lazy"
alt="Export security page with password set for the PFX" />
<figcaption>Child: protect the .pfx with a password.</figcaption>
</figure>

<figure class="shot">
<img src="images/step22.png" loading="lazy"
alt="Completing export: childcert.pfx with keys and certification path" />
<figcaption>childcert.pfx with private key and chain.</figcaption>
</figure>

</div>

<div class="phase">

Phase 5 · Configure point-to-site

</div>

### Upload the root certificate

Open `rootcert.cer` in Notepad and copy the text **between**
`-----BEGIN CERTIFICATE-----` and `-----END CERTIFICATE-----`, without
those two lines. Back on Point-to-site configuration, add a root
certificate (the lab named it `azure`; `P2SRootCert` would be clearer),
paste the data, set the tunnel type to **IKEv2 and OpenVPN (SSL)**, and
click **Save**. The gateway shows *Updating…* for a few minutes, and the
certificate status changes to **Valid**.

<figure class="shot">
<img src="images/step23.png" loading="lazy"
alt="rootcert.cer open in Notepad with the Base-64 body selected" />
<figcaption>Copy only the Base-64 body.</figcaption>
</figure>

<figure class="shot">
<img src="images/step24.png" loading="lazy"
alt="Point-to-site configuration updating, root certificate status Valid, tunnel type IKEv2 and OpenVPN" />
<figcaption>Root certificate accepted.</figcaption>
</figure>

A shortcut that skips the export wizard: in the PowerShell session where
`$cert` is the root, run `[Convert]::ToBase64String($cert.RawData)` and
paste the output.

<div class="phase">

Phase 6 · Connect from Windows

</div>

### Download and install the VPN client

Click **Download VPN configuration package**. The zip has a folder per
client type. For the native Windows client (IKEv2/SSTP), run
`WindowsAmd64\VpnClientSetupAmd64.exe`. Windows SmartScreen flags it
because the package isn't signed with a well-known publisher; it came
straight from your own gateway, so **More info → Run anyway** is fine
here. For OpenVPN, import the profile from the `AzureVPN` or `OpenVPN`
folder into the Azure VPN Client instead.

<div class="pair">

<figure class="shot">
<img src="images/step25.png" loading="lazy"
alt="Download VPN configuration package and downloaded vpnclientconfiguration.zip" />
<figcaption>The package is generated for this gateway.</figcaption>
</figure>

<figure class="shot">
<img src="images/step26.png" loading="lazy"
alt="WindowsAmd64 folder with VpnClientSetupAmd64" />
<figcaption>Native Windows client installer.</figcaption>
</figure>

</div>

<figure class="shot">
<img src="images/step27.png" loading="lazy"
alt="Windows SmartScreen warning for VpnClientSetupAmd64.exe" />
<figcaption>Expected warning for the generated installer.</figcaption>
</figure>

### Connect

**Settings → Network & Internet → VPN** now lists **VNET-HUB**. Click
**Connect**, then **Connect** in the Azure VPN dialog. Windows asks for
elevation for `CMROUTE.DLL`, which adds the routes for the VNet ranges
to your routing table. Click **Continue**. The status changes to
**Connected**.

<div class="pair">

<figure class="shot">
<img src="images/step28.png" loading="lazy"
alt="Windows VPN settings with VNET-HUB connection" />
<figcaption>The installer added a VPN profile named after the
VNet.</figcaption>
</figure>

<figure class="shot">
<img src="images/step29.png" loading="lazy"
alt="Azure VPN connection dialog for VNET-HUB" />
<figcaption>Connection Manager dialog.</figcaption>
</figure>

</div>

<div class="pair">

<figure class="shot">
<img src="images/step30.png" loading="lazy"
alt="Prompt for elevated privilege to run CMROUTE.DLL to update routing table" />
<figcaption>CMROUTE installs routes to the VNets.</figcaption>
</figure>

<figure class="shot">
<img src="images/step31.png" loading="lazy"
alt="VNET-HUB VPN status Connected" />
<figcaption>Connected.</figcaption>
</figure>

</div>

## 6. Reach the spoke: troubleshooting the first ping

With the VPN connected, the real test is reaching a VM by its private
IP. The VM is `vnet1srv01` at `10.0.0.4` in **VNET-SP1**. The first
attempt failed:

<figure class="shot">
<img src="images/step32.png" loading="lazy"
alt="ping 10.0.0.4 returning Request timed out with 100% loss" />
<figcaption>Connected to the VPN, but every ping to the spoke VM timed
out.</figcaption>
</figure>

### Step 1: check the client's routes

A timeout over a VPN usually means the laptop doesn't send that traffic
into the tunnel at all. Check which Azure ranges the VPN client
installed:

    route print | findstr "10.0.0.0 12.0.0.0"

             12.0.0.0      255.255.0.0         On-link       192.168.1.2     43

Only the hub's `12.0.0.0/16` was there, reached through the laptop's VPN
address `192.168.1.2`. There was **no route for SP1's `10.0.0.0/16`**,
so pings to `10.0.0.4` went out through the normal internet connection
and disappeared.

### Step 2: understand why

The gateway only advertises VNets it is allowed to carry traffic for.
VNET-SP1 is peered to the hub, but the two **gateway transit** settings
were off (section 4). So the gateway told clients about the hub only,
and SP1 had no route back to the VPN address pool either.

### Step 3: enable gateway transit on both sides of the peering

**Hub side:** VNET-HUB → **Peerings → SP1-HUB** → tick *Allow gateway or
route server in 'VNET-HUB' to forward traffic to 'VNET-SP1'* → **Save**.
Leave *Enable 'VNET-HUB' to use 'VNET-SP1's' remote gateway* unticked,
because SP1 has no gateway.

<figure class="shot">
<img src="images/step33.png" loading="lazy"
alt="SP1-HUB peering settings with Allow gateway or route server in VNET-HUB to forward traffic to VNET-SP1 ticked" />
<figcaption>Hub side: gateway transit allowed for VNET-SP1.</figcaption>
</figure>

**Spoke side:** VNET-SP1 → **Peerings** → the link to the hub
(`hub-sp1`) → tick *Enable 'VNET-SP1' to use 'VNET-HUB's' remote gateway
or route server* → **Save**. Wait for the peering state to show
**Connected** again. Repeat both sides for VNET-SP2 if you want VPN
access there too.

The same in Cloud Shell:

    az network vnet peering update -g Network-RG --vnet-name VNET-HUB -n SP1-HUB \
      --set allowGatewayTransit=true

    # Find the spoke-side peering name, then enable remote gateway use
    az network vnet peering list -g Network-RG --vnet-name VNET-SP1 --query "[].name" -o tsv
    az network vnet peering update -g Network-RG --vnet-name VNET-SP1 -n hub-sp1 \
      --set useRemoteGateways=true

### Step 4: reinstall the VPN client

The Windows client gets its route list when it's installed, so a client
installed earlier never learns about SP1. Disconnect, download the VPN
configuration package again, run the new
`WindowsAmd64\VpnClientSetupAmd64.exe`, reconnect, and click
**Continue** on the CMROUTE prompt.

### Step 5: test again

    route print | findstr "10.0.0.0 12.0.0.0"   :: now shows both 12.0.0.0 and 10.0.0.0, on-link via 192.168.1.x
    ping 10.0.0.4                                :: Reply from 10.0.0.4
    ssh nsunke@10.0.0.4                          :: SSH over the VPN, no public IP needed

The ping gets replies and SSH to `10.0.0.4` works. In the portal,
vpn-gateway → Point-to-site configuration → **Allocated IP addresses**
lists the laptop's VPN address. Since the VM is now reachable privately,
you can remove its public IP and the internet-facing `ssh` rule from the
[NSG
post](../azure-network-security-groups-nsg/azure-network-security-groups-nsg.html).

### If it still times out

<div class="tbl">

| Check             | How                                                   | What to look for                                                                                                                                |
|-------------------|-------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------|
| Client route      | `route print`                                         | A line for the spoke's range. If it's missing, transit isn't on or the client wasn't reinstalled.                                               |
| VM route back     | VM → Networking → NIC → **Effective routes**          | The P2S pool (`192.168.1.0/24`) with next hop *Virtual network gateway*. If it's missing, *useRemoteGateways* on the spoke peering didn't save. |
| Address overlap   | `ipconfig`                                            | Your Wi-Fi or Ethernet adapter must not be in the P2S pool's range (see fix 1).                                                                 |
| VM running        | Portal → VM overview                                  | Status *Running*, not *Stopped (deallocated)*.                                                                                                  |
| NSG / OS firewall | NIC → **Effective security rules**; `sudo ufw status` | The default *AllowVnetInBound* covers VPN clients; check that no custom Deny rule blocks them. Ubuntu answers ping by default.                  |

</div>

NSGs still apply to VPN traffic. Traffic from the P2S address pool
arrives through the gateway and is normally matched by the
`VirtualNetwork` service tag, so `nsg1`'s default *AllowVnetInBound*
rule lets VPN clients in. To restrict them, add rules with the pool as
the source.

## 7. Two more things to fix

Gateway transit was the fix that got the ping working (section 6). Two
other choices in this lab work, but are worth correcting before you
reuse the design.

<div class="lab">

### Fix 1: change the address pool if it clashes with your LAN

`192.168.1.0/24` is the default LAN range on many home routers. If
`ipconfig` shows your Wi-Fi or Ethernet adapter on `192.168.1.x`, the
VPN and local routes will conflict. Change the pool to an unused range,
such as `172.16.201.0/24`, save, then download and install the client
package again.

</div>

<div class="lab">

### Fix 2: use a private range for the hub

`12.0.0.0/16` isn't in the RFC 1918 private ranges; it's real internet
address space owned by another company. Azure allows it, but resources
in the hub and VPN clients can no longer reach those public addresses.
In a real design, use something like `10.100.0.0/16` for the hub. A
subnet in use can't be renumbered, so changing it means rebuilding the
hub's subnets.

</div>

## 8. CLI reference

    RG=Network-RG; LOC=westus3

    # Hub with GatewaySubnet
    az network vnet create -g $RG -n VNET-HUB -l $LOC --address-prefixes 10.100.0.0/16 \
      --subnet-name SUBNET1 --subnet-prefixes 10.100.0.0/24
    az network vnet subnet create -g $RG --vnet-name VNET-HUB -n GatewaySubnet \
      --address-prefixes 10.100.1.0/27

    # Gateway (runs 20–45 minutes; --no-wait returns immediately)
    az network public-ip create -g $RG -n vpn-ip --sku Standard --zone 1 2 3
    az network vnet-gateway create -g $RG -n vpn-gateway -l $LOC \
      --vnet VNET-HUB --public-ip-addresses vpn-ip \
      --gateway-type Vpn --vpn-type RouteBased --sku VpnGw1AZ --no-wait

    # Peerings with gateway transit (after the gateway exists)
    az network vnet peering create -g $RG -n HUB-to-SP1 --vnet-name VNET-HUB \
      --remote-vnet VNET-SP1 --allow-vnet-access --allow-forwarded-traffic --allow-gateway-transit
    az network vnet peering create -g $RG -n SP1-to-HUB --vnet-name VNET-SP1 \
      --remote-vnet VNET-HUB --allow-vnet-access --allow-forwarded-traffic --use-remote-gateways

    # Point-to-site with certificate auth
    az network vnet-gateway update -g $RG -n vpn-gateway \
      --address-prefixes 172.16.201.0/24 --client-protocol IkeV2 OpenVPN
    az network vnet-gateway root-cert create -g $RG --gateway-name vpn-gateway \
      -n P2SRootCert --public-cert-data "$(sed '1d;$d' rootcert.cer | tr -d '\r\n')"

    # Client package URL
    az network vnet-gateway vpn-client generate -g $RG -n vpn-gateway -o tsv

## 9. Cost and clean-up

<div class="cost">

**The VPN gateway is the expensive part of this lab.** A VpnGw1AZ
gateway and its public IP are billed every hour they exist, whether or
not anyone is connected, and on a free account that uses up credit
quickly. VNets, subnets, peerings (apart from data transfer) and NSGs
cost little or nothing. Delete the gateway as soon as you finish
testing.

</div>

    # Remove the gateway and its IP, keeping the VNets
    az network vnet-gateway delete -g Network-RG -n vpn-gateway
    az network public-ip delete -g Network-RG -n vpn-ip

    # Or remove the whole lab
    az group delete -n Network-RG --yes --no-wait

On the PC, remove the VNET-HUB VPN profile, and delete the certificates
from `certmgr.msc` if you don't need them again. Keep `childcert.pfx`
somewhere safe or delete it; anyone with it and its password could
connect while the gateway exists.

## 10. Quiz: test yourself

Pick an answer to see whether it's right and why.

<div id="quiz" class="quiz">

</div>

Reset quiz

## 11. Key takeaways

- **Hub-and-spoke** puts shared services in one hub VNet and workloads
  in peered spokes. Spokes don't reach each other through peering alone.
- A VPN gateway lives in a subnet named exactly **GatewaySubnet** (/27
  or larger, no NSG). **AZ** SKUs are zone-redundant, and creation takes
  20–45 minutes.
- **Point-to-site** connects single computers. Choose a non-overlapping
  **address pool**, a tunnel type (OpenVPN, IKEv2, SSTP) and
  authentication (certificate, Entra ID, RADIUS).
- With certificate auth, the gateway holds the **root's public key**;
  each client holds a **child certificate with its private key**.
- Spokes use the hub's gateway only with **gateway transit**:
  *allowGatewayTransit* on the hub side, *useRemoteGateways* on the
  spoke side. Without it, the client gets no route to the spoke and
  pings time out. Check with `route print`, and reinstall the client
  after changing peerings.
- Delete the gateway when you're done; it's billed hourly.

**Next in the series:** switching P2S to Microsoft Entra ID
authentication with MFA, and adding Azure Firewall to route
spoke-to-spoke traffic.

Written by Sunke Nithin while learning Azure. Lab performed on an Azure
free account; subscription IDs, network IDs, the directory name, the
gateway's public IP and the credit balance in the screenshots are
blurred.

</div>
