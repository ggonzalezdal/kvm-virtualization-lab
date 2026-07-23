# 07 - Building an Isolated LAN

## Objective

In this phase, we expanded the virtual lab by creating a second,
isolated virtual network in libvirt and connecting all three Alpine
virtual machines to it.

The goal was to build a realistic multi-network environment that will
later allow Alpine-Lab-01 to act as a Linux router between two different
networks.

------------------------------------------------------------------------

# Lab Architecture

## Before

``` text
                    Linux Mint
                KVM / libvirt Host
                        │
                   virbr0 (NAT)
               192.168.122.0/24
                        │
        ┌───────────────┼───────────────┐
        │               │               │
   Alpine-01       Alpine-02       Alpine-03
```

## After

``` text
                           Linux Mint
                     KVM / libvirt Host
                               │
        ┌──────────────────────┴──────────────────────┐
        │                                             │
        │                                             │
   virbr0 (default NAT)                     virbr10 (lab-isolated)
  192.168.122.0/24                          10.10.10.0/24
        │                                             │
   ┌────┼───────────────┐                  ┌──────────┼──────────────┐
   │    │               │                  │          │              │
  eth0 eth0            eth0              eth1       eth1           eth1
   │    │               │                  │          │              │
Alpine-01          Alpine-02         Alpine-01   Alpine-02    Alpine-03
192.168.122.252    192.168.122.137    10.10.10.1 10.10.10.2   10.10.10.3
Alpine-03
192.168.122.161
```

Each VM now has:

  VM              eth0              eth1
  --------------- ----------------- ------------
  Alpine-Lab-01   192.168.122.252   10.10.10.1
  Alpine-Lab-02   192.168.122.137   10.10.10.2
  Alpine-Lab-03   192.168.122.161   10.10.10.3

## Creating the Isolated Network

Created `lab-isolated.xml`:

``` xml
<network>
  <name>lab-isolated</name>
  <bridge name="virbr10" stp="on" delay="0"/>
  <ip address="10.10.10.254" netmask="255.255.255.0">
  </ip>
</network>
```

Network properties:

-   Network name: `lab-isolated`
-   Bridge: `virbr10`
-   Subnet: `10.10.10.0/24`
-   Bridge IP: `10.10.10.254`
-   DHCP: Disabled
-   NAT: Disabled

## libvirt Commands

``` bash
virsh net-define lab-isolated.xml
virsh net-start lab-isolated
virsh net-autostart lab-isolated
```

## Adding Secondary NICs

Each VM received a second VirtIO interface:

``` bash
virsh attach-interface \
  --domain <VM> \
  --type network \
  --source lab-isolated \
  --model virtio \
  --config
```

`--config` modifies the persistent VM definition.

## Temporary Network Configuration

Alpine-Lab-01

``` bash
sudo ip link set eth1 up
sudo ip addr add 10.10.10.1/24 dev eth1
```

Alpine-Lab-02

``` bash
sudo ip link set eth1 up
sudo ip addr add 10.10.10.2/24 dev eth1
```

Alpine-Lab-03

``` bash
sudo ip link set eth1 up
sudo ip addr add 10.10.10.3/24 dev eth1
```

These settings are temporary and disappear after reboot.

## Routing

Example routing table from Alpine-Lab-01:

``` text
default via 192.168.122.1 dev eth0
10.10.10.0/24 dev eth1 proto kernel scope link src 10.10.10.1
192.168.122.0/24 dev eth0 proto kernel scope link src 192.168.122.252
```

The kernel automatically created the connected routes.

## ARP Observations

Useful capture filter:

``` bash
sudo tcpdump -n -i eth1 'arp or icmp'
```

First ping:

-   ARP Request
-   ARP Reply
-   ICMP Echo Request
-   ICMP Echo Reply

Subsequent pings:

-   ICMP Echo Request
-   ICMP Echo Reply

Reason: ARP cache.

## Key Concepts Learned

-   Linux bridges
-   Virtual Ethernet switches
-   Multi-homed Linux hosts
-   VirtIO network adapters
-   Temporary IP configuration
-   Automatic connected routes
-   ARP cache
-   Broadcast vs unicast
-   Relationship between host `vnet` interfaces and guest `eth`
    interfaces

## Next Phase

-   Make networking persistent
-   Enable IPv4 forwarding
-   Configure NAT
-   Turn Alpine-Lab-01 into a Linux router
-   Route Alpine-Lab-02 and Alpine-Lab-03 through Alpine-Lab-01
