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
-   Distributed gateway, DNS server, and search domain via DHCP.

### DNS

-   Implemented a local DNS server.
-   Configured DNS forwarding.
-   Created the local `lab.local` DNS zone.
-   Enabled automatic hostname resolution.
-   Configured Alpine-Lab-01 to use its own DNS service.

### System Administration

-   Prevented DHCP from overwriting `/etc/resolv.conf`.
-   Modularized dnsmasq configuration using `/etc/dnsmasq.d/lab.conf`.
-   Refactored service configuration using drop-in configuration
    practices.

### Verification

-   Verified DHCP lease allocation.
-   Verified DNS forwarding.
-   Verified local hostname resolution.
-   Verified SSH connectivity using hostnames.
-   Completed end-to-end network service validation.

### Documentation

-   Added `docs/09-network-services-dhcp-dns.md`.
-   Updated `README.md`, `LAB_STATUS.md`, and `CHANGELOG.md`.

------------------------------------------------------------------------

## Phase 5 -- Firewall & Security

### Netfilter and iptables

-   Introduced Linux Netfilter packet filtering.
-   Studied `INPUT`, `OUTPUT`, and `FORWARD`.
-   Implemented default-deny `INPUT` and `FORWARD` policies.
-   Introduced stateful filtering with `conntrack`.
-   Studied `NEW`, `ESTABLISHED`, and `RELATED`.

### Router INPUT Hardening

-   Explicitly allowed loopback.
-   Allowed `RELATED,ESTABLISHED`.
-   Restricted SSH to the trusted LAN.
-   Allowed DHCP on the trusted LAN interface.
-   Restricted DNS UDP/53 and TCP/53 to the trusted LAN.
-   Restricted ICMP Echo Request to the trusted LAN.
-   Used packet and byte counters to verify rule matches.

Final INPUT model:

``` text
INPUT DROP
│
├── lo                                             ACCEPT
├── RELATED,ESTABLISHED                            ACCEPT
├── eth1 + 10.10.10.0/24 + TCP/22 NEW            ACCEPT
├── eth1 + UDP/67                                  ACCEPT
├── eth1 + 10.10.10.0/24 + UDP/53                ACCEPT
├── eth1 + 10.10.10.0/24 + TCP/53 NEW            ACCEPT
├── eth1 + 10.10.10.0/24 + ICMP echo-request     ACCEPT
└── rate-limited IPTABLES-DROP logging
```

### Stateful FORWARD Filtering

-   Added `RELATED,ESTABLISHED` return-traffic handling.
-   Added explicit NEW forwarding from `eth1` to `eth0` for
    `10.10.10.0/24`.
-   Changed the default `FORWARD` policy to `DROP`.
-   Verified outbound connectivity from Alpine-Lab-02 and Alpine-Lab-03.
-   Verified unsolicited upstream-to-LAN NEW traffic is dropped.

Final forwarding model:

``` text
LAN -> WAN NEW                     ACCEPT
WAN -> LAN ESTABLISHED/RELATED     ACCEPT
WAN -> LAN unsolicited NEW         DROP
```

### Routing and Longest-Prefix Matching

-   Confirmed Linux Mint has a directly connected `10.10.10.0/24` route
    through `virbr10`.
-   Added a temporary `/32` route to force traffic for `10.10.10.2`
    through Alpine-Lab-01.
-   Demonstrated longest-prefix matching:

``` text
/32 > /24 > /0
```

-   Verified forced unsolicited traffic reached the `FORWARD DROP`
    policy.
-   Removed the temporary route afterward.

### Firewall Logging and Hardening

-   Added rate-limited `IPTABLES-DROP:` logging to INPUT.
-   Added rate-limited `FORWARD-DROP:` logging to FORWARD.
-   Verified BusyBox `syslogd` and `klogd`.
-   Confirmed firewall events in `/var/log/messages`.
-   Hardened SSH, DNS, and ICMP exposure by interface and source.
-   Kept DHCP restricted by interface because an initial DHCP client can
    use source `0.0.0.0`.

### Nmap and DROP vs REJECT

-   Installed Nmap on Alpine-Lab-02.
-   Demonstrated `open`, `closed`, and `filtered`.
-   Verified TCP/22 as open.
-   Temporarily exposed TCP/8888 with no listener and observed `closed`.
-   Left TCP/9999 under DROP and observed `filtered`.
-   Tested `REJECT --reject-with tcp-reset` and observed immediate
    rejection.
-   Removed temporary diagnostic rules afterward.

### Reverse-Path Filtering

-   Investigated strict `rp_filter=1`.
-   Distinguished routing plausibility from firewall authorization:

``` text
rp_filter  -> Is the source plausible according to routing?
iptables   -> Is the source authorized by security policy?
```

### NAT / MASQUERADE and conntrack

-   Revisited the persistent MASQUERADE rule:

``` bash
-A POSTROUTING -s 10.10.10.0/24 -o eth0 -j MASQUERADE
```

-   Verified NAT and conntrack behavior with packet counters.
-   Reinforced that NAT establishes a translation for a tracked flow
    while filter rules continue evaluating packets in that flow.

### Persistence and Recovery

-   Saved the final firewall.
-   Verified `/etc/iptables/rules-save`.
-   Rebooted Alpine-Lab-01.
-   Verified routing, IP forwarding, iptables, dnsmasq, syslog, klogd,
    DNS, Internet routing, NAT, INPUT logging, and FORWARD policy.
-   Verified Alpine-Lab-02 and Alpine-Lab-03 connectivity after reboot.

### Documentation

-   Finalized `docs/10-firewall-security.md`.
-   Updated `README.md`, `LAB_STATUS.md`, and `CHANGELOG.md`.
-   Committed and pushed the Phase 5 checkpoint.
-   Created final Phase 5 snapshots.

### Phase 5 Complete

Final filter policies:

``` text
INPUT    DROP
FORWARD  DROP
OUTPUT   ACCEPT
```

------------------------------------------------------------------------

## Phase 6 -- Linux Services & Service Exposure

### OpenRC Service Management

-   Reviewed the distinction between service definitions and daemon
    processes.
-   Used `rc-status`, `rc-service`, and `rc-update`.
-   Demonstrated with `crond` that runtime state and boot enablement are
    independent.
-   Established:

``` text
rc-service start/stop   -> runtime state NOW
rc-update add/del       -> behavior on future boots
```

-   Inspected OpenRC init scripts including `/etc/init.d/sshd` and
    `/etc/init.d/nginx`.

### nginx Installation

-   Installed nginx on Alpine-Lab-02:

``` bash
sudo apk add nginx
```

-   Inspected nginx configuration, executable, OpenRC files, and
    document root.
-   Installed `iproute2` for `ss`.
-   Started nginx and inspected its master and worker processes.
-   Observed initial wildcard listeners:

``` text
0.0.0.0:80
[::]:80
```

### First HTTP Service

-   Replaced the default deliberate 404 configuration with a normal
    static document root.
-   Created `/var/lib/nginx/html/lab.html`.
-   Validated configuration with `nginx -t`.
-   Verified HTTP `200 OK` locally and remotely.
-   Used `curl`, `curl -i`, and `curl -I` to distinguish body,
    headers+body, and HEAD requests.

### nginx Logging

-   Inspected `/var/log/nginx/access.log`.
-   Inspected `/var/log/nginx/error.log`.
-   Observed HTTP 200, 304, and 404 behavior.
-   Generated deliberate missing-resource requests.
-   Confirmed firewall-dropped requests never reach nginx access
    logging.

### Filesystem Permissions and Least Privilege

-   Inspected `/var/lib/nginx/html/lab.html` with `namei -l`.
-   Identified nginx master and worker process privileges.
-   Verified the nginx user can read static content.
-   Verified the nginx user cannot modify the root-owned static page.
-   Reinforced directory traversal (`x`) permissions and least
    privilege.

### Specific Service Binding

-   Backed up the original server configuration.
-   Changed nginx to:

``` nginx
server {
    listen 10.10.10.2:80 default_server;
    # listen [::]:80 default_server;

    root /var/lib/nginx/html;
    index index.html;
}
```

-   Validated the configuration.
-   Investigated graceful reload/socket reuse behavior.
-   Confirmed no active configuration still requested wildcard binding.
-   Performed a full restart.
-   Verified the final listener:

``` text
10.10.10.2:80
```

-   Verified `127.0.0.1:80` no longer accepts connections.
-   Verified `10.10.10.2:80` continues returning HTTP 200.

Established:

``` text
Binding   -> WHERE the service listens
Routing   -> CAN the client find a network path
Firewall  -> IS the traffic permitted
```

### Remote Connectivity and DNS

-   Verified HTTP access from Linux Mint (`10.10.10.254`).
-   Verified HTTP access from Alpine-Lab-03 (`10.10.10.3`).
-   Correlated both clients with nginx access-log source addresses.
-   Verified hostname-based HTTP access from Alpine-Lab-01.
-   Determined Linux Mint can route to Alpine-Lab-02 but normally does
    not use Alpine-Lab-01 for DNS.
-   Verified lab DNS directly:

``` bash
dig @10.10.10.1 alpine-lab-02.lab.local
```

-   Received `10.10.10.2`.
-   Deferred split-DNS configuration because it was outside Phase 6.
-   Recorded `.local` as an mDNS-reserved naming consideration.

### Alpine-Lab-02 Host Firewall

-   Installed iptables.
-   Confirmed the initial empty firewall used ACCEPT policies.

### DROP vs REJECT vs No Listener

-   Added a temporary TCP/80 DROP rule and observed client timeout.
-   Verified firewall counters increased.
-   Confirmed nginx received no request.
-   Replaced DROP with `REJECT --reject-with tcp-reset` and observed
    immediate failure.
-   Stopped nginx with TCP/80 permitted and observed immediate failure
    because no service was listening.

Final comparison:

``` text
Listening + allowed   -> HTTP 200
Listening + DROP      -> timeout
Listening + REJECT    -> immediate failure
Not listening         -> immediate failure
```

### Default-Deny Host Firewall

Built the permanent INPUT firewall:

``` text
1  lo                                  ACCEPT
2  ESTABLISHED,RELATED                 ACCEPT
3  10.10.10.0/24 -> TCP/22 NEW        ACCEPT
4  10.10.10.0/24 -> TCP/80 NEW        ACCEPT
   INPUT policy                        DROP
```

-   Verified SSH remained available.
-   Verified HTTP remained available.
-   Tested TCP/9999 and confirmed it timed out under the default DROP
    policy.

### Firewall Persistence

-   Saved the rules with:

``` bash
sudo iptables-save | sudo tee /etc/iptables/rules-save >/dev/null
```

-   Enabled boot restoration:

``` bash
sudo rc-update add iptables default
```

-   Reinforced that Netfilter rules live in the kernel and the OpenRC
    service restores the saved state at boot.

### nginx Persistence

-   Enabled nginx in the default OpenRC runlevel.
-   Rebooted Alpine-Lab-02.
-   Verified nginx automatically returned.
-   Verified the specific `10.10.10.2:80` listener.
-   Verified HTTP access after reboot.

### Full Phase 6 Reboot Verification

After reboot, the firewall returned as:

``` text
INPUT policy DROP
lo                                  ACCEPT
RELATED,ESTABLISHED                 ACCEPT
10.10.10.0/24 -> TCP/22 NEW        ACCEPT
10.10.10.0/24 -> TCP/80 NEW        ACCEPT
```

-   Observed the SSH NEW-rule counter increment after reconnecting.
-   Verified firewall persistence.
-   Verified nginx persistence.
-   Verified TCP/80 listener persistence.
-   Verified remote HTTP `200 OK`.

### Firewall Logging Decision

-   Deliberately did not add persistent firewall logging to
    Alpine-Lab-02.
-   Phase 5 already covered firewall logging in depth.
-   Temporary logging remains available for future DNAT troubleshooting.

### Documentation

-   Added `docs/11-linux-services-nginx.md`.
-   Updated `README.md`.
-   Updated `LAB_STATUS.md`.
-   Updated `CHANGELOG.md`.

### Phase 6 Complete

Final Alpine-Lab-02 capabilities:

-   nginx web server managed through OpenRC
-   persistent service startup
-   custom static HTTP content
-   access and error logging
-   least-privilege filesystem access
-   specific binding to `10.10.10.2:80`
-   remote HTTP access from the lab network
-   default-deny host INPUT firewall
-   stateful conntrack handling
-   SSH restricted to `10.10.10.0/24`
-   HTTP restricted to `10.10.10.0/24`
-   unapproved inbound ports dropped
-   firewall persistence
-   full reboot/recovery verification

Final nginx exposure:

``` text
10.10.10.2:80 -> nginx
```

------------------------------------------------------------------------

## Next -- Phase 7

Before beginning Phase 7:

-   Review the final repository changes with `git status` and
    `git diff`.
-   Commit and push the completed Phase 6 documentation.
-   Shut down the VMs cleanly.
-   Create the final Phase 6 VM snapshot(s).
-   Create the corresponding Linux Mint / VirtualBox snapshot.

Phase 7 will introduce **DNAT / Service Publishing**.

Initial conceptual target:

``` text
Alpine-Lab-01:8080 -> DNAT -> Alpine-Lab-02:80
```

This will connect routing, NAT, firewalling, conntrack, service binding,
and the Alpine-Lab-02 host firewall into a complete end-to-end
service-publishing flow.
