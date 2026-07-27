# 08 - Persistent Linux Router

## Objective

Transform **Alpine-Lab-01** into a persistent Linux router that provides
Internet access to an isolated LAN composed of Alpine-Lab-02 and
Alpine-Lab-03.

------------------------------------------------------------------------

## Final Topology

``` text
                    Internet
                        |
             libvirt NAT (192.168.122.1)
                        |
                Alpine-Lab-01
      eth0: 192.168.122.252 (DHCP)
      eth1: 10.10.10.1/24
                        |
          -----------------------------
          |                           |
    Alpine-Lab-02               Alpine-Lab-03
    10.10.10.2/24               10.10.10.3/24
          GW: 10.10.10.1        GW: 10.10.10.1
```

------------------------------------------------------------------------

## Tasks Completed

### 1. Persistent interface configuration

**Alpine-Lab-01**

`/etc/network/interfaces`

``` text
auto lo
iface lo inet loopback
iface lo inet6 loopback

auto eth0
iface eth0 inet dhcp
iface eth0 inet6 auto

auto eth1
iface eth1 inet static
    address 10.10.10.1
    netmask 255.255.255.0
```

------------------------------------------------------------------------

### 2. Persistent IPv4 forwarding

Created:

``` text
/etc/sysctl.d/99-router.conf
```

Contents:

``` text
# KVM Lab Router
net.ipv4.ip_forward = 1
```

Verified:

-   OpenRC `sysctl` service enabled at boot.
-   `/etc/sysctl.d/*.conf` processed automatically.
-   `cat /proc/sys/net/ipv4/ip_forward` returned `1`.

------------------------------------------------------------------------

### 3. Persistent NAT

Created NAT rule:

``` bash
iptables -t nat -A POSTROUTING \
    -s 10.10.10.0/24 \
    -o eth0 \
    -j MASQUERADE
```

Saved permanently:

``` bash
rc-service iptables save
```

Rules stored in:

``` text
/etc/iptables/rules-save
```

Enabled restoration during boot:

``` bash
rc-update add iptables boot
```

------------------------------------------------------------------------

### 4. Removed direct Internet access from clients

Both Alpine-Lab-02 and Alpine-Lab-03 originally had:

-   one NIC on `default`
-   one NIC on `lab-isolated`

The `default` NIC was removed with:

``` bash
virsh detach-interface <vm-name> \
    --type network \
    --mac <mac-address> \
    --config
```

After removal the remaining interface became `eth0`.

------------------------------------------------------------------------

### 5. Updated client networking

**Alpine-Lab-02**

``` text
auto lo
iface lo inet loopback
iface lo inet6 loopback

auto eth0
iface eth0 inet static
    address 10.10.10.2
    netmask 255.255.255.0
    gateway 10.10.10.1

iface eth0 inet6 auto
```

**Alpine-Lab-03**

``` text
auto lo
iface lo inet loopback
iface lo inet6 loopback

auto eth0
iface eth0 inet static
    address 10.10.10.3
    netmask 255.255.255.0
    gateway 10.10.10.1

iface eth0 inet6 auto
```

Applied with:

``` bash
sudo ifdown eth0
sudo ifup eth0
```

------------------------------------------------------------------------

## Verification

### Router

``` bash
cat /proc/sys/net/ipv4/ip_forward
```

Result:

``` text
1
```

------------------------------------------------------------------------

### Routing

**Alpine-Lab-02**

``` bash
ip route get 8.8.8.8
```

Result:

``` text
8.8.8.8 via 10.10.10.1 dev eth0
```

**Alpine-Lab-03**

``` bash
ip route get 8.8.8.8
```

Result:

``` text
8.8.8.8 via 10.10.10.1 dev eth0
```

------------------------------------------------------------------------

### Connectivity

Successful:

-   SSH between router and clients using `10.10.10.x`
-   Internet access from Alpine-Lab-02
-   Internet access from Alpine-Lab-03

After rebooting Alpine-Lab-01:

-   `eth1` restored automatically
-   IP forwarding enabled automatically
-   NAT restored automatically
-   Clients retained Internet connectivity

------------------------------------------------------------------------

## Concepts Learned

-   Linux routing
-   Default gateways
-   Multiple network interfaces
-   IPv4 forwarding
-   Network Address Translation (MASQUERADE)
-   OpenRC services
-   Persistent kernel parameters
-   Persistent firewall rules
-   libvirt virtual networks
-   VM network interface management with `virsh`

------------------------------------------------------------------------

## Project Milestone

The lab now behaves like a small enterprise network:

-   Alpine-Lab-01 is the only gateway to the outside world.
-   Alpine-Lab-02 and Alpine-Lab-03 cannot bypass the router.
-   The routed network is fully persistent across reboots.

The platform is now ready for the next phase: **DHCP server
deployment**.
