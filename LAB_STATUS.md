# Lab Status

This file tracks the current state of the KVM Virtualization Lab.

------------------------------------------------------------------------

# Current Phase

## Phase 5 --- Firewall & Security

Current topic:

Stateful Netfilter / iptables firewalling

Status:

🚧 In Progress

Current checkpoint:

**Stateful INPUT firewall complete and persisted. Stateful FORWARD
filtering is next.**

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
            │   • DHCP Server (dnsmasq)
            │   • DNS Server (dnsmasq)
            │   • Stateful Firewall (iptables)
            │   • SSH Server
            │   • eth0: 192.168.122.252/24
            │   • eth1: 10.10.10.1/24
            │
            ├── Alpine-Lab-02
            │   • Internal Client
            │   • 10.10.10.2/24
            │   • Gateway: 10.10.10.1
            │   • DNS: 10.10.10.1
            │
            └── Alpine-Lab-03
                • DHCP Client
                • Internal Client
                • 10.10.10.3/24
                • Gateway: 10.10.10.1
                • DNS: 10.10.10.1
```

------------------------------------------------------------------------

# Current Firewall State

Alpine-Lab-01 filter policies:

``` text
INPUT    DROP
FORWARD  ACCEPT
OUTPUT   ACCEPT
```

Persisted INPUT rules:

``` text
INPUT DROP
│
├── lo                          ACCEPT
├── RELATED,ESTABLISHED         ACCEPT
├── TCP/22 NEW                  ACCEPT   SSH
├── eth1 UDP/67                 ACCEPT   DHCP
├── eth1 UDP/53                 ACCEPT   DNS
└── eth1 ICMP echo-request      ACCEPT   ping
```

NAT remains enabled for the isolated LAN:

``` text
10.10.10.0/24 -> eth0 -> MASQUERADE
```

Firewall state is persisted in:

``` text
/etc/iptables/rules-save
```

using the Alpine OpenRC `iptables` service.

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

## Phase 5 --- Firewall & Security

### Completed so far

✔ Netfilter / iptables fundamentals

✔ INPUT, OUTPUT and FORWARD chain concepts

✔ Default DROP policy for INPUT

✔ Stateful filtering with conntrack

✔ RELATED,ESTABLISHED traffic handling

✔ SSH permitted through TCP/22

✔ DHCP permitted from the isolated LAN through UDP/67

✔ DNS permitted from the isolated LAN through UDP/53

✔ ICMP Echo Request permitted from the isolated LAN

✔ Loopback traffic explicitly permitted

✔ ICMP NEW / ESTABLISHED conntrack experiment

✔ Firewall rule counters inspected and understood

✔ Firewall persistence configured and verified

✔ DNS troubleshooting under a default-deny firewall

✔ SSH hostname and host-key troubleshooting

✔ Alpine-Lab-02 connectivity verified

✔ Alpine-Lab-03 connectivity verified

### Important troubleshooting completed

A local DNS failure on Alpine-Lab-01 was traced with `tcpdump` to DNS
queries travelling through `lo`. The default-deny INPUT firewall was
blocking this traffic because DNS had only been allowed through `eth1`.

The permanent fix was:

``` text
-A INPUT -i lo -j ACCEPT
```

A second DNS problem caused `alpine-lab-01` to resolve to `127.0.0.1`
from other clients. The source was Alpine-Lab-01's `/etc/hosts` combined
with dnsmasq hostname expansion.

Alpine-Lab-01 now uses:

``` text
127.0.0.1    localhost localhost.localdomain
10.10.10.1   alpine-lab-01.lab.local alpine-lab-01
::1          localhost localhost.localdomain
```

DNS now correctly resolves:

``` text
alpine-lab-01 -> 10.10.10.1
alpine-lab-02 -> 10.10.10.2
```

SSH access by hostname was reverified. The Alpine-Lab-01 ED25519 host
fingerprint was independently checked before accepting the corrected
hostname/key association.

------------------------------------------------------------------------

# Client Health

## Alpine-Lab-02

``` text
IP:       10.10.10.2/24
Gateway:  10.10.10.1
DNS:      10.10.10.1
Domain:   lab.local
```

Verified:

-   ✔ Router connectivity
-   ✔ Internet connectivity
-   ✔ DNS
-   ✔ SSH by IP
-   ✔ SSH by hostname

## Alpine-Lab-03

``` text
IP:       10.10.10.3/24
Gateway:  10.10.10.1
DNS:      10.10.10.1
Domain:   lab.local
```

Verified:

-   ✔ Router connectivity
-   ✔ Internet connectivity
-   ✔ DNS
-   ✔ SSH from Alpine-Lab-01 by IP
-   ✔ SSH from Alpine-Lab-01 by hostname

------------------------------------------------------------------------

# Next Goal

Continue **Phase 5 --- Firewall & Security**.

Next objective:

**Implement a stateful FORWARD firewall on Alpine-Lab-01.**

Current situation:

``` text
FORWARD policy ACCEPT
```

The next stage will move toward:

``` text
FORWARD policy DROP
```

with explicit stateful rules so that:

``` text
LAN -> WAN
NEW traffic                         ACCEPT

WAN -> LAN
ESTABLISHED,RELATED traffic         ACCEPT

WAN -> LAN
Unsolicited NEW traffic             DROP
```

This will apply the same stateful firewall principles already learned
with INPUT to traffic travelling through Alpine-Lab-01.

Later Phase 5 objectives:

-   Firewall logging
-   Additional network hardening
-   Port forwarding

------------------------------------------------------------------------

# Latest Snapshots

Previous network-services checkpoint:

-   Linux Mint --- network-services checkpoint
-   Alpine-Lab-01 --- 07-dhcp-dns-complete
-   Alpine-Lab-02 --- network-services checkpoint
-   Alpine-Lab-03 --- network-services checkpoint

A new **Phase 5 INPUT firewall checkpoint** should now be created before
changing the FORWARD policy.

------------------------------------------------------------------------

# Repository Status

Current branch:

main

Current documentation milestone:

``` text
10-firewall-security.md
```

Current state:

Phase 5 INPUT firewall documentation prepared.

Next repository actions:

1. Update `README.md`, `LAB_STATUS.md`, and `CHANGELOG.md`.
2. Review changes with `git status` / `git diff`.
3. Commit the Phase 5 INPUT firewall checkpoint.
4. Create VM snapshots.
5. Create the corresponding Linux Mint / VirtualBox snapshot.
6. Begin stateful FORWARD firewalling.
