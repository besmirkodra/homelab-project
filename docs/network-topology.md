# Cross-Site Network Topology & Mesh Routing

This document details the multi-site network architecture connecting the primary hosting node in Germany (`srv`) and the edge monitoring node in Albania (`rmb`) over an encrypted Tailscale mesh VPN overlay.

---

## Architecture Diagram

```text
               +----------------------------------------+
               |        Tailscale Mesh Overlay          |
               |             (100.64.X.Y/10)            |
               +-------------------+--------------------+
                                   |
           +-----------------------+-----------------------+
           |                                               |
           v                                               v
+-----------------------+                       +-----------------------+
|  Site 1: Germany      |                       |  Site 2: Albania      |
|  Node: srv            |                       |  Node: rmb            |
|  LAN: 192.168.1.0/24  |                       |  LAN: 192.168.2.0/24  |
+-----------+-----------+                       +-----------+-----------+
            |                                               |
            +---> LXC 100: nextcloud-de                     +---> LXC 100: Exit Node (Tailscale)
            +---> LXC 101: jellyfin-media                   +---> LXC 101: checkmk-monitoring
            +---> LXC 102: navidrome-audio
            +---> LXC 103: qbittorrent-downloader
            +---> LXC 104: pihole-dns
            +---> LXC 105: dev-webserver

---

## Subnet Breakdown

| Site | Proxmox Host | Local Subnet | Tailscale Virtual Subnet | Primary Workloads |
| :--- | :--- | :--- | :--- | :--- |
| **Germany** | `srv` | `192.168.1.0/24` | `100.64.X.Y/10` | Media, File Storage (Nextcloud), Torrenting, Web Server |
| **Albania** | `rmb` | `192.168.2.0/24` | `100.64.X.Y/10` | Dedicated Exit Node, Checkmk Infrastructure Monitoring |

---

## Routing & Traffic Flow

### 1. Inter-Site Mesh Tunneling
* **Overlay Layer:** All node-to-node communication between `srv` and `rmb` is routed directly using WireGuard-encrypted tunnels managed by Tailscale.
* **Direct Peer Connections:** NAT traversal (STUN/DERP fallback) ensures low-latency point-to-point connections across residential ISP connections.

### 2. Edge Routing & Exit Node Operations (`rmb`)
* **Dedicated Exit Node:** LXC `100` on the Albanian node (`rmb`) is provisioned as an active Tailscale Exit Node.
* **Full-Tunnel Routing:** Remote traffic can be securely backhauled through the Albanian edge node to route external requests through the regional ISP interface.
* **Resource Optimization:** The `rmb` node is streamlined to allocate processing power exclusively to VPN routing and Checkmk telemetry gathering.

### 3. Monitoring Reachability
* **Checkmk Instance (`rmb` / LXC 101):** Reaches containers on the German node (`srv`) over the Tailscale overlay network.
* **ICMP & Agent Checks:** SNMP and Checkmk Raw agents report container health metrics across the mesh network without requiring public port forwarding.

---

## Security Policy & Access Control

1. **Zero Open Ports:** No inbound WAN ports (e.g., 80, 443, 22) are exposed on either site's local router. All ingress traffic requires Tailscale authentication.
2. **Subnet Isolation:** Inter-container routing within each node relies on local bridge interfaces (`vmbr0`), restricting unprivileged LXC containers from unauthorized host access.

