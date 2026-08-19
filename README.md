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
        │   ├── Stateful Firewall (iptables)
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
-   ✅ Linux longest-prefix routing experiment

## Security

-   ✅ Netfilter / iptables fundamentals
-   ✅ Default-deny INPUT policy
-   ✅ Stateful INPUT filtering with conntrack
-   ✅ Loopback traffic explicitly allowed
-   ✅ SSH access permitted
-   ✅ DHCP permitted from the isolated LAN
-   ✅ DNS permitted from the isolated LAN
-   ✅ ICMP Echo Request permitted from the isolated LAN
-   ✅ Default-deny FORWARD policy
-   ✅ Stateful FORWARD filtering with conntrack
-   ✅ LAN-initiated outbound forwarding explicitly permitted
-   ✅ Return ESTABLISHED/RELATED traffic explicitly permitted
-   ✅ Unsolicited WAN-to-LAN NEW traffic blocked and verified
-   ✅ Firewall rules persisted with OpenRC
-   ⏳ Firewall logging
-   ⏳ Additional network hardening
-   ⏳ Port forwarding / DNAT

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
        Router • NAT • DHCP • DNS • Firewall
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

**Status:** 🚧 In Progress

Completed:

-   Netfilter / iptables fundamentals
-   INPUT, OUTPUT and FORWARD chain concepts
-   Default DROP policy for INPUT
-   Stateful INPUT filtering using conntrack
-   SSH firewall rules
-   DHCP firewall rules
-   DNS firewall rules
-   ICMP filtering
-   Loopback handling
-   Default DROP policy for FORWARD
-   Stateful FORWARD filtering using conntrack
-   Explicit LAN-to-WAN NEW forwarding
-   Explicit RELATED/ESTABLISHED return traffic
-   Verification of unsolicited WAN-to-LAN blocking
-   Firewall persistence
-   DNS and SSH troubleshooting under a default-deny firewall
-   Routing-path verification and longest-prefix matching experiment

Next:

-   Firewall logging
-   Additional network hardening
-   Port forwarding / DNAT

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
02-snapshot-management.md
03-ssh-access.md
04-manual-vm-cloning.md
05-virt-clone.md
06-networking-foundations.md
07-building-an-isolated-lan.md
08-persistent-linux-router.md
09-network-services-dhcp-dns.md
10-firewall-security.md
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

Current state:

-   ✅ Alpine-Lab-01 uses a default-deny INPUT firewall
-   ✅ Stateful INPUT connection tracking implemented
-   ✅ SSH, DHCP and DNS explicitly permitted
-   ✅ ICMP Echo Request permitted from the trusted LAN
-   ✅ Loopback traffic permitted
-   ✅ Alpine-Lab-01 uses a default-deny FORWARD firewall
-   ✅ LAN-to-WAN NEW traffic explicitly permitted
-   ✅ RELATED/ESTABLISHED forwarded traffic explicitly permitted
-   ✅ Unsolicited WAN-to-LAN NEW traffic blocked
-   ✅ FORWARD behavior verified with packet counters
-   ✅ Firewall rules persisted
-   ✅ Alpine-Lab-02 verified
-   ✅ Alpine-Lab-03 verified
-   ✅ DNS hostname resolution verified
-   ✅ SSH access by hostname verified
-   ✅ Longest-prefix route selection demonstrated with a temporary /32
    route

Current filter policy:

``` text
INPUT    DROP
FORWARD  DROP
OUTPUT   ACCEPT
```

Current forwarding model:

``` text
LAN -> WAN NEW                    ACCEPT
WAN -> LAN ESTABLISHED/RELATED    ACCEPT
WAN -> LAN unsolicited NEW        DROP
```

Next objective:

**Add firewall logging and continue router hardening.**

After logging and additional hardening, Phase 5 will proceed to **port
forwarding / DNAT**.

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
-   Linux routing
-   Linux network services
-   Netfilter / iptables
-   Stateful firewalling
-   Network security
-   Troubleshooting
-   Infrastructure documentation
-   Git workflow
-   Enterprise Linux administration
-   LPIC-1 preparation

Rather than collecting isolated commands, the goal is to build a
realistic virtual infrastructure while understanding **why every
component works**.
