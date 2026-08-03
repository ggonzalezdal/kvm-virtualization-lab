# 09 -- Network Services: DHCP & DNS with dnsmasq

## Objectives

In this phase the lab evolved from a routed network into a small
infrastructure network. Alpine-Lab-01 now provides:

-   Router
-   NAT Gateway
-   DHCP Server
-   DNS Server
-   Local hostname resolution

------------------------------------------------------------------------

# Final Topology

``` text
                    Internet
                        |
                 Linux Mint Host
                        |
              libvirt default NAT
                 192.168.122.1
                        |
                 Alpine-Lab-01
         Router + NAT + DHCP + DNS
          eth0 : DHCP (upstream)
          eth1 : 10.10.10.1/24
                        |
        --------------------------------
        |                              |
  Alpine-Lab-02                 Alpine-Lab-03
      DHCP                           DHCP
      10.10.10.2                     10.10.10.3
```

------------------------------------------------------------------------

# Installing dnsmasq

``` bash
apk add dnsmasq
rc-service dnsmasq start
rc-update add dnsmasq default
```

------------------------------------------------------------------------

# DHCP Configuration

Main configuration was finally moved to:

``` text
/etc/dnsmasq.d/lab.conf
```

Key settings:

``` ini
interface=eth1
except-interface=eth0

dhcp-range=10.10.10.100,10.10.10.200,255.255.255.0,7d

dhcp-host=52:54:00:6a:2f:b0,10.10.10.2
dhcp-host=52:54:00:c4:a2:51,10.10.10.3

dhcp-option=option:router,10.10.10.1
dhcp-option=option:dns-server,10.10.10.1
dhcp-option=option:domain-name,lab.local
```

Clients automatically receive:

-   IP address
-   Subnet mask
-   Default gateway
-   DNS server
-   DNS search domain

------------------------------------------------------------------------

# DNS Configuration

``` ini
domain=lab.local
local=/lab.local/
expand-hosts

address=/alpine-lab-01/10.10.10.1

no-resolv
server=192.168.122.1
```

Meaning:

-   `domain=lab.local` defines the local DNS domain.
-   `local=/lab.local/` makes dnsmasq authoritative for the lab domain.
-   `expand-hosts` appends the domain to local hostnames.
-   `address=` creates a static DNS record for the router.
-   `no-resolv` ignores `/etc/resolv.conf` for upstream DNS.
-   `server=` defines the upstream DNS forwarder.

------------------------------------------------------------------------

# Alpine-Lab-01 as a DNS Client

`/etc/resolv.conf`

``` text
search lab.local
nameserver 10.10.10.1
```

Now every application queries the local dnsmasq instance first.

------------------------------------------------------------------------

# Preventing DHCP from Overwriting DNS

Edited:

``` text
/etc/udhcpc/udhcpc.conf
```

``` ini
NO_DNS="eth0"
```

The DHCP client still accepts:

-   IP address
-   Default gateway

but no longer overwrites `/etc/resolv.conf`.

------------------------------------------------------------------------

# Modular Configuration

Main file:

``` text
/etc/dnsmasq.conf
```

contains only:

``` ini
local-service
conf-dir=/etc/dnsmasq.d/,*.conf
```

Lab configuration resides in:

``` text
/etc/dnsmasq.d/lab.conf
```

This follows common Linux administration practice and keeps vendor
configuration separate from administrator configuration.

------------------------------------------------------------------------

# Verification Commands

``` bash
rc-service dnsmasq status

nslookup alpine-lab-02
nslookup google.com

ping alpine-lab-02
ping alpine-lab-03

ssh airgon@alpine-lab-02

cat /var/lib/misc/dnsmasq.leases

cat /etc/resolv.conf
```

------------------------------------------------------------------------

# Troubleshooting Highlights

During implementation we verified each layer individually:

1.  Confirmed DHCP leases.
2.  Confirmed DNS resolution with `nslookup`.
3.  Confirmed resolver behaviour using `getent`.
4.  Configured Alpine-Lab-01 to use its own DNS server.
5.  Prevented DHCP from overwriting resolver configuration.
6.  Added the `lab.local` search domain.
7.  Refactored configuration into `/etc/dnsmasq.d/`.

This systematic approach isolated the real causes instead of applying
trial-and-error fixes.

------------------------------------------------------------------------

# Lessons Learned

-   Difference between DHCP and DNS.
-   How `dnsmasq` integrates both services.
-   Role of `/etc/resolv.conf`.
-   Purpose of `udhcpc`.
-   DNS forwarding versus authoritative DNS.
-   DNS search domains.
-   Modular Linux configuration using `*.d` directories.
-   Importance of verifying each networking layer independently.

------------------------------------------------------------------------

# Phase Status

**Phase 4 -- Network Services: COMPLETE**

The lab now provides:

-   Routing
-   NAT
-   DHCP
-   DNS
-   Automatic hostname resolution
-   Centralized network services

The next phase focuses on **Firewall & Security** using `iptables` and
Linux Netfilter.
