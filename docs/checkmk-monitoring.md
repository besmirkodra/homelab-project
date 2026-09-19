# Cross-Site Infrastructure Monitoring with Checkmk

This document outlines the monitoring setup deployed across the hybrid home-lab environment. A centralized **Checkmk Raw Edition** instance deployed in Albania actively monitors physical hosts, network virtual bridges, and LXC container workloads in Germany over an encrypted Tailscale mesh overlay.

---

## Architecture Overview

```text
+-----------------------------------------------------------------------------------+
|                                 Site 1: Germany                                   |
|                                                                                   |
|  Proxmox Host: srv (192.168.1.1 / 100.64.X.Y)                                     |
|  +-----------------------------------------------------------------------------+  |
|  | LXC 100: nextcloud-de  | LXC 101: jellyfin-media | LXC 104: pihole-dns     |  |
|  | [Checkmk Agent / ICMP] | [Checkmk Agent / ICMP] | [Checkmk Agent / ICMP] |  |
|  +-----------------------------------------------------------------------------+  |
+------------------------------------------^----------------------------------------+
                                           |
                                Tailscale Mesh Overlay
                                   (100.64.X.Y/10)
                                           |
+------------------------------------------v----------------------------------------+
|                                 Site 2: Albania                                   |
|                                                                                   |
|  Proxmox Host: rmb (192.168.2.1 / 100.64.X.Y)                                     |
|  +-----------------------------------------------------------------------------+  |
|  | LXC 101: checkmk-monitoring                                                 |  |
|  | [Checkmk Raw Server Instance]                                              |  |
|  +-----------------------------------------------------------------------------+  |
+-----------------------------------------------------------------------------------+
```
# Metric & Health Coverage
Checkmk collects telemetry across three primary layers without requiring exposed public WAN ports:

## 1. Proxmox Host Telemetry (srv & rmb)
CPU & RAM Utilization: Real-time load averages and per-core usage.

Storage Pools: ZFS / LVM-thin pool usage, S.M.A.R.T. disk health, and mount point status (/mnt/pve/Windows_Mirror).

Kernel & System: Uptime, systemd service state, temperature sensors, and interface traffic on vmbr0.

## 2. LXC Container Health
State Monitoring: Container uptime and lifecycle tracking (running/stopped/failed).

Resource Limits: Memory usage vs. allocated limits (e.g., Nextcloud 3096MB cap) and swap consumption.

Network Throughput: Ingress/egress bandwidth monitoring per virtual interface (veth*).

## 3. Network & Service Monitoring
Cross-Site Latency: Continuous ICMP ping monitoring across the Tailscale overlay to track packet loss and jitter between Germany and Albania.

DNS Resolution: Synthetic queries against Pi-hole (LXC 104) to verify resolution performance.

# Agent Configuration & Security
## 1. Transport Layer: All monitoring data transmitted between Germany and Albania is encapsulated inside WireGuard tunnels via Tailscale, enforcing zero-trust access control.

## 2. Agent Deployment:

- Proxmox hosts run the lightweight check-mk-agent daemon restricted to query requests from the Checkmk container IP.

- Container monitoring uses standard SNMP polling and local agent plugins where applicable.

## 3. Alerting Thresholds:

- CPU / Memory: WARNING at 85% usage, CRITICAL at 95% usage for > 5 minutes.

- Disk Space: WARNING at 80% capacity, CRITICAL at 90% capacity on storage mounts.

- Tailscale Connectivity: CRITICAL alert triggered if inter-site ping latency exceeds 150ms or packet loss exceeds 5%.
