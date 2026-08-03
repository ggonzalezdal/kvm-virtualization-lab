# Lab Status

This file tracks the current state of the KVM Virtualization Lab.

------------------------------------------------------------------------

# Current Phase

## Phase 5 --- Firewall & Security

Current topic:

Linux Netfilter & iptables Fundamentals

Status:

🚧 Ready to Begin

------------------------------------------------------------------------

# Current Infrastructure

``` text
Windows 11 Host
│
└── VirtualBox
    │
    └── Linux Mint 22.2
        │
        └── KVM / libvirt
            │
            ├── Alpine-Lab-01
            │   • Router
            │   • NAT Gateway
            │   • DHCP Server
            │   • DNS Server
            │   • SSH Server
            │   • 10.10.10.1
            │
            ├── Alpine-Lab-02
            │   • DHCP Client
            │   • 10.10.10.2
            │
            └── Alpine-Lab-03
                • DHCP Client
                • 10.10.10.3
```

------------------------------------------------------------------------

# Completed Milestones

## Phase 1 --- KVM Fundamentals

✔ KVM installation

✔ virsh fundamentals

✔ Alpine installation

------------------------------------------------------------------------

## Phase 2 --- Virtual Machine Management

✔ SSH key authentication

✔ Manual VM cloning

✔ virt-clone

✔ XML editing

✔ Snapshot strategy

------------------------------------------------------------------------

## Phase 3 --- Networking Foundations

✔ Custom virtual network

✔ Linux router

✔ Static addressing

✔ IP forwarding

✔ NAT using iptables

✔ Internet access

✔ Inter-VM routing

✔ Documentation complete

------------------------------------------------------------------------

## Phase 4 --- Network Services

✔ dnsmasq installed

✔ DHCP server

✔ Static DHCP reservations

✔ DNS server

✔ DNS forwarding

✔ Local DNS zone (lab.local)

✔ DNS search domain

✔ Automatic hostname resolution

✔ Modular configuration using /etc/dnsmasq.d

✔ Documentation complete

------------------------------------------------------------------------

# Next Goal

Begin **Phase 5 -- Firewall & Security**

Objectives:

-   Understand Netfilter architecture
-   Learn iptables fundamentals
-   Implement a stateful firewall
-   Harden Alpine-Lab-01
-   Secure communication between networks

------------------------------------------------------------------------

# Latest Snapshots

To be created after completion of Phase 4:

-   Linux Mint --- 09-network-services
-   Alpine-Lab-01 --- 09-network-services
-   Alpine-Lab-02 --- 09-network-services
-   Alpine-Lab-03 --- 09-network-services

------------------------------------------------------------------------

# Repository Status

Current branch:

main

Working tree:

Ready for Phase 4 documentation commit

Documentation:

Up to date
