# KVM Virtualization Lab

A hands-on learning repository focused on **KVM, QEMU, libvirt, Alpine
Linux, Linux system administration, networking, virtualization, and
LPIC-1 preparation**.

The objective of this project is not simply to learn commands, but to
build, document, and understand a complete virtual infrastructure using
industry best practices.

------------------------------------------------------------------------

# Current Lab Architecture

``` text
Windows 11 Host
└── VirtualBox
    └── Linux Mint 22.2 (KVM Host)
        ├── Alpine-Lab-01
        │   ├── Router
        │   ├── NAT Gateway
        │   ├── DHCP Server (dnsmasq)
        │   ├── DNS Server (dnsmasq)
        │   ├── SSH Server
        │   └── 10.10.10.1
        │
        ├── Alpine-Lab-02
        │   ├── DHCP Client
        │   └── Internal Client
        │
        └── Alpine-Lab-03
            ├── DHCP Client
            └── Internal Client
```

This is a **nested virtualization laboratory**.

``` text
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

The entire environment is managed primarily from the command line using
**virsh**, SSH, and standard Linux administration tools.

------------------------------------------------------------------------

# Current Progress

## Infrastructure

-   ✅ Nested virtualization operational
-   ✅ KVM/QEMU installed
-   ✅ libvirt configured
-   ✅ virt-manager installed
-   ✅ Alpine Linux 3.24 deployed
-   ✅ VirtIO storage configured

## Virtual Machines

-   ✅ Alpine-Lab-01
-   ✅ Alpine-Lab-02
-   ✅ Alpine-Lab-03

## Administration

-   ✅ SSH key authentication
-   ✅ Secure remote administration
-   ✅ OpenRC service management
-   ✅ Git repository workflow
-   ✅ Documentation for every milestone

## Virtualization

-   ✅ virsh command-line management
-   ✅ VM cloning (manual)
-   ✅ VM cloning (virt-clone)
-   ✅ Snapshot strategy
-   ✅ XML inspection
-   ✅ VM lifecycle management

## Networking

-   ✅ Default libvirt NAT network
-   ✅ Custom isolated network
-   ✅ Linux router configuration
-   ✅ Static addressing
-   ✅ IP forwarding
-   ✅ NAT using iptables
-   ✅ Internet access through Alpine-Lab-01
-   ✅ Inter-VM routing
-   ✅ DHCP using dnsmasq
-   ✅ Static DHCP reservations
-   ✅ DNS forwarding
-   ✅ Local DNS zone (lab.local)
-   ✅ Automatic hostname resolution

------------------------------------------------------------------------

# Current Topology

``` text
                           Internet
                               │
                        libvirt NAT
                               │
                    192.168.122.0/24
                               │
                     Alpine-Lab-01
             Router • NAT • DHCP • DNS
           192.168.122.x / 10.10.10.1
                               │
                     lab-isolated
                      10.10.10.0/24
                    ┌──────────┴──────────┐
                    │                     │
              Alpine-Lab-02        Alpine-Lab-03
               DHCP Client          DHCP Client
                10.10.10.2           10.10.10.3
```

------------------------------------------------------------------------

# Roadmap

## Phase 1 --- KVM Fundamentals

**Status:** ✅ Complete

## Phase 2 --- Virtual Machine Management

**Status:** ✅ Complete

## Phase 3 --- Networking Foundations

**Status:** ✅ Complete

## Phase 4 --- Network Services

**Status:** ✅ Complete

Topics

-   DHCP server (dnsmasq)
-   DHCP reservations
-   DNS server
-   DNS forwarding
-   Local DNS zone
-   Search domains
-   Automatic hostname resolution
-   Modular configuration using `/etc/dnsmasq.d`

------------------------------------------------------------------------

## Phase 5 --- Firewall & Security

Topics

-   Netfilter architecture
-   iptables
-   Stateful firewalling
-   Default DROP policies
-   Logging
-   Network hardening
-   Port forwarding

------------------------------------------------------------------------

## Phase 6 --- Advanced Networking

-   VLANs
-   Static routes
-   Linux bridges
-   Bonding
-   VPN concepts

------------------------------------------------------------------------

## Phase 7 --- Storage Management

-   qcow2
-   qemu-img
-   Storage pools
-   Storage volumes
-   Filesystems

------------------------------------------------------------------------

## Phase 8 --- Automation

-   VM templates
-   cloud-init
-   Automated provisioning

------------------------------------------------------------------------

## Phase 9 --- Linux Server Services

-   nginx
-   Apache
-   PostgreSQL
-   MariaDB
-   Samba
-   NFS

------------------------------------------------------------------------

# Documentation

``` text
docs/

01-virsh-fundamentals.md
02-ssh-access.md
03-manual-vm-cloning.md
04-virt-clone.md
05-networking-foundations.md
06-building-an-isolated-lan.md
07-persistent-linux-router.md
08-network-services-dhcp-dns.md
```

Additional documentation is added after every completed milestone.

------------------------------------------------------------------------

# Repository Structure

``` text
kvm-virtualization-lab/

├── README.md
├── docs/
├── commands/
├── xml/
│   ├── vm-definitions/
│   └── network-definitions/
├── scripts/
└── images/
```

------------------------------------------------------------------------

# Documentation Style

Every lab document follows the same structure:

-   Objective
-   Environment
-   Theory
-   Commands
-   Explanation
-   Expected Output
-   Troubleshooting
-   Final Verification
-   LPIC-1 Relevance

------------------------------------------------------------------------

# Git Workflow

Every completed milestone includes:

-   Documentation
-   Git commit
-   VM snapshots
-   Verification tests

This guarantees that the laboratory can always be restored to a known
working state.

------------------------------------------------------------------------

# Current Milestone

## Phase 5 --- Firewall & Security

Next objectives:

-   Understand Linux Netfilter architecture
-   Learn iptables fundamentals
-   Implement a stateful firewall
-   Harden Alpine-Lab-01
-   Secure traffic between lab networks

------------------------------------------------------------------------

# Final Project

Build and fully document a small enterprise-style virtual
infrastructure.

``` text
Linux Mint KVM Host
│
└── libvirt
    │
    ├── Alpine Router
    ├── Debian Web Server (nginx)
    ├── Rocky Linux Database Server (PostgreSQL)
    └── Linux Client
```

The environment will be:

-   Managed entirely from the command line
-   Administered remotely using SSH
-   Connected through custom virtual networks
-   Protected with snapshots and backups
-   Fully documented
-   Version controlled with Git

------------------------------------------------------------------------

# Learning Objectives

This repository demonstrates practical skills in:

-   Linux system administration
-   Virtualization
-   KVM/QEMU
-   libvirt
-   Networking
-   DHCP
-   DNS
-   Linux network services
-   Troubleshooting
-   Infrastructure documentation
-   Git workflow
-   Enterprise Linux administration
-   LPIC-1 preparation

Rather than collecting isolated commands, the goal is to build a
realistic virtual infrastructure while understanding **why every
component works**.
