# Multi-Site Dual-Node Proxmox Home Lab Infrastructure

Architectural documentation, container configurations, and network topologies for a distributed multi-site Proxmox VE environment spanning Germany and Albania.

## Infrastructure Overview

* **German Node (`srv`):** Primary hosting node for media streaming, Nextcloud file management, torrenting, and local web development workloads.
* **Albanian Node (`rmb`):** Edge service node running a dedicated Tailscale Exit Node and Checkmk infrastructure monitoring instance.
* **Inter-Site Connectivity:** Encrypted Tailscale mesh overlay interconnecting isolated local subnets across international sites.

## Repository Structure



text
├── docs/               # Network topologies and architecture documentation
├── proxmox/            # LXC container configurations for srv and rmb nodes
├── tailscale/          # Mesh routing layout and ACL rules
└── scripts/            # Infrastructure maintenance and backup scripts
