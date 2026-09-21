# Multi-Site Dual-Node Proxmox Home Lab Infrastructure

Architectural documentation, container configurations, and network topologies for a distributed multi-site Proxmox VE environment spanning Germany and Albania.

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
Documentation Quick Links
Network Topology & Routing Architecture

Checkmk Infrastructure Monitoring Setup

Storage Layout & Hardware Passthrough
