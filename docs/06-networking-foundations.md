# 06 - Networking Foundations

## Overview

This document summarizes the networking concepts explored during Phase 3
of the KVM Virtualization Lab.

Rather than simply configuring virtual machines, the objective of this
phase was to understand how packets actually travel through a network by
observing them in real time.

The experiments were performed using:

-   Linux Mint 22.2 (KVM host)
-   libvirt default NAT network (`virbr0`)
-   Alpine-Lab-01
-   tcpdump
-   ping
-   ssh
-   iproute2 tools

------------------------------------------------------------------------

# Lab Topology

``` text
                     Internet
                         │
                  Home Router
                  192.168.1.1
                         │
                  Linux Mint Host
                192.168.1.63
                         │
               enp0s3 (physical NIC)
                         │
                 Linux Routing + NAT
                         │
                     virbr0
                192.168.122.1/24
                         │
        ┌────────────┬────────────┬────────────┐
        │            │            │
 Alpine-01      Alpine-02    Alpine-03
192.168.122.252  .137          .161
```

------------------------------------------------------------------------

# Concepts Learned

## Virtual NICs

Each virtual machine owns a virtual network interface card (vNIC).

From the guest operating system, the interface behaves exactly like a
physical Ethernet adapter.

Example:

``` text
eth0
```

------------------------------------------------------------------------

## Linux Bridge (virbr0)

The libvirt default network creates a Linux bridge called `virbr0`.

The bridge behaves similarly to an Ethernet switch. Every VM connected
to the default network exchanges Ethernet frames through this bridge.

------------------------------------------------------------------------

## DHCP

The default libvirt network provides DHCP.

Example leases:

-   192.168.122.252
-   192.168.122.137
-   192.168.122.161

Verified with:

``` bash
virsh net-dhcp-leases default
```

------------------------------------------------------------------------

# TCP

TCP is a connection-oriented protocol.

## TCP Three-Way Handshake

Observed using:

``` bash
sudo tcpdump -i eth0 tcp port 22
```

Captured packets:

``` text
[S]
[S.]
[.]
```

Meaning:

``` text
Client                Server

SYN  --------------------->

      <------------------- SYN + ACK

ACK  --------------------->
```

Only after this does SSH begin.

## TCP Flags

  Flag   Meaning
  ------ -----------
  S      SYN
  .      ACK
  S.     SYN + ACK
  P.     PSH + ACK
  F      FIN
  R      RST

## Sequence Numbers

`seq 1:44` means bytes 1 through 43 are carried in that packet.

## ACK Numbers

`ack 44` means:

> I have successfully received bytes 1 through 43. Send byte 44 next.

------------------------------------------------------------------------

# SSH

SSH is an application protocol that runs on top of TCP.

``` text
SSH
 │
TCP
 │
IP
```

------------------------------------------------------------------------

# ARP

ARP translates IP addresses into MAC addresses.

Request:

``` text
Who has 192.168.122.252?
```

Reply:

``` text
192.168.122.252 is-at 52:54:00:88:90:ca
```

View the ARP cache:

``` bash
ip neigh
```

------------------------------------------------------------------------

# Routing

Display the routing table:

``` bash
ip route
```

Ask Linux how it would reach a destination:

``` bash
ip route get 192.168.122.252
ip route get 8.8.8.8
```

Local destinations are sent directly. Remote destinations are forwarded
to the default gateway.

------------------------------------------------------------------------

# ICMP

Observed with:

``` bash
sudo tcpdump -i eth0 icmp
```

ICMP Echo Request and Echo Reply are the packets used by `ping`.

------------------------------------------------------------------------

# NAT

One of the key experiments.

Inside the virtual network:

``` text
192.168.122.252 → 8.8.8.8
```

Leaving Linux Mint:

``` text
192.168.1.63 → 8.8.8.8
```

Linux Mint performs Source NAT before forwarding traffic to the
Internet, then reverses the translation for returning packets.

------------------------------------------------------------------------

# Complete Packet Journey

``` text
SSH
 ↓
TCP
 ↓
IP
 ↓
Routing Table
 ↓
ARP
 ↓
Ethernet
 ↓
Linux Bridge
 ↓
NAT
 ↓
Home Router
 ↓
Internet
```

------------------------------------------------------------------------

# Useful Commands

``` bash
ip addr
ip route
ip route get <IP>
ip neigh
ss -tuln
ss -tn
sudo tcpdump -i eth0 tcp port 22
sudo tcpdump -i eth0 icmp
sudo tcpdump -i eth0 arp
```

------------------------------------------------------------------------

# Key Takeaways

-   Ethernet uses MAC addresses.
-   IP uses IP addresses.
-   ARP resolves IP → MAC.
-   Routing determines the next hop.
-   TCP provides reliable connections.
-   SSH runs on top of TCP.
-   ICMP powers `ping`.
-   Linux Mint acts as the router and NAT gateway for the libvirt
    network.

------------------------------------------------------------------------

# Current Status

Completed:

-   Phase 1 -- KVM Fundamentals
-   Phase 2 -- VM Management
-   Phase 3 -- Networking Foundations

Next:

-   Multiple libvirt networks
-   Routing between networks
-   Firewalls
-   DNS
-   DHCP
-   Production-style virtual infrastructure
