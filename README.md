# 🖥️ Multi-Site Home Lab Infrastructure

[![Proxmox VE](https://img.shields.io/badge/Hypervisor-Proxmox%20VE-E57008?style=flat&logo=proxmox&logoColor=white)](https://www.proxmox.com/)
[![Tailscale](https://img.shields.io/badge/Networking-Tailscale%20Mesh%20VPN-242424?style=flat&logo=tailscale&logoColor=white)](https://tailscale.com/)
[![Linux Containers](https://img.shields.io/badge/Virtualization-LXC%20%2F%20Docker-2496ED?style=flat&logo=docker&logoColor=white)](https://linuxcontainers.org/)
[![Debian](https://img.shields.io/badge/OS-Debian%20%2F%20Ubuntu-A81D33?style=flat&logo=debian&logoColor=white)](https://www.debian.org/)
[![GitHub](https://img.shields.io/badge/Repository-besmirkodra%2Fhomelab--project-181717?style=flat&logo=github)](https://github.com/besmirkodra/homelab-project)

A documented, self-hosted multi-site infrastructure topology featuring virtualized nodes, LXC containers, secure cross-site mesh networking, self-hosted media and storage services, and local DNS ad-blocking, spanning Germany and Albania..


---

## Interactive Architecture & Network Diagram

```mermaid
flowchart TB
    subgraph OVERLAY["Tailscale Encrypted Mesh Overlay (100.64.X.Y/10)"]
        direction LR
        TS_GER["Tailscale Node: srv"] <-->|WireGuard Mesh Tunnel| TS_ALB["Tailscale Node: rmb"]
    end

    subgraph DE["Site 1: Germany (Aachen) - Host: srv"]
        direction TB
        DE_HOST["Proxmox VE Host (srv)<br/>LAN: 192.168.1.1"]
        DE_BRIDGE["Virtual Bridge (vmbr0)"]
        DE_HOST --- DE_BRIDGE

        subgraph DE_CONTAINERS["LXC Workloads"]
            CT100["CT 100: nextcloud-de<br/>File Storage & Data Mounts"]
            CT101["CT 101: jellyfin-media<br/>GPU Passthrough (/dev/dri)"]
            CT102["CT 102: navidrome-audio<br/>Music Streaming"]
            CT103["CT 103: qbittorrent-downloader<br/>Download Client"]
            CT104["CT 104: pihole-dns<br/>DNS Ad-blocking"]
            CT105["CT 105: dev-webserver<br/>Development Web Hosting"]
        end

        DE_BRIDGE --- DE_CONTAINERS
    end

    subgraph AL["Site 2: Albania - Host: rmb"]
        direction TB
        AL_HOST["Proxmox VE Host (rmb)<br/>LAN: 192.168.2.1"]
        AL_BRIDGE["Virtual Bridge (vmbr0)"]
        AL_HOST --- AL_BRIDGE

        subgraph AL_CONTAINERS["LXC Workloads"]
            AL_CT100["CT 100: rmb-vpn<br/>Dedicated Exit Node"]
            AL_CT101["CT 101: checkmk-monitoring<br/>Checkmk Raw Instance"]
        end

        AL_BRIDGE --- AL_CONTAINERS
    end

    %% Cross-site Monitoring Connections
    AL_CT101 -.->|Telemetry & Health Checks| DE_HOST
    AL_CT101 -.->|Agent / ICMP Checks| DE_CONTAINERS
```
---

## Infrastructure Overview

* **German Node (`srv`):** Primary hosting node in Germany for media streaming, Nextcloud file management, torrenting, and local web development workloads.
* **Albanian Node (`rmb`):** Edge service node in Albania running a dedicated Tailscale Exit Node and Checkmk infrastructure monitoring instance.
* **Inter-Site Connectivity:** Encrypted Tailscale mesh overlay interconnecting isolated local subnets across international sites.

---

## Repository Structure

```text
├── docs/
│   ├── network-topology.md      # Tailscale mesh routing and subnet breakdown
│   ├── checkmk-monitoring.md    # Cross-site telemetry and agent configuration
│   └── storage-architecture.md  # Storage mounts and GPU passthrough (/dev/dri)
├── proxmox/
│   ├── german-node-srv/configs/ # LXC container configuration templates (100–105)
│   └── albanian-node-rmb/configs/# LXC container configuration templates (100–101)
├── tailscale/                   # Mesh routing policies and ACL definitions
└── scripts/                     # Infrastructure maintenance and backup scripts
```

---

## Documentation Quick Links

* [Network Topology & Routing Architecture](docs/network-topology.md)
* [Checkmk Infrastructure Monitoring Setup](docs/checkmk-monitoring.md)
* [Storage Layout & Hardware Passthrough](docs/storage-architecture.md)
