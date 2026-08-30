**# Lab Status**

This file tracks the current state of the KVM Virtualization Lab.

------------------------------------------------------------------------

**# Current Phase

## Phase 5 --- Firewall & Security

Current topic:

Stateful Netfilter / iptables firewalling, logging, hardening, and recovery

Status:

✅ **Complete**

Current checkpoint:

**The stateful INPUT and FORWARD firewalls are complete, hardened, logged,
persisted, and verified after a full reboot. Phase 5 technical work is
finished; documentation, Git checkpointing, and snapshots remain.**

------------------------------------------------------------------------

# Current Infrastructure**

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

            │   • Router

            │   • NAT Gateway

            │   • DHCP Server (dnsmasq)

            │   • DNS Server (dnsmasq)

            │   • Stateful Firewall (iptables)

            │   • SSH Server

            │   • eth0: 192.168.122.252/24

            │   • eth1: 10.10.10.1/24

            │

            ├── Alpine-Lab-02

            │   • Internal Client

            │   • 10.10.10.2/24

            │   • Gateway: 10.10.10.1

            │   • DNS: 10.10.10.1

            │

            └── Alpine-Lab-03

                • DHCP Client

                • Internal Client

                • 10.10.10.3/24

                • Gateway: 10.10.10.1

                • DNS: 10.10.10.1

```

------------------------------------------------------------------------

**# Current Firewall State

Alpine-Lab-01 filter policies:

```text
INPUT    DROP
FORWARD  DROP
OUTPUT   ACCEPT
```

Persisted INPUT model:

```text
INPUT DROP
│
├── lo                                             ACCEPT
├── RELATED,ESTABLISHED                            ACCEPT
├── eth1 + 10.10.10.0/24 + TCP/22 NEW             ACCEPT  SSH
├── eth1 + UDP/67                                  ACCEPT  DHCP
├── eth1 + 10.10.10.0/24 + UDP/53                 ACCEPT  DNS
├── eth1 + 10.10.10.0/24 + TCP/53 NEW             ACCEPT  DNS
├── eth1 + 10.10.10.0/24 + ICMP echo-request      ACCEPT  ping
└── rate-limited "IPTABLES-DROP:"                  LOG
```

Persisted FORWARD model:

```text
FORWARD DROP
│
├── RELATED,ESTABLISHED                            ACCEPT
├── eth1 -> eth0
│   source 10.10.10.0/24
│   NEW                                            ACCEPT
└── rate-limited "FORWARD-DROP:"                   LOG
```

Resulting forwarding policy:

```text
LAN -> WAN NEW                     ACCEPT
WAN -> LAN ESTABLISHED,RELATED     ACCEPT
WAN -> LAN unsolicited NEW         DROP
```

NAT:

```text
10.10.10.0/24 -> eth0 -> MASQUERADE
```

Firewall state is persisted in:

```text
/etc/iptables/rules-save
```

using the Alpine OpenRC `iptables` service.

Final firewall save:

```text
Thu Aug 27 13:42:40 2026
```

Firewall logging is handled through BusyBox `klogd` and `syslogd`, with
messages currently stored in:

```text
/var/log/messages
```

------------------------------------------------------------------------

# Completed Milestones**

**## Phase 1 --- KVM Fundamentals**

✔ KVM installation

✔ virsh fundamentals

✔ Alpine installation

------------------------------------------------------------------------

**## Phase 2 --- Virtual Machine Management**

✔ SSH key authentication

✔ Manual VM cloning

✔ virt-clone

✔ XML editing

✔ Snapshot strategy

------------------------------------------------------------------------

**## Phase 3 --- Networking Foundations**

✔ Custom virtual network

✔ Linux router

✔ Static addressing

✔ IP forwarding

✔ NAT using iptables

✔ Internet access

✔ Inter-VM routing

✔ Documentation complete

------------------------------------------------------------------------

**## Phase 4 --- Network Services**

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

**## Phase 5 --- Firewall & Security**

**### INPUT firewall completed**

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

**### FORWARD firewall completed**

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

**### Routing experiment completed**

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

This demonstrated Linux ****longest-prefix matching****:

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

**### Important INPUT troubleshooting completed**

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

127.0.0.1    localhost localhost.localdomain

10.10.10.1   alpine-lab-01.lab.local alpine-lab-01

::1          localhost localhost.localdomain

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

**### Firewall logging and hardening completed

✔ Added rate-limited INPUT logging with prefix `IPTABLES-DROP:`

✔ Added rate-limited FORWARD logging with prefix `FORWARD-DROP:`

✔ Verified BusyBox `syslogd` and `klogd`

✔ Confirmed firewall events in `/var/log/messages`

✔ Restricted SSH to `eth1` and source `10.10.10.0/24`

✔ Restricted DNS UDP/53 to `eth1` and source `10.10.10.0/24`

✔ Added and restricted DNS TCP/53 for trusted LAN clients

✔ Restricted ICMP Echo Request to `eth1` and source `10.10.10.0/24`

✔ Kept DHCP restricted by interface rather than source because an initial
DHCP client may legitimately use source address `0.0.0.0`

### Packet-analysis experiments completed

✔ Installed and used Nmap from Alpine-Lab-02

✔ Demonstrated `open`, `closed`, and `filtered` TCP port states

✔ Compared silent `DROP` with active `REJECT --reject-with tcp-reset`

✔ Observed normal kernel TCP retransmissions with `nc`

✔ Observed raw SYN scanning behavior with `nmap -sS`

✔ Investigated strict reverse-path filtering with `rp_filter=1`

✔ Distinguished routing plausibility (`rp_filter`) from firewall authorization
(`iptables`)

✔ Verified NAT/MASQUERADE and conntrack behavior using packet counters

### Reboot and recovery verification completed

After the final firewall was saved, Alpine-Lab-01 was rebooted.

Verified after reboot:

```text
eth0                         192.168.122.252/24
eth1                         10.10.10.1/24
default route                via 192.168.122.1
net.ipv4.ip_forward          1

iptables                     started
dnsmasq                      started
syslog                       started
klogd                        started

TCP/22                       sshd
UDP/53                       dnsmasq
TCP/53                       dnsmasq
UDP/67                       dnsmasq
```

Alpine-Lab-02 end-to-end tests after the reboot succeeded:

```text
10.10.10.2/24                         OK
default via 10.10.10.1                OK
nameserver 10.10.10.1                 OK
search lab.local                      OK

ping 10.10.10.1                       OK
nslookup alpine-lab-01.lab.local      OK
ping 8.8.8.8                          OK
ping google.com                       OK
```

A final forbidden connection:

```bash
nc -vz -w 2 10.10.10.1 9999
```

timed out as expected and generated post-reboot log entries such as:

```text
IPTABLES-DROP: IN=eth1 SRC=10.10.10.2 DST=10.10.10.1
PROTO=TCP SPT=35307 DPT=9999 SYN
```

This verified the complete path from firewall decision to kernel logging,
`klogd`, `syslogd`, and `/var/log/messages`.

------------------------------------------------------------------------

# Client Health**

**## Alpine-Lab-02**

``` text

IP:       10.10.10.2/24

Gateway:  10.10.10.1

DNS:      10.10.10.1

Domain:   lab.local

```

Verified:

-   ✔ Router connectivity

-   ✔ Internet connectivity through the stateful FORWARD firewall

-   ✔ DNS

-   ✔ SSH by IP

-   ✔ SSH by hostname

**## Alpine-Lab-03**

``` text

IP:       10.10.10.3/24

Gateway:  10.10.10.1

DNS:      10.10.10.1

Domain:   lab.local

```

Verified:

-   ✔ Router connectivity

-   ✔ Internet connectivity through the stateful FORWARD firewall

-   ✔ DNS

-   ✔ SSH from Alpine-Lab-01 by IP

-   ✔ SSH from Alpine-Lab-01 by hostname

------------------------------------------------------------------------

**# Stateful FORWARD Verification**

Before testing both clients:

``` text

RELATED,ESTABLISHED    10

NEW                     2

DROP policy              0

```

After the outbound tests:

``` text

RELATED,ESTABLISHED    30

NEW                     6

DROP policy              0

```

Delta:

``` text

RELATED,ESTABLISHED    +20

NEW                     +4

DROP                     +0

```

This showed that legitimate LAN-initiated traffic was fully handled by

the explicit stateful rules.

The deliberate unsolicited WAN-to-LAN test then produced:

``` text

FORWARD policy DROP     1 packet / 84 bytes

```

This verified both sides of the firewall design:

``` text

LAN -> WAN NEW                    ACCEPT

WAN -> LAN ESTABLISHED/RELATED    ACCEPT

WAN -> LAN unsolicited NEW        DROP

```

------------------------------------------------------------------------

**# Next Goal

**Phase 5 --- Firewall & Security is technically complete.**

No additional firewall rules should be added before preserving this known-good
state.

Immediate objectives:

1. Update the remaining repository documentation.
2. Review the final changes with `git status` and `git diff`.
3. Commit and push the completed Phase 5 documentation.
4. Shut down the VMs cleanly.
5. Create final VM snapshots.
6. Create the corresponding Linux Mint / VirtualBox snapshot.
7. Begin Phase 6 only after the checkpoint is safely preserved.

Port forwarding / DNAT is deferred until a later networking phase where an
internal service exists that is worth publishing deliberately.

------------------------------------------------------------------------

# Latest Snapshots**

Previous Phase 5 INPUT firewall checkpoint snapshots were created after

the INPUT firewall documentation and Git checkpoint.

A new snapshot checkpoint should be created after the current stateful

FORWARD documentation is committed and pushed.

------------------------------------------------------------------------

**# Repository Status

Current branch:

```text
main
```

Current documentation milestone:

```text
Phase 5 --- Firewall & Security complete
```

Files being updated for the final Phase 5 checkpoint:

```text
docs/10-firewall-security.md
README.md
LAB_STATUS.md
CHANGELOG.md
```

Current repository workflow:

```text
Documentation update
        ↓
git status / git diff
        ↓
Git commit
        ↓
Push main
        ↓
Clean VM shutdown
        ↓
VM snapshots
        ↓
Linux Mint / VirtualBox snapshot
        ↓
Phase 6
```

The technical firewall configuration should remain unchanged while this final
checkpoint is documented and preserved.
