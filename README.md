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
        │   ├── nginx Web Server
        │   ├── Host Firewall (iptables)
        │   ├── SSH Server
        │   └── 10.10.10.2
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
-   ✅ Local DNS zone (`lab.local`)
-   ✅ Automatic hostname resolution
-   ✅ Linux longest-prefix routing experiment

## Security

-   ✅ Netfilter / iptables fundamentals
-   ✅ Default-deny INPUT policy on router
-   ✅ Default-deny FORWARD policy on router
-   ✅ Stateful filtering with conntrack
-   ✅ Loopback traffic explicitly allowed
-   ✅ SSH restricted to trusted sources
-   ✅ DHCP and DNS restricted to the isolated LAN
-   ✅ ICMP filtering
-   ✅ LAN-initiated outbound forwarding
-   ✅ Return ESTABLISHED/RELATED forwarding
-   ✅ Unsolicited WAN-to-LAN NEW traffic blocked
-   ✅ Rate-limited firewall logging
-   ✅ Nmap open / closed / filtered behavior verified
-   ✅ DROP vs REJECT behavior verified
-   ✅ Firewall persistence with OpenRC
-   ✅ Host firewall on Alpine-Lab-02

## Linux Services & Service Exposure

-   ✅ nginx installed on Alpine-Lab-02
-   ✅ OpenRC service lifecycle and boot enablement
-   ✅ Custom static web page served over HTTP
-   ✅ nginx access and error logs inspected
-   ✅ nginx process privileges and document-root permissions
    investigated
-   ✅ nginx bound specifically to `10.10.10.2:80`
-   ✅ Remote HTTP access verified from the isolated LAN
-   ✅ SSH and HTTP restricted to `10.10.10.0/24`
-   ✅ Default-deny INPUT policy on Alpine-Lab-02
-   ✅ DROP vs REJECT vs no-listener behavior verified
-   ✅ nginx and firewall persistence verified after reboot

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
               nginx :80            DHCP Client
              Host Firewall
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

Topics:

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

**Status:** ✅ Complete

Completed:

-   Netfilter / iptables fundamentals
-   INPUT, OUTPUT, and FORWARD chain concepts
-   Default-deny INPUT and FORWARD policies
-   Stateful filtering with conntrack
-   SSH, DHCP, DNS, ICMP, and loopback rules
-   Explicit LAN-to-WAN NEW forwarding
-   Explicit RELATED/ESTABLISHED return traffic
-   Firewall logging
-   Nmap open / closed / filtered behavior
-   DROP vs REJECT behavior
-   `rp_filter` vs iptables investigation
-   NAT/MASQUERADE and conntrack verification
-   Firewall persistence and reboot recovery

------------------------------------------------------------------------

## Phase 6 --- Linux Services & Service Exposure

**Status:** ✅ Complete

Completed:

-   OpenRC service-management fundamentals
-   nginx installation and lifecycle management
-   Processes and listening sockets
-   HTTP testing with curl
-   nginx access and error logs
-   Document-root permissions and least privilege
-   Specific service binding to `10.10.10.2:80`
-   Lab DNS/service-name testing
-   Host firewall on Alpine-Lab-02
-   Default-deny INPUT policy
-   SSH and HTTP restricted to `10.10.10.0/24`
-   DROP vs REJECT vs no-listener troubleshooting
-   nginx and iptables persistence verified after reboot

------------------------------------------------------------------------

## Phase 7 --- DNAT / Service Publishing

**Status:** ⏳ Next

Planned:

-   Publish Alpine-Lab-02 nginx through Alpine-Lab-01
-   DNAT / port forwarding
-   Example mapping: `Alpine-Lab-01:8080 -> 10.10.10.2:80`
-   FORWARD-chain implications
-   Source-address and host-firewall behavior
-   End-to-end packet-flow verification

------------------------------------------------------------------------

## Phase 8 --- Storage Management

-   qcow2
-   qemu-img
-   Storage pools
-   Storage volumes
-   Filesystems

------------------------------------------------------------------------

## Phase 9 --- Containers & Automation

-   Containers
-   VM templates
-   cloud-init
-   Automated provisioning

------------------------------------------------------------------------

## Phase 10 --- Application & Database Architecture

-   Application services
-   PostgreSQL / MariaDB
-   Multi-tier architecture
-   Service-to-service connectivity
-   Application/database security boundaries

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
11-linux-services-nginx.md
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

## Phase 6 --- Linux Services & Service Exposure

**Status:** ✅ Complete

Final state on Alpine-Lab-02:

-   ✅ nginx installed and managed with OpenRC
-   ✅ nginx enabled at boot and verified after reboot
-   ✅ Custom page available at `/lab.html`
-   ✅ nginx bound specifically to `10.10.10.2:80`
-   ✅ Access and error logging verified
-   ✅ nginx worker permissions and document-root access investigated
-   ✅ Host firewall implemented with iptables
-   ✅ Default-deny `INPUT` policy
-   ✅ Loopback explicitly allowed
-   ✅ `ESTABLISHED,RELATED` traffic allowed
-   ✅ NEW SSH connections allowed from `10.10.10.0/24`
-   ✅ NEW HTTP connections allowed from `10.10.10.0/24`
-   ✅ Unapproved inbound ports dropped
-   ✅ DROP vs REJECT vs stopped-service behavior verified
-   ✅ Firewall rules saved to `/etc/iptables/rules-save`
-   ✅ iptables enabled through OpenRC
-   ✅ Firewall persistence verified after reboot
-   ✅ HTTP and SSH connectivity verified after reboot

Current nginx exposure:

``` text
10.10.10.2:80 -> nginx
```

Current Alpine-Lab-02 INPUT model:

``` text
lo                                  ACCEPT
ESTABLISHED,RELATED                 ACCEPT
10.10.10.0/24 -> TCP/22 NEW        ACCEPT
10.10.10.0/24 -> TCP/80 NEW        ACCEPT
everything else                    DROP
```

The next project step is to commit, push, and snapshot this known-good
Phase 6 state before beginning **Phase 7 --- DNAT / Service
Publishing**.

------------------------------------------------------------------------

# Final Project

Build and fully document a small enterprise-style virtual
infrastructure.

``` text
Linux Mint KVM Host
│
└── libvirt
    │
    ├── Alpine-Lab-01  Router / NAT / DHCP / DNS / Firewall
    ├── Alpine-Lab-02  nginx Web Server / Host Firewall
    └── Alpine-Lab-03  Internal Client

Future phases:
    ├── DNAT / service publishing
    ├── Storage
    ├── Containers & automation
    └── Application / database architecture
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
-   nginx
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
