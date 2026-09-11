# KVM Virtualization Lab

A hands-on learning repository focused on **KVM, QEMU, libvirt, Alpine Linux, Linux system administration, networking, virtualization, containers, automation, and LPIC-1 preparation**.

The objective of this project is not simply to learn commands, but to build, document, automate, and understand a complete virtual infrastructure using industry-relevant practices.

---

# Current Lab Architecture

```text
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
        │   ├── Persistent ext4 Data Storage (/srv/data)
        │   ├── SSH Server
        │   └── 10.10.10.2
        │
        ├── Alpine-Lab-03
        │   ├── DHCP Client
        │   └── Internal Client
        │
        └── Automated VM Provisioning
            ├── Alpine-Template-v2
            ├── qcow2 copy-on-write overlays
            ├── cloud-init / NoCloud
            ├── offline seed injection with qemu-nbd
            ├── virt-install --import
            └── automatic DHCP discovery
```

This is a **nested virtualization laboratory**.

```text
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

The environment is managed primarily from the command line using **virsh**, SSH, and standard Linux administration tools.

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
- ✅ Clean reusable Alpine template
- ✅ Automatically provisioned Alpine instances

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
- ✅ Golden/template image provisioning
- ✅ qcow2 backing images and copy-on-write overlays
- ✅ Automated `virt-install --import` deployment

## Networking

- ✅ Default libvirt NAT network
- ✅ Custom isolated network
- ✅ Linux router configuration
- ✅ Static addressing
- ✅ IP forwarding
- ✅ NAT using iptables
- ✅ Internet access through Alpine-Lab-01
- ✅ Inter-VM routing
- ✅ DHCP using dnsmasq
- ✅ Static DHCP reservations
- ✅ DNS forwarding
- ✅ Local DNS zone (`lab.local`)
- ✅ Automatic hostname resolution
- ✅ Linux longest-prefix routing experiment
- ✅ Automated libvirt DHCP lease discovery

## Security

- ✅ Netfilter / iptables fundamentals
- ✅ Default-deny INPUT policy on router
- ✅ Default-deny FORWARD policy on router
- ✅ Stateful filtering with conntrack
- ✅ Loopback traffic explicitly allowed
- ✅ SSH restricted to trusted sources
- ✅ DHCP and DNS restricted to the isolated LAN
- ✅ ICMP filtering
- ✅ LAN-initiated outbound forwarding
- ✅ Return ESTABLISHED/RELATED forwarding
- ✅ Unsolicited WAN-to-LAN NEW traffic blocked
- ✅ Rate-limited firewall logging
- ✅ Nmap open / closed / filtered behavior verified
- ✅ DROP vs REJECT behavior verified
- ✅ Firewall persistence with OpenRC
- ✅ Host firewall on Alpine-Lab-02

## Storage Management

- ✅ libvirt storage pools and volumes inspected
- ✅ qcow2 and RAW image behavior compared
- ✅ `qemu-img` create / info / check / convert / resize
- ✅ Additional qcow2 data disk created and attached persistently
- ✅ Guest block-device enumeration investigated
- ✅ MBR partition and ext4 filesystem created
- ✅ Persistent `/srv/data` mount configured using filesystem UUID
- ✅ Mount persistence verified after reboot
- ✅ Virtual disk resized from 2 GiB to 3 GiB
- ✅ Partition and ext4 filesystem expanded independently
- ✅ Existing data preserved through resize
- ✅ `/etc/fstab` UUID failure diagnosed and recovered
- ✅ qcow2 backing images used for automated VM provisioning
- ✅ Guest filesystems accessed offline through `qemu-nbd`

## Linux Services & Service Exposure

- ✅ nginx installed on Alpine-Lab-02
- ✅ OpenRC service lifecycle and boot enablement
- ✅ Custom static web page served over HTTP
- ✅ nginx access and error logs inspected
- ✅ nginx process privileges and document-root permissions investigated
- ✅ nginx bound specifically to `10.10.10.2:80`
- ✅ Remote HTTP access verified from the isolated LAN
- ✅ SSH and HTTP restricted to `10.10.10.0/24`
- ✅ Default-deny INPUT policy on Alpine-Lab-02
- ✅ DROP vs REJECT vs no-listener behavior verified
- ✅ nginx and firewall persistence verified after reboot

## Containers & Automation

- ✅ Podman container fundamentals and lifecycle
- ✅ Rootless and daemonless container concepts
- ✅ Images and disposable containers
- ✅ Port publishing
- ✅ Bind mounts and named volumes
- ✅ Custom images using Containerfiles
- ✅ Dedicated container networking
- ✅ PostgreSQL persistent storage
- ✅ Flask + PostgreSQL multi-container application
- ✅ CRUD REST API
- ✅ KVM lab integration
- ✅ Basic stack startup automation

## VM Provisioning & Automation

- ✅ Clean Alpine template built and sealed
- ✅ cloud-init installed and enabled with OpenRC
- ✅ NoCloud datasource configured
- ✅ External NoCloud seed-device failure investigated
- ✅ NoCloud seed injected directly into guest qcow2 filesystem
- ✅ qcow2 copy-on-write instances created from template
- ✅ `qemu-nbd` used for offline guest customization
- ✅ Instance-specific hostname and metadata generated automatically
- ✅ User and SSH key provisioned automatically
- ✅ Bash cleanup safety implemented with `trap`
- ✅ VM created and started with `virt-install --import`
- ✅ DHCP lease detected automatically
- ✅ Final SSH-ready VM created from one provisioning command

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
         Router • NAT • DHCP • DNS • Firewall
            192.168.122.x / 10.10.10.1
                               │
                      lab-isolated
                       10.10.10.0/24
                    ┌──────────┴──────────┐
                    │                     │
              Alpine-Lab-02        Alpine-Lab-03
               nginx :80             DHCP Client
              Host Firewall
                10.10.10.2            10.10.10.3
```

Automated provisioning is performed independently through the Linux Mint KVM host and the libvirt `default` network:

```text
Alpine-Template-v2.qcow2
          │
          │ qcow2 backing image
          ▼
Alpine-Auto-XX.qcow2
          │
          │ qemu-nbd + NoCloud injection
          ▼
virt-install --import
          │
          ▼
cloud-init first boot
          │
          ▼
libvirt DHCP → SSH-ready VM
```

---

# Roadmap

## Phase 1 --- KVM Fundamentals

**Status:** ✅ Complete

## Phase 2 --- Virtual Machine Management

**Status:** ✅ Complete

## Phase 3 --- Networking Foundations

**Status:** ✅ Complete

## Phase 4 --- Network Services

**Status:** ✅ Complete

Completed:

- DHCP server (dnsmasq)
- DHCP reservations
- DNS server
- DNS forwarding
- Local DNS zone
- Search domains
- Automatic hostname resolution
- Modular configuration using `/etc/dnsmasq.d`

---

## Phase 5 --- Firewall & Security

**Status:** ✅ Complete

Completed:

- Netfilter / iptables fundamentals
- INPUT, OUTPUT, and FORWARD chain concepts
- Default-deny INPUT and FORWARD policies
- Stateful filtering with conntrack
- SSH, DHCP, DNS, ICMP, and loopback rules
- Explicit LAN-to-WAN NEW forwarding
- Explicit RELATED/ESTABLISHED return traffic
- Firewall logging
- Nmap open / closed / filtered behavior
- DROP vs REJECT behavior
- `rp_filter` vs iptables investigation
- NAT/MASQUERADE and conntrack verification
- Firewall persistence and reboot recovery

---

## Phase 6 --- Linux Services & Service Exposure

**Status:** ✅ Complete

Completed:

- OpenRC service-management fundamentals
- nginx installation and lifecycle management
- Processes and listening sockets
- HTTP testing with curl
- nginx access and error logs
- Document-root permissions and least privilege
- Specific service binding to `10.10.10.2:80`
- Lab DNS/service-name testing
- Host firewall on Alpine-Lab-02
- Default-deny INPUT policy
- SSH and HTTP restricted to `10.10.10.0/24`
- DROP vs REJECT vs no-listener troubleshooting
- nginx and iptables persistence verified after reboot

---

## Phase 7 --- DNAT / Service Publishing

**Status:** ✅ Complete

Completed:

- Published Alpine-Lab-02 nginx through Alpine-Lab-01
- DNAT mapping: `192.168.122.252:8080 -> 10.10.10.2:80`
- PREROUTING destination translation
- FORWARD-chain rule for published HTTP traffic
- Backend host-firewall rule for upstream clients
- Source-address preservation through DNAT
- conntrack inspection and reverse-NAT verification
- Nmap service detection through the published endpoint
- tcpdump comparison on `eth0`, `eth1`, and `any`
- TCP connection analysis
- Controlled failure testing
- iptables persistence and reboot recovery
- Final HTTP `200 OK` verification

---

## Phase 8 --- Storage Management

**Status:** ✅ Complete

Completed:

- libvirt storage pools and volumes
- qcow2 and RAW sparse-allocation behavior
- `qemu-img` inspection, creation, checking, conversion, and resizing
- Additional qcow2 data disk created and attached persistently
- Guest block-device identification and `vda` / `vdb` enumeration behavior
- MBR partitioning and ext4 filesystem creation
- Persistent `/srv/data` mount using UUID in `/etc/fstab`
- Reboot persistence verification
- End-to-end resize: qcow2 → virtual disk → partition → ext4
- Online ext4 expansion with `resize2fs`
- Data-integrity verification after resizing
- Controlled `/etc/fstab` UUID failure and recovery

---

## Phase 9 --- Containers & Automation

**Status:** ✅ Complete

Completed:

- Podman container fundamentals and lifecycle
- Rootless and daemonless container concepts
- Images and disposable containers
- Port publishing
- Bind mounts and named volumes
- Container logs and environment variables
- Custom images using Containerfiles
- Dedicated container networking with `app-net`
- Container-to-container DNS/name resolution
- PostgreSQL with persistent `postgres-data` volume
- Flask + PostgreSQL multi-container application
- CRUD REST API: GET / POST / PUT / DELETE
- KVM lab access through `10.10.10.254:5000`
- Basic startup automation with `start-stack.sh`

---

## Phase 10 --- VM Provisioning & Automation

**Status:** ✅ Complete

Completed:

- Clean reusable `Alpine-Template-v2` golden image
- cloud-init installation and OpenRC boot integration
- NoCloud datasource configuration
- Investigation of external ISO9660/VFAT NoCloud seed failures
- Direct NoCloud seed injection into the qcow2 guest filesystem
- qcow2 copy-on-write overlays backed by the template
- Offline guest filesystem access with `qemu-nbd`
- Automated `user-data` and instance-specific `meta-data`
- Automatic hostname, user, and SSH key provisioning
- Locked cloud-init user-account issue diagnosed and corrected
- Defensive Bash cleanup using state variables and `trap`
- Automated VM creation with `virt-install --import`
- Automatic libvirt DHCP lease discovery
- End-to-end provisioning verified with `Alpine-Auto-04`
- SSH-ready Alpine VM produced from a single script command

Final provisioning command:

```bash
./provision-vm.sh Alpine-Auto-04
```

Example result:

```text
VM Alpine-Auto-04 created successfully.
IP address: 192.168.122.237
SSH:
  ssh airgon@192.168.122.237
```

---

## Phase 11 --- Final Integration / Capstone

**Status:** ⏳ Next

Planned:

- Integrate the infrastructure built throughout the lab
- Final verification and recovery testing
- Documentation cleanup
- Architecture diagram
- Final README and status review
- Final snapshots and Git checkpoint

---

# Documentation

```text
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
12-service-publishing-dnat.md
13-storage-management.md
14-containers-automation.md
15-vm-provisioning-automation.md
```

Additional documentation is added after every completed milestone.

---

# Repository Structure

```text
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

---

# Documentation Style

Every lab document follows the same general structure:

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

Every completed milestone includes:

- Documentation
- Git commit
- VM snapshots when guest state changes justify them
- Verification tests

This keeps the laboratory recoverable and records known-good checkpoints without creating unnecessary snapshots.

---

# Current Milestone

## Phase 10 --- VM Provisioning & Automation

**Status:** ✅ Complete

Final provisioning pipeline:

```text
Alpine-Template-v2
        │
        │ qcow2 backing image
        ▼
Alpine-Auto-XX.qcow2
        │
        │ qemu-nbd
        ▼
guest root filesystem
        │
        │ inject user-data + meta-data
        ▼
/var/lib/cloud/seed/nocloud/
        │
        ▼
virt-install --import
        │
        ▼
cloud-init / NoCloud
        │
        ├── hostname
        ├── airgon user
        └── SSH public key
        │
        ▼
libvirt DHCP
        │
        ▼
SSH-ready Alpine VM
```

Final Phase 10 state:

- ✅ Clean reusable Alpine template created
- ✅ cloud-init / NoCloud configured
- ✅ External seed-device problem diagnosed
- ✅ Direct seed injection through `qemu-nbd` implemented
- ✅ qcow2 overlay provisioning verified
- ✅ User and SSH key creation automated
- ✅ Safe cleanup logic implemented with `trap`
- ✅ VM creation automated with `virt-install`
- ✅ DHCP address discovery automated
- ✅ `Alpine-Auto-04` provisioned successfully
- ✅ SSH access worked immediately after provisioning

The next project phase is **Phase 11 --- Final Integration / Capstone**.

---

# Final Project

Build and fully document a small enterprise-style virtual infrastructure.

```text
Linux Mint KVM Host
│
└── libvirt
    │
    ├── Alpine-Lab-01
    │   Router / NAT / DHCP / DNS / Firewall
    │
    ├── Alpine-Lab-02
    │   nginx Web Server / Host Firewall / Persistent Storage
    │
    ├── Alpine-Lab-03
    │   Internal Client
    │
    ├── Podman Application Stack
    │   Flask + PostgreSQL + persistent volume
    │
    └── Automated VM Provisioning
        Alpine template + qcow2 overlays
        + cloud-init + qemu-nbd + Bash
```

Completed infrastructure includes:

- Routing and NAT
- DHCP and DNS
- Stateful firewalling
- Linux services
- DNAT / service publishing
- Storage management
- Containers and application/database integration
- Template-based automated VM provisioning

Final remaining phase:

- Final integration / capstone

The environment is:

- Managed primarily from the command line
- Administered remotely using SSH
- Connected through custom virtual networks
- Protected with snapshots and known-good checkpoints
- Fully documented
- Version controlled with Git
- Increasingly automated

---

# Learning Objectives

This repository demonstrates practical skills in:

- Linux system administration
- Virtualization
- KVM/QEMU
- libvirt
- Networking
- DHCP
- DNS
- Linux routing
- Linux network services
- nginx
- Netfilter / iptables
- Stateful firewalling
- Network security
- Storage management
- qcow2 / virtual disk administration
- Filesystems and persistent mounts
- Podman and Linux containers
- Container networking and persistent storage
- Multi-container application architecture
- cloud-init and NoCloud
- Golden/template images
- Copy-on-write VM provisioning
- Offline image customization with `qemu-nbd`
- Bash infrastructure automation
- SSH key provisioning
- Troubleshooting
- Infrastructure documentation
- Git workflow
- Enterprise Linux administration
- LPIC-1 preparation

Rather than collecting isolated commands, the goal is to build a realistic virtual infrastructure while understanding **why every component works**.
