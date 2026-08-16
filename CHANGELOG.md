k# Changelog

All notable changes to this laboratory are documented here in
chronological order.

------------------------------------------------------------------------

## Phase 1 -- KVM Fundamentals

-   Installed Linux Mint KVM environment.
-   Installed Alpine Linux.
-   Learned `virsh` fundamentals.
-   Created baseline snapshots.
-   Established the initial Git repository structure.

------------------------------------------------------------------------

## Phase 2 -- Virtual Machine Management

-   Configured SSH key authentication.
-   Learned manual VM cloning.
-   Learned `virt-clone`.
-   Explored and edited libvirt XML definitions.
-   Implemented a VM snapshot strategy.
-   Documented cloning procedures.

------------------------------------------------------------------------

## Phase 3 -- Networking Foundations

-   Built a custom isolated virtual network.
-   Converted Alpine-Lab-01 into a Linux router.
-   Enabled persistent IP forwarding.
-   Configured NAT using `iptables`.
-   Verified Internet connectivity for internal clients.
-   Implemented static addressing.
-   Documented networking experiments and packet analysis.

------------------------------------------------------------------------

## Phase 4 -- Network Services

### DHCP

-   Installed and configured `dnsmasq`.
-   Configured DHCP for the isolated network.
-   Added static DHCP reservations for Alpine-Lab-02 and Alpine-Lab-03.
-   Distributed gateway, DNS server and search domain via DHCP.

### DNS

-   Implemented a local DNS server.
-   Configured DNS forwarding.
-   Created the local `lab.local` DNS zone.
-   Enabled automatic hostname resolution.
-   Configured Alpine-Lab-01 to use its own DNS service.

### System Administration

-   Prevented DHCP from overwriting `/etc/resolv.conf`.
-   Modularized the `dnsmasq` configuration using
    `/etc/dnsmasq.d/lab.conf`.
-   Refactored the service configuration following Linux drop-in
    configuration best practices.

### Verification

-   Verified DHCP lease allocation.
-   Verified DNS forwarding.
-   Verified local hostname resolution.
-   Verified SSH connectivity using hostnames.
-   Completed end-to-end network service validation.

### Documentation

-   Added:
    -   `09-network-services-dhcp-dns.md`
-   Updated:
    -   `README.md`
    -   `LAB_STATUS.md`
    -   `CHANGELOG.md`

------------------------------------------------------------------------

## Phase 5 -- Firewall & Security

### Netfilter and iptables

-   Introduced the Linux Netfilter packet-filtering architecture.
-   Studied the roles of the `INPUT`, `OUTPUT`, and `FORWARD` chains.
-   Configured Alpine-Lab-01 with a default `DROP` policy on `INPUT`.
-   Kept `OUTPUT` and `FORWARD` at `ACCEPT` during the first firewall
    checkpoint.
-   Introduced stateful packet filtering using `conntrack`.
-   Studied the `NEW`, `ESTABLISHED`, and `RELATED` connection states.

### Router INPUT Hardening

-   Permitted loopback traffic explicitly.
-   Permitted `RELATED,ESTABLISHED` traffic.
-   Permitted new SSH connections on TCP port 22.
-   Permitted DHCP requests from the isolated LAN on UDP port 67.
-   Permitted DNS requests from the isolated LAN on UDP port 53.
-   Permitted ICMP Echo Requests from the isolated LAN.
-   Inspected packet and byte counters to verify firewall rule matches.

Current INPUT policy:

```text
INPUT DROP
│
├── lo                          ACCEPT
├── RELATED,ESTABLISHED         ACCEPT
├── TCP/22 NEW                  ACCEPT   SSH
├── eth1 UDP/67                 ACCEPT   DHCP
├── eth1 UDP/53                 ACCEPT   DNS
└── eth1 ICMP echo-request      ACCEPT   ping
```

### ICMP and Connection Tracking

-   Investigated why Alpine-Lab-02 could reach the Internet while ping
    to the router itself initially failed.
-   Demonstrated the difference between traffic entering `INPUT` and
    traffic traversing `FORWARD`.
-   Performed a temporary ICMP conntrack experiment.
-   Observed the first Echo Request as `NEW` and subsequent Echo
    Requests as `ESTABLISHED`.
-   Removed the temporary diagnostic rules after the experiment.

### DHCP and DNS Firewall Troubleshooting

-   Diagnosed DHCP failure caused by the new default-deny INPUT policy.
-   Confirmed DHCP requests reaching `eth1`.
-   Added the required UDP/67 INPUT rule.
-   Added the required UDP/53 INPUT rule for DNS clients.
-   Diagnosed local DNS timeouts on Alpine-Lab-01 using:

    ```bash
    sudo tcpdump -ni lo 'port 53'
    ```

-   Confirmed that Alpine-Lab-01's own DNS queries traversed the
    loopback interface.
-   Added an explicit loopback INPUT rule:

    ```bash
    -A INPUT -i lo -j ACCEPT
    ```

### DNS Hostname Correction

-   Diagnosed `alpine-lab-01` incorrectly resolving to `127.0.0.1`
    from Alpine-Lab-02.
-   Traced the incorrect address to Alpine-Lab-01's `/etc/hosts`
    combined with dnsmasq hostname expansion.
-   Corrected Alpine-Lab-01 so its LAN hostname resolves to
    `10.10.10.1`.
-   Verified:

    ```text
    alpine-lab-01 -> 10.10.10.1
    alpine-lab-02 -> 10.10.10.2
    ```

### SSH Host Identity Troubleshooting

-   Encountered an SSH `REMOTE HOST IDENTIFICATION HAS CHANGED`
    warning after correcting DNS.
-   Removed the obsolete hostname association from `known_hosts`.
-   Verified Alpine-Lab-01's actual ED25519 host-key fingerprint
    directly on the server before accepting it.
-   Confirmed the fingerprint:

    ```text
    SHA256:RgJxw2RXzvc2VMLjZrrxyfGCsH8ZQvBVtXsQIhCHV88
    ```

-   Verified successful SSH access to Alpine-Lab-01 by hostname.
-   Verified SSH access from Alpine-Lab-01 to Alpine-Lab-02 and
    Alpine-Lab-03 by IP and hostname.

### Client Verification

Alpine-Lab-02 verified with:

-   `10.10.10.2/24`
-   Default gateway `10.10.10.1`
-   DNS server `10.10.10.1`
-   Router connectivity
-   Internet connectivity
-   Local DNS
-   SSH by IP and hostname

Alpine-Lab-03 verified with:

-   `10.10.10.3/24`
-   Default gateway `10.10.10.1`
-   DNS server `10.10.10.1`
-   Router connectivity
-   Internet connectivity
-   Local DNS
-   SSH connectivity

### Firewall Persistence

-   Saved the final live firewall using:

    ```bash
    sudo rc-service iptables save
    ```

-   Verified persistent rules in:

    ```text
    /etc/iptables/rules-save
    ```

-   Confirmed the existing NAT/MASQUERADE rule remained intact:

    ```bash
    -A POSTROUTING -s 10.10.10.0/24 -o eth0 -j MASQUERADE
    ```

### Documentation

-   Added:
    -   `10-firewall-security.md`
-   Updated:
    -   `README.md`
    -   `LAB_STATUS.md`
    -   `CHANGELOG.md`

### Current Checkpoint

Phase 5 is **in progress**.

Completed:

-   Stateful INPUT firewall
-   Router service allowances
-   Loopback handling
-   DHCP/DNS firewall integration
-   ICMP filtering
-   Firewall persistence
-   DNS and SSH troubleshooting
-   Client verification

Current forwarding policy:

```text
FORWARD ACCEPT
```

The next stage will implement explicit stateful forwarding rules before
changing the default `FORWARD` policy to `DROP`.

------------------------------------------------------------------------

## Next -- Phase 5 Stateful FORWARD Filtering

Next objectives:

-   Understand stateful filtering in the `FORWARD` chain.
-   Permit new connections from the isolated LAN toward external
    networks.
-   Permit `ESTABLISHED,RELATED` return traffic.
-   Block unsolicited new connections entering the isolated LAN.
-   Change the default `FORWARD` policy from `ACCEPT` to `DROP`.
-   Verify the policy using ping, DNS, SSH, packet counters and packet
    capture.
-   Continue with firewall logging and additional hardening.

