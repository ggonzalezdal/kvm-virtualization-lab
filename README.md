**# KVM Virtualization Lab**

A hands-on learning repository focused on **KVM, QEMU, libvirt, Alpine

Linux, Linux system administration, networking, virtualization, and

LPIC-1 preparation**.

The objective of this project is not simply to learn commands, but to

build, document, and understand a complete virtual infrastructure using

industry best practices.

------------------------------------------------------------------------

**# Current Lab Architecture**

``` text

Windows 11 Host

└── VirtualBox

    └── Linux Mint 22.2 (KVM Host)

        ├── Alpine-Lab-01

        │   ├── Router

        │   ├── NAT Gateway

        │   ├── DHCP Server (dnsmasq)

        │   ├── DNS Server (dnsmasq)

        │   ├── Stateful Firewall (iptables)

        │   ├── SSH Server

        │   └── 10.10.10.1

        │

        ├── Alpine-Lab-02

        │   ├── nginx Web Server

        │   ├── Host Firewall (iptables)

        │   ├── SSH Server

        │   └── 10.10.10.2

        │

        └── Alpine-Lab-03

            ├── DHCP Client

            └── Internal Client

```

This is a ****nested virtualization laboratory****.

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

****virsh****, SSH, and standard Linux administration tools.

------------------------------------------------------------------------

**# Current Progress**

**## Infrastructure**

-   ✅ Nested virtualization operational

-   ✅ KVM/QEMU installed

-   ✅ libvirt configured

-   ✅ virt-manager installed

-   ✅ Alpine Linux 3.24 deployed

-   ✅ VirtIO storage configured

**## Virtual Machines**

-   ✅ Alpine-Lab-01

-   ✅ Alpine-Lab-02

-   ✅ Alpine-Lab-03

**## Administration**

-   ✅ SSH key authentication

-   ✅ Secure remote administration

-   ✅ OpenRC service management

-   ✅ Git repository workflow

-   ✅ Documentation for every milestone

**## Virtualization**

-   ✅ virsh command-line management

-   ✅ VM cloning (manual)

-   ✅ VM cloning (virt-clone)

-   ✅ Snapshot strategy

-   ✅ XML inspection

-   ✅ VM lifecycle management

**## Networking**

-   ✅ Default libvirt NAT network

-   ✅ Custom isolated network

-   ✅ Linux router configuration

-   ✅ Static addressing

-   ✅ IP forwarding

-   ✅ NAT using iptables

-   ✅ Internet access through Alpine-Lab-01

-   ✅ Inter-VM routing

-   ✅ DHCP using dnsmasq

-   ✅ Static DHCP reservations

-   ✅ DNS forwarding

-   ✅ Local DNS zone (`lab.local`)

-   ✅ Automatic hostname resolution

-   ✅ Linux longest-prefix routing experiment

**## Security**

-   ✅ Netfilter / iptables fundamentals

-   ✅ Default-deny INPUT policy on router

-   ✅ Default-deny FORWARD policy on router

-   ✅ Stateful filtering with conntrack

-   ✅ Loopback traffic explicitly allowed

-   ✅ SSH restricted to trusted sources

-   ✅ DHCP and DNS restricted to the isolated LAN

-   ✅ ICMP filtering

-   ✅ LAN-initiated outbound forwarding

-   ✅ Return ESTABLISHED/RELATED forwarding

-   ✅ Unsolicited WAN-to-LAN NEW traffic blocked

-   ✅ Rate-limited firewall logging

-   ✅ Nmap open / closed / filtered behavior verified

-   ✅ DROP vs REJECT behavior verified

-   ✅ Firewall persistence with OpenRC

-   ✅ Host firewall on Alpine-Lab-02

**## Linux Services & Service Exposure**

-   ✅ nginx installed on Alpine-Lab-02

-   ✅ OpenRC service lifecycle and boot enablement

-   ✅ Custom static web page served over HTTP

-   ✅ nginx access and error logs inspected

-   ✅ nginx process privileges and document-root permissions

    investigated

-   ✅ nginx bound specifically to `10.10.10.2:80`

-   ✅ Remote HTTP access verified from the isolated LAN

-   ✅ SSH and HTTP restricted to `10.10.10.0/24`

-   ✅ Default-deny INPUT policy on Alpine-Lab-02

-   ✅ DROP vs REJECT vs no-listener behavior verified

-   ✅ nginx and firewall persistence verified after reboot

------------------------------------------------------------------------

**# Current Topology**

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

                    │                     │

              Alpine-Lab-02        Alpine-Lab-03

               nginx :80            DHCP Client

              Host Firewall

                10.10.10.2           10.10.10.3

```

------------------------------------------------------------------------

**# Roadmap**

**## Phase 1 --- KVM Fundamentals**

****Status:**** ✅ Complete

**## Phase 2 --- Virtual Machine Management**

****Status:**** ✅ Complete

**## Phase 3 --- Networking Foundations**

****Status:**** ✅ Complete

**## Phase 4 --- Network Services**

****Status:**** ✅ Complete

Topics:

-   DHCP server (dnsmasq)

-   DHCP reservations

-   DNS server

-   DNS forwarding

-   Local DNS zone

-   Search domains

-   Automatic hostname resolution

-   Modular configuration using `/etc/dnsmasq.d`

------------------------------------------------------------------------

**## Phase 5 --- Firewall & Security**

****Status:**** ✅ Complete

Completed:

-   Netfilter / iptables fundamentals

-   INPUT, OUTPUT, and FORWARD chain concepts

-   Default-deny INPUT and FORWARD policies

-   Stateful filtering with conntrack

-   SSH, DHCP, DNS, ICMP, and loopback rules

-   Explicit LAN-to-WAN NEW forwarding

-   Explicit RELATED/ESTABLISHED return traffic

-   Firewall logging

-   Nmap open / closed / filtered behavior

-   DROP vs REJECT behavior

-   `rp_filter` vs iptables investigation

-   NAT/MASQUERADE and conntrack verification

-   Firewall persistence and reboot recovery

------------------------------------------------------------------------

**## Phase 6 --- Linux Services & Service Exposure**

****Status:**** ✅ Complete

Completed:

-   OpenRC service-management fundamentals

-   nginx installation and lifecycle management

-   Processes and listening sockets

-   HTTP testing with curl

-   nginx access and error logs

-   Document-root permissions and least privilege

-   Specific service binding to `10.10.10.2:80`

-   Lab DNS/service-name testing

-   Host firewall on Alpine-Lab-02

-   Default-deny INPUT policy

-   SSH and HTTP restricted to `10.10.10.0/24`

-   DROP vs REJECT vs no-listener troubleshooting

-   nginx and iptables persistence verified after reboot

------------------------------------------------------------------------

**## Phase 7 --- DNAT / Service Publishing**

**Status:** ✅ Complete

Completed:

-   Published Alpine-Lab-02 nginx through Alpine-Lab-01

-   DNAT mapping: `192.168.122.252:8080 -> 10.10.10.2:80`

-   PREROUTING destination translation

-   FORWARD-chain rule for published HTTP traffic

-   Backend host-firewall rule for upstream clients

-   Source-address preservation through DNAT

-   conntrack inspection and reverse-NAT verification

-   Nmap service detection through the published endpoint

-   tcpdump comparison on `eth0`, `eth1`, and `any`

-   TCP SYN / SYN-ACK / ACK / PSH / FIN / RST analysis

-   TCP sequence-number and acknowledgement analysis

-   Controlled failure testing: missing DNAT, missing FORWARD rule,
    backend INPUT DROP, and stopped nginx

-   iptables persistence and reboot recovery

-   Final HTTP `200 OK` verification
------------------------------------------------------------------------

**## Phase 8 --- Storage Management**

-   qcow2

-   qemu-img

-   Storage pools

-   Storage volumes

-   Filesystems

------------------------------------------------------------------------

**## Phase 9 --- Containers & Automation**

-   Containers

-   VM templates

-   cloud-init

-   Automated provisioning

------------------------------------------------------------------------

**## Phase 10 --- Application & Database Architecture**

-   Application services

-   PostgreSQL / MariaDB

-   Multi-tier architecture

-   Service-to-service connectivity

-   Application/database security boundaries

------------------------------------------------------------------------

