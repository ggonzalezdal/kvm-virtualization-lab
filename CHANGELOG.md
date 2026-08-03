# Changelog

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
    -   `08-network-services-dhcp-dns.md`
-   Updated:
    -   `README.md`
    -   `LAB_STATUS.md`
    -   `CHANGELOG.md`

------------------------------------------------------------------------

## Next Phase -- Firewall & Security

Planned topics:

-   Linux Netfilter architecture
-   iptables fundamentals
-   Stateful firewalling
-   Router hardening
-   Packet filtering
-   Logging
-   Port forwarding
