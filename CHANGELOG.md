**# Changelog**

All notable changes to this laboratory are documented here in

chronological order.

------------------------------------------------------------------------

**## Phase 1 -- KVM Fundamentals**

-   Installed Linux Mint KVM environment.

-   Installed Alpine Linux.

-   Learned `virsh` fundamentals.

-   Created baseline snapshots.

-   Established the initial Git repository structure.

------------------------------------------------------------------------

**## Phase 2 -- Virtual Machine Management**

-   Configured SSH key authentication.

-   Learned manual VM cloning.

-   Learned `virt-clone`.

-   Explored and edited libvirt XML definitions.

-   Implemented a VM snapshot strategy.

-   Documented cloning procedures.

------------------------------------------------------------------------

**## Phase 3 -- Networking Foundations**

-   Built a custom isolated virtual network.

-   Converted Alpine-Lab-01 into a Linux router.

-   Enabled persistent IP forwarding.

-   Configured NAT using `iptables`.

-   Verified Internet connectivity for internal clients.

-   Implemented static addressing.

-   Documented networking experiments and packet analysis.

------------------------------------------------------------------------

**## Phase 4 -- Network Services**

**### DHCP**

-   Installed and configured `dnsmasq`.

-   Configured DHCP for the isolated network.

-   Added static DHCP reservations for Alpine-Lab-02 and Alpine-Lab-03.

-   Distributed gateway, DNS server and search domain via DHCP.

**### DNS**

-   Implemented a local DNS server.

-   Configured DNS forwarding.

-   Created the local `lab.local` DNS zone.

-   Enabled automatic hostname resolution.

-   Configured Alpine-Lab-01 to use its own DNS service.

**### System Administration**

-   Prevented DHCP from overwriting `/etc/resolv.conf`.

-   Modularized the `dnsmasq` configuration using

    `/etc/dnsmasq.d/lab.conf`.

-   Refactored the service configuration following Linux drop-in

    configuration best practices.

**### Verification**

-   Verified DHCP lease allocation.

-   Verified DNS forwarding.

-   Verified local hostname resolution.

-   Verified SSH connectivity using hostnames.

-   Completed end-to-end network service validation.

**### Documentation**

-   Added:

    -   `09-network-services-dhcp-dns.md`

-   Updated:

    -   `README.md`

    -   `LAB_STATUS.md`

    -   `CHANGELOG.md`

------------------------------------------------------------------------

**## Phase 5 -- Firewall & Security**

**### Netfilter and iptables**

-   Introduced the Linux Netfilter packet-filtering architecture.

-   Studied the roles of the `INPUT`, `OUTPUT`, and `FORWARD` chains.

-   Configured Alpine-Lab-01 with a default `DROP` policy on `INPUT`.

-   Initially kept `OUTPUT` and `FORWARD` at `ACCEPT` while building and

    testing the first firewall checkpoint.

-   Introduced stateful packet filtering using `conntrack`.

-   Studied the `NEW`, `ESTABLISHED`, and `RELATED` connection states.

**### Router INPUT Hardening**

-   Permitted loopback traffic explicitly.

-   Permitted `RELATED,ESTABLISHED` traffic.

-   Permitted new SSH connections on TCP port 22.

-   Permitted DHCP requests from the isolated LAN on UDP port 67.

-   Permitted DNS requests from the isolated LAN on UDP port 53.

-   Permitted ICMP Echo Requests from the isolated LAN.

-   Inspected packet and byte counters to verify firewall rule matches.

Current INPUT policy:

``` text

INPUT DROP

│

├── lo                          ACCEPT

├── RELATED,ESTABLISHED         ACCEPT

├── TCP/22 NEW                  ACCEPT   SSH

├── eth1 UDP/67                 ACCEPT   DHCP

├── eth1 UDP/53                 ACCEPT   DNS

└── eth1 ICMP echo-request      ACCEPT   ping

```

**### ICMP and Connection Tracking**

-   Investigated why Alpine-Lab-02 could reach the Internet while ping

    to the router itself initially failed.

-   Demonstrated the difference between traffic entering `INPUT` and

    traffic traversing `FORWARD`.

-   Performed a temporary ICMP conntrack experiment.

-   Observed the first Echo Request as `NEW` and subsequent Echo

    Requests as `ESTABLISHED`.

-   Removed the temporary diagnostic rules after the experiment.

**### DHCP and DNS Firewall Troubleshooting**

-   Diagnosed DHCP failure caused by the new default-deny INPUT policy.

-   Confirmed DHCP requests reaching `eth1`.

-   Added the required UDP/67 INPUT rule.

-   Added the required UDP/53 INPUT rule for DNS clients.

-   Diagnosed local DNS timeouts on Alpine-Lab-01 using:

    ``` bash

    sudo tcpdump -ni lo 'port 53'

    ```

-   Confirmed that Alpine-Lab-01's own DNS queries traversed the

    loopback interface.

-   Added an explicit loopback INPUT rule:

    ``` bash

    -A INPUT -i lo -j ACCEPT

    ```

**### DNS Hostname Correction**

-   Diagnosed `alpine-lab-01` incorrectly resolving to `127.0.0.1` from

    Alpine-Lab-02.

-   Traced the incorrect address to Alpine-Lab-01's `/etc/hosts`

    combined with dnsmasq hostname expansion.

-   Corrected Alpine-Lab-01 so its LAN hostname resolves to

    `10.10.10.1`.

-   Verified:

    ``` text

    alpine-lab-01 -> 10.10.10.1

    alpine-lab-02 -> 10.10.10.2

    ```

**### SSH Host Identity Troubleshooting**

-   Encountered an SSH `REMOTE HOST IDENTIFICATION HAS CHANGED` warning

    after correcting DNS.

-   Removed the obsolete hostname association from `known_hosts`.

-   Verified Alpine-Lab-01's actual ED25519 host-key fingerprint

    directly on the server before accepting it.

-   Confirmed the fingerprint:

    ``` text

    SHA256\:RgJxw2RXzvc2VMLjZrrxyfGCsH8ZQvBVtXsQIhCHV88

    ```

-   Verified successful SSH access to Alpine-Lab-01 by hostname.

-   Verified SSH access from Alpine-Lab-01 to Alpine-Lab-02 and

    Alpine-Lab-03 by IP and hostname.

**### Client Verification**

Alpine-Lab-02 verified with:

-   `10.10.10.2/24`

-   Default gateway `10.10.10.1`

-   DNS server `10.10.10.1`

-   Router connectivity

-   Internet connectivity

-   Local DNS

-   SSH by IP and hostname

Alpine-Lab-03 verified with:

-   `10.10.10.3/24`

-   Default gateway `10.10.10.1`

-   DNS server `10.10.10.1`

-   Router connectivity

-   Internet connectivity

-   Local DNS

-   SSH connectivity

**### Stateful FORWARD Filtering**

-   Inspected the original `FORWARD` chain while its default policy was

    still `ACCEPT`.

-   Used client traffic and rule counters to confirm that Internet-bound

    traffic from the isolated LAN traverses `FORWARD`.

-   Added the stateful return-traffic rule:

    ``` bash

    -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

    ```

-   Added an explicit rule allowing the isolated LAN to initiate new

    traffic toward the external interface:

    ``` bash

    -A FORWARD -s 10.10.10.0/24 -i eth1 -o eth0 \\

        -m conntrack --ctstate NEW -j ACCEPT

    ```

-   Initially retained `FORWARD ACCEPT` as a temporary safety net while

    verifying the explicit rules.

-   Observed conntrack counters showing the first packet of a tracked

    ICMP flow as `NEW`, with subsequent packets matching

    `RELATED,ESTABLISHED`.

-   Verified outbound connectivity from both Alpine-Lab-02 and

    Alpine-Lab-03 using the explicit rules.

-   Changed the default forwarding policy to:

    ``` text

    FORWARD DROP

    ```

Current forwarding policy:

``` text

FORWARD DROP

│

├── RELATED,ESTABLISHED                         ACCEPT

└── eth1 -> eth0

    source 10.10.10.0/24

    NEW                                         ACCEPT

```

The resulting stateful model is:

``` text

LAN -> WAN NEW                    ACCEPT

WAN -> LAN ESTABLISHED/RELATED    ACCEPT

WAN -> LAN unsolicited NEW        DROP

```

**### DNS and FORWARD Chain Analysis**

-   Confirmed that `nslookup` from a lab client does generate network

    traffic, but the client sends its DNS request to Alpine-Lab-01

    itself at `10.10.10.1:53`.

-   Therefore the client DNS request traverses `INPUT`, not `FORWARD`.

-   When dnsmasq needs an upstream answer, dnsmasq generates the query

    locally and that traffic traverses `OUTPUT`.

-   Reinforced the distinction between traffic destined for the router,

    traffic generated by the router, and traffic routed through it.

**### Routing and Longest-Prefix Matching Experiment**

-   Discovered that Linux Mint already has a directly connected route:

    ``` text

    10.10.10.0/24 dev virbr10

    ```

-   Confirmed that Linux Mint therefore reaches Alpine-Lab-02 directly

    through `virbr10`, normally bypassing Alpine-Lab-01.

-   Determined that adding another `10.10.10.0/24` route through

    Alpine-Lab-01 would not provide a clean forced-path test because a

    directly connected route to the same prefix already exists.

-   Added a temporary, more-specific host route:

    ``` bash

    sudo ip route add 10.10.10.2/32 \\

        via 192.168.122.252 dev virbr0

    ```

-   Demonstrated Linux longest-prefix matching:

    ``` text

    /32 > /24 > /0

    ```

-   Forced traffic for `10.10.10.2` through Alpine-Lab-01 while leaving

    the rest of `10.10.10.0/24` on the directly connected route.

-   Sent a new Linux Mint -\> Alpine-Lab-02 ICMP flow through

    Alpine-Lab-01.

-   Verified that the unsolicited `eth0 -> eth1` NEW flow matched no

    explicit allow rule and reached the `FORWARD DROP` policy.

-   Observed the DROP policy counter increase by:

    ``` text

    1 packet, 84 bytes

    ```

-   Removed the temporary `/32` route after the experiment and restored

    Linux Mint's normal routing table.

**### Stateful FORWARD Verification**

Before the final client tests:

``` text

RELATED,ESTABLISHED    10

NEW                     2

DROP policy              0

```

After normal outbound tests from the lab clients:

``` text

RELATED,ESTABLISHED    30

NEW                     6

DROP policy              0

```

This confirmed that legitimate LAN-initiated traffic was handled

entirely by the explicit stateful rules.

The deliberate unsolicited external-to-LAN test then produced:

``` text

FORWARD policy DROP     1 packet / 84 bytes

```

This verified both the permitted and denied directions of the forwarding

policy.

**### Firewall Persistence**

-   Saved the completed live firewall using:

    ``` bash

    sudo rc-service iptables save

    ```

-   Verified persistent rules in:

    ``` text

    /etc/iptables/rules-save

    ```

-   Confirmed the saved filter policies:

    ``` text

    INPUT    DROP

    FORWARD  DROP

    OUTPUT   ACCEPT

    ```

-   Confirmed both stateful FORWARD rules were persisted.

-   Confirmed the existing NAT/MASQUERADE rule remained intact:

    ``` bash

    -A POSTROUTING -s 10.10.10.0/24 -o eth0 -j MASQUERADE

    ```

**

### Firewall Logging

- Added rate-limited logging for packets reaching the end of the `INPUT`
  chain:

  ```bash
  -A INPUT -m limit --limit 5/min --limit-burst 10 \
      -j LOG --log-prefix "IPTABLES-DROP: "
  ```

- Added equivalent rate-limited logging to `FORWARD`:

  ```bash
  -A FORWARD -m limit --limit 5/min --limit-burst 10 \
      -j LOG --log-prefix "FORWARD-DROP: "
  ```

- Verified BusyBox `syslogd` and `klogd`.

- Kept firewall messages in `/var/log/messages`.

- Generated deliberate denied traffic and confirmed `IPTABLES-DROP:` entries.

- Confirmed repeated `nc` SYN packets used the same source port because they
  were retransmissions from the same TCP connection attempt.

### Nmap and DROP vs REJECT

- Installed Nmap on Alpine-Lab-02.

- Used SYN scanning to distinguish:

  ```text
  open
  closed
  filtered
  ```

- Verified SSH TCP/22 as `open`.

- Temporarily allowed TCP/8888 with no listening service and observed it as
  `closed` because the TCP stack returned RST.

- Left TCP/9999 under silent `DROP` and observed it as `filtered`.

- Temporarily tested:

  ```bash
  REJECT --reject-with tcp-reset
  ```

  on TCP/9999 and observed the port become immediately `closed`.

- Removed all temporary test rules afterward.

- Compared normal kernel-managed TCP connection attempts from `nc` with raw
  SYN scanning from `nmap -sS`.

### Service Exposure and INPUT Hardening

- Restricted SSH to the trusted LAN:

  ```bash
  -A INPUT -s 10.10.10.0/24 -i eth1 -p tcp \
      --dport 22 -m conntrack --ctstate NEW -j ACCEPT
  ```

- Restricted DNS UDP/53 to `eth1` and source `10.10.10.0/24`.

- Added DNS TCP/53 support and restricted it to the trusted LAN:

  ```bash
  -A INPUT -s 10.10.10.0/24 -i eth1 -p tcp \
      --dport 53 -m conntrack --ctstate NEW -j ACCEPT
  ```

- Verified UDP/53 and TCP/53 with Nmap.

- Restricted ICMP Echo Request to `eth1` and source `10.10.10.0/24`.

- Intentionally kept DHCP restricted by interface rather than source address,
  because a client may initially use source `0.0.0.0` before receiving its
  lease.

### Reverse-Path Filtering Experiment

- Temporarily assigned `10.20.20.2/24` to Alpine-Lab-02 to test the hardened
  ICMP source rule.

- Discovered strict reverse-path filtering on Alpine-Lab-01:

  ```text
  net.ipv4.conf.eth1.rp_filter = 1
  net.ipv4.conf.all.rp_filter = 1
  ```

- Observed that the unexpected source was initially rejected by `rp_filter`
  before reaching the normal `INPUT` firewall path.

- Added a temporary route:

  ```bash
  sudo ip route add 10.20.20.0/24 dev eth1
  ```

  so the source became routing-plausible.

- Retested and observed `IPTABLES-DROP:` entries for source `10.20.20.2`,
  proving that the hardened firewall source restriction independently denied
  the packet.

- Established the distinction:

  ```text
  rp_filter   -> Is the source plausible according to routing?
  iptables    -> Is the source authorized by security policy?
  ```

- Removed the temporary address and route after testing.

### FORWARD Logging Verification

- Repeated the controlled Linux Mint `/32` route technique to force
  unsolicited traffic through Alpine-Lab-01.

- Sent three Echo Requests from the upstream side toward Alpine-Lab-02.

- Verified:

  ```text
  FORWARD-DROP log     +3
  FORWARD policy DROP  +3
  ```

- Confirmed log entries with:

  ```text
  IN=eth0 OUT=eth1
  SRC=192.168.122.1
  DST=10.10.10.2
  ```

- Removed the temporary Linux Mint route after the experiment.

### NAT / MASQUERADE and conntrack

- Revisited the existing rule:

  ```bash
  -A POSTROUTING -s 10.10.10.0/24 -o eth0 -j MASQUERADE
  ```

- Confirmed that Alpine-Lab-01 translates internal client traffic to its
  `eth0` address `192.168.122.252`.

- Used a three-ping experiment to compare filter and NAT counters:

  ```text
  FORWARD NEW                  +1
  FORWARD RELATED,ESTABLISHED  +5
  MASQUERADE                   +1
  ```

- Reinforced that filtering evaluates packets throughout a tracked flow,
  whereas NAT establishes the translation for the flow and conntrack reuses
  that mapping.

### Final Firewall Persistence

- Saved the completed firewall again with:

  ```bash
  sudo rc-service iptables save
  ```

- Final saved state dated:

  ```text
  Thu Aug 27 13:42:40 2026
  ```

- Confirmed persisted policies:

  ```text
  INPUT    DROP
  FORWARD  DROP
  OUTPUT   ACCEPT
  ```

- Confirmed persisted INPUT service restrictions, INPUT logging, FORWARD
  stateful rules, FORWARD logging, and NAT/MASQUERADE.

### Full Reboot and Recovery Verification

- Rebooted Alpine-Lab-01 after saving the final configuration.

- Verified persistent router identity:

  ```text
  eth0    192.168.122.252/24
  eth1    10.10.10.1/24
  ```

- Verified routing and forwarding:

  ```text
  default via 192.168.122.1 dev eth0
  10.10.10.0/24 dev eth1
  192.168.122.0/24 dev eth0

  net.ipv4.ip_forward = 1
  ```

- Verified automatic service recovery:

  ```text
  iptables   started
  dnsmasq    started
  syslog     started
  klogd      started
  ```

- Verified listening services:

  ```text
  TCP/22     sshd
  UDP/53     dnsmasq
  TCP/53     dnsmasq
  UDP/67     dnsmasq
  ```

- Verified Alpine-Lab-02 retained:

  ```text
  10.10.10.2/24
  default via 10.10.10.1
  nameserver 10.10.10.1
  search lab.local
  ```

- Successfully tested:

  ```bash
  ping -c 2 10.10.10.1
  nslookup alpine-lab-01.lab.local
  ping -c 3 8.8.8.8
  ping -c 2 google.com
  ```

- Generated a final forbidden TCP connection:

  ```bash
  nc -vz -w 2 10.10.10.1 9999
  ```

  and confirmed post-reboot logging:

  ```text
  IPTABLES-DROP: IN=eth1 SRC=10.10.10.2 DST=10.10.10.1
  PROTO=TCP SPT=35307 DPT=9999 SYN
  ```

- This verified the complete firewall-to-kernel-to-logging path after reboot.

### Documentation

- Finalized and expanded:

  - `docs/10-firewall-security.md`

- Updated:

  - `README.md`
  - `LAB_STATUS.md`
  - `CHANGELOG.md`

### Phase 5 Complete

Phase 5 is **complete**.

Final capabilities:

- Stateful default-deny INPUT firewall
- Stateful default-deny FORWARD firewall
- Explicit trusted-LAN service exposure
- SSH source/interface hardening
- DHCP firewall integration
- DNS over UDP and TCP
- ICMP source/interface hardening
- Rate-limited INPUT DROP logging
- Rate-limited FORWARD DROP logging
- Nmap-based firewall verification
- DROP vs REJECT testing
- Reverse-path filtering analysis
- Stateful LAN-to-WAN forwarding
- Unsolicited upstream-to-LAN blocking
- NAT/MASQUERADE with conntrack
- Firewall persistence
- Router/service persistence
- Full reboot and recovery verification

Final filter policies:

```text
INPUT    DROP
FORWARD  DROP
OUTPUT   ACCEPT
```

Final forwarding model:

```text
LAN -> WAN NEW                     ACCEPT
WAN -> LAN ESTABLISHED/RELATED     ACCEPT
WAN -> LAN unsolicited NEW         DROP
```

------------------------------------------------------------------------

## Next -- Phase 6

Before beginning Phase 6:

- Review the final repository changes with `git status` and `git diff`.
- Commit and push the completed Phase 5 documentation.
- Shut down the VMs cleanly.
- Create the final Phase 5 VM snapshots.
- Create the corresponding Linux Mint / VirtualBox snapshot.

Port forwarding / DNAT is deferred until a later networking phase where an
internal service exists that can be published deliberately.
