# KVM Virtualization Lab

A hands-on learning repository focused on **KVM, QEMU, libvirt, Alpine Linux, Linux system administration, networking, virtualization, and LPIC-1 preparation**.

The objective of this project is not simply to learn commands, but to build, document, and understand a complete virtual infrastructure using industry best practices.

---

# Current Lab Architecture

```text
Windows 11 Host
└── VirtualBox
    └── Linux Mint 22.2 (KVM Host)
        ├── Alpine-Lab-01
        │   ├── Router
        │   ├── NAT Gateway
        │   ├── SSH Server
        │   └── 10.10.10.1
        │
        ├── Alpine-Lab-02
        │   └── Internal Client
        │
        └── Alpine-Lab-03
            └── Internal Client
```

This is a **nested virtualization laboratory**.

```
Windows
    ↓
VirtualBox
    ↓
Linux Mint
    ↓
KVM / libvirt
    ↓
Multiple Linux virtual machines
```

The entire environment is managed primarily from the command line using **virsh**, SSH, and standard Linux administration tools.

---

# Current Progress

## Infrastructure

- ✅ Nested virtualization operational
- ✅ KVM/QEMU installed
- ✅ libvirt configured
- ✅ virt-manager installed
- ✅ Alpine Linux 3.24 deployed
- ✅ VirtIO storage configured

## Virtual Machines

- ✅ Alpine-Lab-01
- ✅ Alpine-Lab-02
- ✅ Alpine-Lab-03

## Administration

- ✅ SSH key authentication
- ✅ Secure remote administration
- ✅ OpenRC service management
- ✅ Git repository workflow
- ✅ Documentation for every milestone

## Virtualization

- ✅ virsh command-line management
- ✅ VM cloning (manual)
- ✅ VM cloning (virt-clone)
- ✅ Snapshot strategy
- ✅ XML inspection
- ✅ VM lifecycle management

## Networking

- ✅ Default libvirt NAT network
- ✅ Custom isolated network
- ✅ Static addressing
- ✅ Router configuration
- ✅ IP forwarding
- ✅ NAT using iptables
- ✅ Internet access through Alpine-Lab-01
- ✅ Inter-VM routing

---

# Current Topology

```text
                           Internet
                               │
                        libvirt NAT
                               │
                    192.168.122.0/24
                               │
                     Alpine-Lab-01
                    Router / Gateway
                 192.168.122.x
                     10.10.10.1
                               │
                     lab-isolated
                      10.10.10.0/24
                    ┌──────────┴──────────┐
                    │                     │
              Alpine-Lab-02        Alpine-Lab-03
                10.10.10.2            10.10.10.3
```

---

# Roadmap

## Phase 1 — KVM Fundamentals

**Status:** ✅ Complete

Topics

- Nested virtualization
- KVM installation
- libvirt architecture
- virsh fundamentals
- Virtual machine lifecycle

---

## Phase 2 — Virtual Machine Management

**Status:** ✅ Complete

Topics

- VM cloning
- Manual cloning
- virt-clone
- XML editing
- Snapshots
- Recovery
- Backup strategy

---

## Phase 3 — Networking Foundations

**Status:** ✅ Complete

Topics

- Virtual NICs
- Bridges
- NAT
- Routing
- ARP
- ICMP
- TCP/IP fundamentals
- Packet capture
- Custom isolated networks
- Linux router configuration
- Persistent routing
- NAT with iptables

---

## Phase 4 — Network Services

**Status:** 🚧 In Progress

Topics

- DHCP Server
- DNS Server
- Local name resolution
- DHCP reservations
- DNS forwarding
- Service verification

---

## Phase 5 — Security

Topics

- iptables
- nftables
- Stateful firewalling
- Default DROP policies
- Logging
- Network hardening

---

## Phase 6 — Advanced Networking

Topics

- Static routes
- VLANs
- Linux bridges
- Bonding
- Multiple subnets
- VPN concepts
- Port forwarding

---

## Phase 7 — Storage Management

Topics

- qcow2
- raw images
- Sparse files
- qemu-img
- Additional virtual disks
- Filesystems
- Persistent mounts
- Storage pools
- Storage volumes

---

## Phase 8 — Automation

Topics

- VM templates
- cloud-init
- Automated provisioning
- Infrastructure reproducibility

---

## Phase 9 — Linux Server Services

Topics

- nginx
- Apache
- PostgreSQL
- MariaDB
- Samba
- NFS

---

# Documentation

Current documentation

```
docs/

01-virsh-fundamentals.md
02-ssh-access.md
03-manual-vm-cloning.md
04-virt-clone.md
05-networking-foundations.md
06-building-an-isolated-lan.md
```

Additional documentation is added after every completed milestone.

---

# Repository Structure

```text
kvm-virtualization-lab/

├── README.md
│
├── docs/
│
├── commands/
│
├── xml/
│   ├── vm-definitions/
│   └── network-definitions/
│
├── scripts/
│
└── images/
```

---

# Documentation Style

Every lab document follows the same structure.

- Objective
- Environment
- Theory
- Commands
- Explanation
- Expected Output
- Troubleshooting
- Final Verification
- LPIC-1 Relevance

---

# Git Workflow

Every completed milestone includes

- Documentation
- Git commit
- VM snapshot(s)
- Verification tests

This guarantees that the laboratory can always be restored to a known working state.

---

# Current Milestone

## Phase 4 — Network Services

Next objective:

Configure **Alpine-Lab-01** as the DHCP server for the isolated **10.10.10.0/24** network.

Once completed, Alpine-Lab-02 and Alpine-Lab-03 will automatically obtain:

- IP address
- Gateway
- DNS server
- Lease information

exactly as they would on a real enterprise network.

---

# Final Project

Build and fully document a small enterprise-style virtual infrastructure.

```text
Linux Mint KVM Host
│
└── libvirt
    │
    ├── Alpine Router
    │
    ├── Debian Web Server
    │      └── nginx
    │
    ├── Rocky Linux Database Server
    │      └── PostgreSQL
    │
    └── Linux Client
```

The environment will be

- managed entirely from the command line
- administered remotely using SSH
- connected through custom virtual networks
- protected with snapshots and backups
- fully documented
- version controlled with Git

---

# Learning Objectives

This repository is designed to demonstrate practical skills in

- Linux system administration
- Virtualization
- KVM/QEMU
- libvirt
- Networking
- Troubleshooting
- Infrastructure documentation
- Git workflow
- Enterprise Linux administration
- LPIC-1 preparation

Rather than collecting isolated commands, the goal is to build a realistic virtual infrastructure while understanding **why every component works**.
