# Lab Status

This file tracks the current state of the KVM Virtualization Lab.

------------------------------------------------------------------------

# Current Phase

## Phase 6 --- Linux Services & Service Exposure

Current topic:

Linux service management, nginx, HTTP service exposure, service binding,
host firewalling, and persistence.

Status:

✅ **Complete**

Current checkpoint:

**Alpine-Lab-02 is now a persistent nginx web server with a default-deny
host firewall. nginx is bound specifically to `10.10.10.2:80`; SSH and
HTTP are permitted from the trusted `10.10.10.0/24` lab network; nginx
and iptables have both been verified after reboot. Phase 6 technical
work is finished; documentation, Git checkpointing, and snapshots
remain.**

------------------------------------------------------------------------

# Current Infrastructure

``` text
Windows 11 Host
│
└── VirtualBox
    └── Linux Mint 22.2
        └── KVM / libvirt
            ├── Alpine-Lab-01
            │   • Router / NAT Gateway
            │   • DHCP + DNS (dnsmasq)
            │   • Stateful Firewall (iptables)
            │   • SSH Server
            │   • eth0: 192.168.122.252/24
            │   • eth1: 10.10.10.1/24
            │
            ├── Alpine-Lab-02
            │   • nginx Web Server
            │   • Host Firewall (iptables)
            │   • SSH Server
            │   • 10.10.10.2/24
            │   • Gateway/DNS: 10.10.10.1
            │
            └── Alpine-Lab-03
                • Internal Client
                • 10.10.10.3/24
                • Gateway/DNS: 10.10.10.1
```

------------------------------------------------------------------------

# Alpine-Lab-01 Firewall State

Phase 5 remains fully complete.

``` text
INPUT    DROP
FORWARD  DROP
OUTPUT   ACCEPT
```

Persisted INPUT model:

``` text
INPUT DROP
│
├── lo                                             ACCEPT
├── RELATED,ESTABLISHED                            ACCEPT
├── eth1 + 10.10.10.0/24 + TCP/22 NEW            ACCEPT  SSH
├── eth1 + UDP/67                                  ACCEPT  DHCP
├── eth1 + 10.10.10.0/24 + UDP/53                ACCEPT  DNS
├── eth1 + 10.10.10.0/24 + TCP/53 NEW            ACCEPT  DNS
├── eth1 + 10.10.10.0/24 + ICMP echo-request     ACCEPT
└── rate-limited IPTABLES-DROP logging
```

Persisted FORWARD model:

``` text
FORWARD DROP
│
├── RELATED,ESTABLISHED                            ACCEPT
├── eth1 -> eth0 + source 10.10.10.0/24 + NEW    ACCEPT
└── rate-limited FORWARD-DROP logging
```

NAT:

``` text
10.10.10.0/24 -> eth0 -> MASQUERADE
```

Rules are persisted in `/etc/iptables/rules-save`. Firewall logging is
handled by BusyBox `klogd` and `syslogd` and stored in
`/var/log/messages`.

------------------------------------------------------------------------

# Alpine-Lab-02 Service State

## nginx

nginx is installed, running, boot-enabled, and verified after reboot.

Current exposure:

``` text
10.10.10.2:80 -> nginx
```

Current server definition:

``` nginx
server {
    listen 10.10.10.2:80 default_server;
    # listen [::]:80 default_server;

    root /var/lib/nginx/html;
    index index.html;
}
```

Custom page:

``` text
/var/lib/nginx/html/lab.html
```

HTTP access has been verified locally and remotely from the isolated lab
network.

Important logs:

``` text
/var/log/nginx/access.log
/var/log/nginx/error.log
```

------------------------------------------------------------------------

# Alpine-Lab-02 Host Firewall

Current policies:

``` text
INPUT    DROP
FORWARD  ACCEPT
OUTPUT   ACCEPT
```

Current INPUT model:

``` text
INPUT DROP
│
├── lo                                  ACCEPT
├── RELATED,ESTABLISHED                 ACCEPT
├── 10.10.10.0/24 + TCP/22 NEW         ACCEPT  SSH
└── 10.10.10.0/24 + TCP/80 NEW         ACCEPT  HTTP
```

Everything else reaching INPUT falls through to the default DROP policy.

Rules are saved in:

``` text
/etc/iptables/rules-save
```

and restored at boot through OpenRC.

No persistent firewall logging was added on Alpine-Lab-02 because
firewall logging was already studied in depth during Phase 5. Temporary
logging can be added later if needed for DNAT troubleshooting.

------------------------------------------------------------------------

# Phase 6 Experiments Completed

## OpenRC and services

✔ Distinguished installed, running, and boot-enabled states\
✔ Used `rc-service` for current runtime state\
✔ Used `rc-update` for boot behavior\
✔ Demonstrated the distinction with `crond`\
✔ Inspected OpenRC init scripts

## nginx and HTTP

✔ Installed nginx\
✔ Inspected nginx configuration structure\
✔ Started and managed nginx through OpenRC\
✔ Inspected master and worker processes\
✔ Inspected listening sockets with `ss`\
✔ Created `/var/lib/nginx/html/lab.html`\
✔ Verified HTTP `200 OK`\
✔ Used `curl`, `curl -i`, and `curl -I`

## Logging

✔ Inspected nginx access and error logs\
✔ Observed HTTP 200, 304, and 404 behavior\
✔ Verified missing resources in the error log\
✔ Confirmed firewall-dropped requests do not reach nginx access logging

## Permissions and least privilege

✔ Inspected path permissions with `namei -l`\
✔ Identified nginx master and worker users\
✔ Verified nginx can read static content\
✔ Verified nginx cannot modify the root-owned static page\
✔ Reinforced directory traverse (`x`) permission behavior

## Service binding

Initial listeners:

``` text
0.0.0.0:80
[::]:80
```

Final listener:

``` text
10.10.10.2:80
```

✔ Changed nginx to a specific IPv4 binding\
✔ Validated configuration with `nginx -t`\
✔ Investigated graceful reload/socket behavior\
✔ Used a full restart to obtain the intended socket state\
✔ Verified `127.0.0.1:80` no longer listens\
✔ Verified `10.10.10.2:80` continues serving HTTP

Key distinction:

``` text
Binding   -> WHERE the service listens
Routing   -> CAN the client find a path
Firewall  -> IS the traffic permitted
```

## DNS experiment

✔ Verified Lab-01 dnsmasq resolves
`alpine-lab-02.lab.local -> 10.10.10.2`\
✔ Verified direct DNS query from Mint with `dig @10.10.10.1`\
✔ Determined Mint normally uses its own external DNS configuration\
✔ Deliberately deferred split-DNS configuration\
✔ Recorded `.local` as an mDNS-reserved naming consideration

## Host firewall experiments

✔ Installed iptables on Alpine-Lab-02\
✔ Confirmed the initial ACCEPT policies\
✔ Tested temporary TCP/80 DROP\
✔ Observed timeout and increasing firewall counters\
✔ Confirmed nginx received no request\
✔ Tested `REJECT --reject-with tcp-reset`\
✔ Observed immediate client failure\
✔ Stopped nginx while TCP/80 was permitted\
✔ Observed immediate failure with no listener

Comparison:

``` text
Listening + allowed   -> HTTP 200
Listening + DROP      -> timeout
Listening + REJECT    -> immediate failure
Not listening         -> immediate failure
```

## Final default-deny firewall

✔ Loopback ACCEPT\
✔ `RELATED,ESTABLISHED` ACCEPT\
✔ NEW SSH from `10.10.10.0/24` ACCEPT\
✔ NEW HTTP from `10.10.10.0/24` ACCEPT\
✔ INPUT policy changed to DROP\
✔ SSH verified\
✔ HTTP verified\
✔ TCP/9999 verified as blocked\
✔ Rules saved with `iptables-save`\
✔ iptables enabled in OpenRC\
✔ Full reboot performed\
✔ Firewall rules restored after reboot\
✔ nginx restored after reboot\
✔ TCP/80 listener restored after reboot\
✔ HTTP connectivity verified after reboot

------------------------------------------------------------------------

# Completed Milestones

## Phase 1 --- KVM Fundamentals

✔ KVM installation\
✔ virsh fundamentals\
✔ Alpine installation

## Phase 2 --- Virtual Machine Management

✔ SSH key authentication\
✔ Manual VM cloning\
✔ virt-clone\
✔ XML editing\
✔ Snapshot strategy

## Phase 3 --- Networking Foundations

✔ Custom virtual network\
✔ Linux router\
✔ Static addressing\
✔ IP forwarding\
✔ NAT\
✔ Internet access\
✔ Inter-VM routing\
✔ Documentation complete

## Phase 4 --- Network Services

✔ dnsmasq\
✔ DHCP and reservations\
✔ DNS and forwarding\
✔ Local DNS zone and search domain\
✔ Automatic hostname resolution\
✔ Modular `/etc/dnsmasq.d` configuration\
✔ Documentation complete

## Phase 5 --- Firewall & Security

✔ Stateful INPUT and FORWARD firewalls\
✔ Default-deny policies\
✔ Logging and hardening\
✔ Nmap packet-analysis experiments\
✔ DROP vs REJECT\
✔ `rp_filter` investigation\
✔ NAT/MASQUERADE and conntrack verification\
✔ Persistence and reboot recovery\
✔ Documentation complete\
✔ Git checkpoint complete\
✔ Final snapshots complete

## Phase 6 --- Linux Services & Service Exposure

✔ OpenRC service-management fundamentals\
✔ nginx installation and service management\
✔ HTTP and static web content\
✔ nginx logs\
✔ Filesystem permissions and least privilege\
✔ Specific service binding\
✔ DNS/service-name experiment\
✔ Host firewall\
✔ DROP vs REJECT vs no-listener experiment\
✔ Default-deny INPUT\
✔ nginx persistence\
✔ iptables persistence\
✔ Reboot verification

**Technical work complete. Documentation/Git/snapshot checkpoint is
being finalized.**

------------------------------------------------------------------------

# Client Health

## Alpine-Lab-02

``` text
IP:       10.10.10.2/24
Gateway:  10.10.10.1
DNS:      10.10.10.1
Domain:   lab.local
Role:     nginx Web Server
```

Verified:

-   ✔ Router connectivity
-   ✔ Internet connectivity through Alpine-Lab-01
-   ✔ DNS
-   ✔ SSH
-   ✔ nginx TCP/80
-   ✔ HTTP access
-   ✔ Host firewall
-   ✔ nginx persistence
-   ✔ Firewall persistence

## Alpine-Lab-03

``` text
IP:       10.10.10.3/24
Gateway:  10.10.10.1
DNS:      10.10.10.1
Domain:   lab.local
Role:     Internal Client
```

Verified:

-   ✔ Router connectivity
-   ✔ Internet connectivity
-   ✔ DNS
-   ✔ SSH from Alpine-Lab-01
-   ✔ HTTP access to Alpine-Lab-02

------------------------------------------------------------------------

# Next Goal

## Phase 7 --- DNAT / Service Publishing

Before beginning Phase 7:

1.  Add `docs/11-linux-services-nginx.md`.
2.  Update `README.md`.
3.  Update `LAB_STATUS.md`.
4.  Update `CHANGELOG.md`.
5.  Review with `git status` and `git diff`.
6.  Commit and push the Phase 6 checkpoint.
7.  Shut down the VMs cleanly.
8.  Create the Phase 6 VM snapshot(s).
9.  Create the Linux Mint / VirtualBox snapshot.

Planned packet flow:

``` text
external/upstream client
          │
          │ TCP :8080
          ▼
    Alpine-Lab-01
   router / firewall
          │
          │ DNAT
          ▼
     10.10.10.2:80
    Alpine-Lab-02
         nginx
```

Initial conceptual mapping:

``` text
Alpine-Lab-01:8080 -> DNAT -> Alpine-Lab-02:80
```

Phase 7 will connect routing, NAT, firewalling, conntrack, service
binding, and host firewalling into one end-to-end packet flow.

------------------------------------------------------------------------

# Latest Snapshots

Phase 5 final checkpoints are already preserved.

Current known-good technical state:

``` text
Phase 6 --- Linux Services & Service Exposure complete
```

A new Phase 6 checkpoint will be created after the documentation is
committed and pushed.

------------------------------------------------------------------------

# Repository Status

Current branch:

``` text
main
```

Current documentation milestone:

``` text
Phase 6 --- Linux Services & Service Exposure complete
```

Files being updated:

``` text
docs/11-linux-services-nginx.md
README.md
LAB_STATUS.md
CHANGELOG.md
```

Workflow:

``` text
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
Phase 7 — DNAT / Service Publishing
```

The known-good Phase 6 service and firewall configuration should remain
unchanged while this checkpoint is documented and preserved.
