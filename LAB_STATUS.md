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

**Stateful INPUT and FORWARD firewalls complete and persisted. Firewall
logging and additional hardening are next.**

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
FORWARD  DROP
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

Persisted FORWARD rules:

``` text
FORWARD DROP
│
├── RELATED,ESTABLISHED         ACCEPT
└── eth1 -> eth0
    source 10.10.10.0/24
    NEW                         ACCEPT
```

The resulting forwarding policy is:

``` text
LAN -> WAN
NEW traffic                         ACCEPT

WAN -> LAN
ESTABLISHED,RELATED traffic         ACCEPT

WAN -> LAN
Unsolicited NEW traffic             DROP
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

Latest saved firewall state:

``` text
Wed Aug 19 09:43:26 2026
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

## Phase 5 --- Firewall & Security

### INPUT firewall completed

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

✔ DNS troubleshooting under a default-deny firewall

✔ SSH hostname and host-key troubleshooting

✔ Alpine-Lab-02 connectivity verified

✔ Alpine-Lab-03 connectivity verified

### FORWARD firewall completed

✔ Observed traffic under the original `FORWARD ACCEPT` policy

✔ Added `RELATED,ESTABLISHED` forwarding rule

✔ Added explicit `NEW` forwarding rule from eth1 to eth0 for
10.10.10.0/24

✔ Used packet counters to verify conntrack state handling

✔ Confirmed normal client traffic matched the explicit FORWARD rules

✔ Changed the default FORWARD policy from ACCEPT to DROP

✔ Verified outbound traffic from Alpine-Lab-02

✔ Verified outbound traffic from Alpine-Lab-03

✔ Verified legitimate outbound traffic did not reach the DROP policy

✔ Verified unsolicited external-to-LAN NEW traffic is dropped

✔ Persisted the completed stateful FORWARD firewall

### Routing experiment completed

Linux Mint was found to have a directly connected route:

``` text
10.10.10.0/24 dev virbr10
```

Therefore traffic from Linux Mint directly to Alpine-Lab-02 normally
bypasses Alpine-Lab-01.

To deliberately force one destination through the router, the temporary
route:

``` bash
sudo ip route add 10.10.10.2/32 via 192.168.122.252 dev virbr0
```

was added.

This demonstrated Linux **longest-prefix matching**:

``` text
/32 > /24 > /0
```

The forced packet reached Alpine-Lab-01 through eth0 and was rejected by
the `FORWARD DROP` policy because it was unsolicited NEW traffic toward
the LAN.

The FORWARD policy counter recorded:

``` text
1 packet, 84 bytes
```

The temporary `/32` route was then removed, restoring Linux Mint's
normal routing table.

### Important INPUT troubleshooting completed

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
-   ✔ Internet connectivity through the stateful FORWARD firewall
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
-   ✔ Internet connectivity through the stateful FORWARD firewall
-   ✔ DNS
-   ✔ SSH from Alpine-Lab-01 by IP
-   ✔ SSH from Alpine-Lab-01 by hostname

------------------------------------------------------------------------

# Stateful FORWARD Verification

Before testing both clients:

``` text
RELATED,ESTABLISHED    10
NEW                     2
DROP policy              0
```

After the outbound tests:

``` text
RELATED,ESTABLISHED    30
NEW                     6
DROP policy              0
```

Delta:

``` text
RELATED,ESTABLISHED    +20
NEW                     +4
DROP                     +0
```

This showed that legitimate LAN-initiated traffic was fully handled by
the explicit stateful rules.

The deliberate unsolicited WAN-to-LAN test then produced:

``` text
FORWARD policy DROP     1 packet / 84 bytes
```

This verified both sides of the firewall design:

``` text
LAN -> WAN NEW                    ACCEPT
WAN -> LAN ESTABLISHED/RELATED    ACCEPT
WAN -> LAN unsolicited NEW        DROP
```

------------------------------------------------------------------------

# Next Goal

Continue **Phase 5 --- Firewall & Security**.

The core stateful firewall is now operational:

``` text
INPUT policy       DROP
FORWARD policy     DROP
OUTPUT policy      ACCEPT
```

Next objectives:

-   Firewall logging
-   Additional network hardening
-   Review service exposure
-   Port forwarding / DNAT
-   Further packet inspection and troubleshooting

------------------------------------------------------------------------

# Latest Snapshots

Previous Phase 5 INPUT firewall checkpoint snapshots were created after
the INPUT firewall documentation and Git checkpoint.

A new snapshot checkpoint should be created after the current stateful
FORWARD documentation is committed and pushed.

------------------------------------------------------------------------

# Repository Status

Current branch:

main

Current documentation milestone:

``` text
10-firewall-security.md
```

Current state:

Phase 5 stateful INPUT and FORWARD firewall documentation prepared.

Files updated for this checkpoint:

``` text
docs/10-firewall-security.md
README.md
LAB_STATUS.md
CHANGELOG.md
```

Next repository actions:

1.  Update `CHANGELOG.md`.
2.  Review changes with `git status` and `git diff`.
3.  Commit the stateful FORWARD firewall checkpoint.
4.  Push `main`.
5.  Shut down the VMs cleanly.
6.  Create VM snapshots.
7.  Create the corresponding Linux Mint / VirtualBox snapshot.
8.  Continue Phase 5 with firewall logging and additional hardening.
