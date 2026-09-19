# Storage Architecture & Hardware Device Passthrough

This document outlines the storage pool design, host-to-container bind mount configurations, and GPU device passthrough configurations for containerized workloads across the Proxmox VE nodes.

---

## Storage Pools & Volume Layout

The primary node (`srv`) utilizes a hybrid storage architecture separating high-speed OS/container root filesystems from bulk data storage.

| Storage Pool | Storage Type | Location / Mount Point | Purpose |
| :--- | :--- | :--- | :--- |
| **`local-lvm`** | LVM-Thin | SSD / NVMe | LXC root filesystems (`/`), fast I/O container disks |
| **`Windows_Mirror`** | NTFS / ZFS / Ext4 | `/mnt/pve/Windows_Mirror` | Bulk storage, Nextcloud media data, persistent shares |

---

## Host-to-LXC Bind Mounts

To maximize performance and eliminate virtual disk encapsulation overhead, bulk data storage directories on the host are directly mounted into unprivileged/privileged LXC containers via Proxmox bind mounts.

### Nextcloud Data Bind Mount (`LXC 100`)
The Nextcloud container accesses high-capacity storage on the host via the `mp0` bind mount definition in `/etc/pve/lxc/100.conf`:

```conf
# Mount point format: mp[N]: /host/path,mp=/container/path
mp0: /mnt/pve/Windows_Mirror/nextcloud_data,mp=/mnt/nextcloud_data
```

Storage Permissions & User Mapping
Host Path: /mnt/pve/Windows_Mirror/nextcloud_data

Container Path: /mnt/nextcloud_data

Access Control: User IDs (uid) and Group IDs (gid) are mapped between the Proxmox host and the LXC container to ensure appropriate read/write privileges for media ingestion and file management.

Hardware GPU Passthrough (/dev/dri)
For hardware-accelerated video encoding and decoding (e.g., Intel Quick Sync / VA-API transcode streams in Jellyfin), Direct Rendering Manager (dri) character devices are passed through from the Proxmox host directly into the container environment.

Device Node Mapping
The following host devices are mapped in the container configuration file (/etc/pve/lxc/100.conf / 101.conf):

```conf
# Render Nodes
dev0: /dev/dri/renderD128,gid=992
dev1: /dev/dri/renderD129,gid=992

# Display Cards
dev2: /dev/dri/card0,gid=44
dev3: /dev/dri/card1,gid=44
```
Driver & Group Permissions
Video & Render Groups: Device permissions are assigned to group IDs 44 (video) and 992 (render) on the host.

Container Access: By explicitly granting access via gid=992 and gid=44, media servers inside LXC containers can interface directly with VA-API / iHD drivers without requiring full hardware emulation or elevated privilege risks.

Serial Device Passthrough
In addition to GPU nodes, character devices for serial communication (e.g., Zigbee/Z-Wave automation dongles or peripheral devices) are passed into LXCs using cgroup v2 rules and LXC mount entries:

```conf
# Enable device cgroup rules for character devices
lxc.cgroup2.devices.allow: c 188:* rwm
lxc.cgroup2.devices.allow: c 189:* rwm


# Bind serial devices
lxc.mount.entry: /dev/serial/by-id dev/serial/by-id none bind,optional,create=dir
lxc.mount.entry: /dev/ttyUSB0 dev/ttyUSB0 none bind,optional,create=file
lxc.mount.entry: /dev/ttyACM0 dev/ttyACM0 none bind,optional,create=file
```
