*\*\****\*\*\\\\***\*\\\\\****# Changelog\\\\***\*\\\\\****\*\***\*\*

All notable changes to this laboratory are documented here in

chronological order.

---

*\*\****\*\*\\\\***\*\\\\\****## Phase 1 -- KVM Fundamentals\\\\***\*\\\\\****\*\***\*\*

-   Installed Linux Mint KVM environment.

-   Installed Alpine Linux.

-   Learned `virsh` fundamentals.

-   Created baseline snapshots.

-   Established the initial Git repository structure.

---

*\*\****\*\*\\\\***\*\\\\\****## Phase 2 -- Virtual Machine Management\\\\***\*\\\\\****\*\***\*\*

-   Configured SSH key authentication.

-   Learned manual VM cloning.

-   Learned `virt-clone`.

-   Explored and edited libvirt XML definitions.

-   Implemented a VM snapshot strategy.

-   Documented cloning procedures.

---

*\*\****\*\*\\\\***\*\\\\\****## Phase 3 -- Networking Foundations\\\\***\*\\\\\****\*\***\*\*

-   Built a custom isolated virtual network.

-   Converted Alpine-Lab-01 into a Linux router.

-   Enabled persistent IP forwarding.

-   Configured NAT using `iptables`.

-   Verified Internet connectivity for internal clients.

-   Implemented static addressing.

-   Documented networking experiments and packet analysis.

---

*\*\****\*\*\\\\***\*\\\\\****## Phase 4 -- Network Services\\\\***\*\\\\\****\*\***\*\*

*\*\****\*\*\\\\***\*\\\\\****### DHCP\\\\***\*\\\\\****\*\***\*\*

-   Installed and configured `dnsmasq`.

-   Configured DHCP for the isolated network.

-   Added static DHCP reservations for Alpine-Lab-02 and Alpine-Lab-03.

-   Distributed gateway, DNS server, and search domain via DHCP.

*\*\****\*\*\\\\***\*\\\\\****### DNS\\\\***\*\\\\\****\*\***\*\*

-   Implemented a local DNS server.

-   Configured DNS forwarding.

-   Created the local `lab.local` DNS zone.

-   Enabled automatic hostname resolution.

-   Configured Alpine-Lab-01 to use its own DNS service.

*\*\****\*\*\\\\***\*\\\\\****### System Administration\\\\***\*\\\\\****\*\***\*\*

-   Prevented DHCP from overwriting `/etc/resolv.conf`.

-   Modularized dnsmasq configuration using `/etc/dnsmasq.d/lab.conf`.

-   Refactored service configuration using drop-in configuration

    practices.

*\*\****\*\*\\\\***\*\\\\\****### Verification\\\\***\*\\\\\****\*\***\*\*

-   Verified DHCP lease allocation.

-   Verified DNS forwarding.

-   Verified local hostname resolution.

-   Verified SSH connectivity using hostnames.

-   Completed end-to-end network service validation.

*\*\****\*\*\\\\***\*\\\\\****### Documentation\\\\***\*\\\\\****\*\***\*\*

-   Added `docs/09-network-services-dhcp-dns.md`.

-   Updated `README.md`, `LAB_STATUS.md`, and `CHANGELOG.md`.

---

*\*\****\*\*\\\\***\*\\\\\****## Phase 5 -- Firewall & Security\\\\***\*\\\\\****\*\***\*\*

*\*\****\*\*\\\\***\*\\\\\****### Netfilter and iptables\\\\***\*\\\\\****\*\***\*\*

-   Introduced Linux Netfilter packet filtering.

-   Studied `INPUT`, `OUTPUT`, and `FORWARD`.

-   Implemented default-deny `INPUT` and `FORWARD` policies.

-   Introduced stateful filtering with `conntrack`.

-   Studied `NEW`, `ESTABLISHED`, and `RELATED`.

*\*\****\*\*\\\\***\*\\\\\****### Router INPUT Hardening\\\\***\*\\\\\****\*\***\*\*

-   Explicitly allowed loopback.

-   Allowed `RELATED,ESTABLISHED`.

-   Restricted SSH to the trusted LAN.

-   Allowed DHCP on the trusted LAN interface.

-   Restricted DNS UDP/53 and TCP/53 to the trusted LAN.

-   Restricted ICMP Echo Request to the trusted LAN.

-   Used packet and byte counters to verify rule matches.

Final INPUT model:

``` text

INPUT DROP

│

├── lo                                             ACCEPT

├── RELATED,ESTABLISHED                            ACCEPT

├── eth1 + 10.10.10.0/24 + TCP/22 NEW            ACCEPT

├── eth1 + UDP/67                                  ACCEPT

├── eth1 + 10.10.10.0/24 + UDP/53                ACCEPT

├── eth1 + 10.10.10.0/24 + TCP/53 NEW            ACCEPT

├── eth1 + 10.10.10.0/24 + ICMP echo-request     ACCEPT

└── rate-limited IPTABLES-DROP logging

```

*\*\****\*\*\\\\***\*\\\\\****### Stateful FORWARD Filtering\\\\***\*\\\\\****\*\***\*\*

-   Added `RELATED,ESTABLISHED` return-traffic handling.

-   Added explicit NEW forwarding from `eth1` to `eth0` for

    `10.10.10.0/24`.

-   Changed the default `FORWARD` policy to `DROP`.

-   Verified outbound connectivity from Alpine-Lab-02 and Alpine-Lab-03.

-   Verified unsolicited upstream-to-LAN NEW traffic is dropped.

Final forwarding model:

``` text

LAN -> WAN NEW                     ACCEPT

WAN -> LAN ESTABLISHED/RELATED     ACCEPT

WAN -> LAN unsolicited NEW         DROP

```

*\*\****\*\*\\\\***\*\\\\\****### Routing and Longest-Prefix Matching\\\\***\*\\\\\****\*\***\*\*

-   Confirmed Linux Mint has a directly connected `10.10.10.0/24` route

    through `virbr10`.

-   Added a temporary `/32` route to force traffic for `10.10.10.2`

    through Alpine-Lab-01.

-   Demonstrated longest-prefix matching:

``` text

/32 > /24 > /0

```

-   Verified forced unsolicited traffic reached the `FORWARD DROP`

    policy.

-   Removed the temporary route afterward.

*\*\****\*\*\\\\***\*\\\\\****### Firewall Logging and Hardening\\\\***\*\\\\\****\*\***\*\*

-   Added rate-limited `IPTABLES-DROP:` logging to INPUT.

-   Added rate-limited `FORWARD-DROP:` logging to FORWARD.

-   Verified BusyBox `syslogd` and `klogd`.

-   Confirmed firewall events in `/var/log/messages`.

-   Hardened SSH, DNS, and ICMP exposure by interface and source.

-   Kept DHCP restricted by interface because an initial DHCP client can

    use source `0.0.0.0`.

*\*\****\*\*\\\\***\*\\\\\****### Nmap and DROP vs REJECT\\\\***\*\\\\\****\*\***\*\*

-   Installed Nmap on Alpine-Lab-02.

-   Demonstrated `open`, `closed`, and `filtered`.

-   Verified TCP/22 as open.

-   Temporarily exposed TCP/8888 with no listener and observed `closed`.

-   Left TCP/9999 under DROP and observed `filtered`.

-   Tested `REJECT --reject-with tcp-reset` and observed immediate

    rejection.

-   Removed temporary diagnostic rules afterward.

*\*\****\*\*\\\\***\*\\\\\****### Reverse-Path Filtering\\\\***\*\\\\\****\*\***\*\*

-   Investigated strict `rp_filter=1`.

-   Distinguished routing plausibility from firewall authorization:

``` text

rp_filter  -> Is the source plausible according to routing?

iptables   -> Is the source authorized by security policy?

```

*\*\****\*\*\\\\***\*\\\\\****### NAT / MASQUERADE and conntrack\\\\***\*\\\\\****\*\***\*\*

-   Revisited the persistent MASQUERADE rule:

``` bash

-A POSTROUTING -s 10.10.10.0/24 -o eth0 -j MASQUERADE

```

-   Verified NAT and conntrack behavior with packet counters.

-   Reinforced that NAT establishes a translation for a tracked flow

    while filter rules continue evaluating packets in that flow.

*\*\****\*\*\\\\***\*\\\\\****### Persistence and Recovery\\\\***\*\\\\\****\*\***\*\*

-   Saved the final firewall.

-   Verified `/etc/iptables/rules-save`.

-   Rebooted Alpine-Lab-01.

-   Verified routing, IP forwarding, iptables, dnsmasq, syslog, klogd,

    DNS, Internet routing, NAT, INPUT logging, and FORWARD policy.

-   Verified Alpine-Lab-02 and Alpine-Lab-03 connectivity after reboot.

*\*\****\*\*\\\\***\*\\\\\****### Documentation\\\\***\*\\\\\****\*\***\*\*

-   Finalized `docs/10-firewall-security.md`.

-   Updated `README.md`, `LAB_STATUS.md`, and `CHANGELOG.md`.

-   Committed and pushed the Phase 5 checkpoint.

-   Created final Phase 5 snapshots.

*\*\****\*\*\\\\***\*\\\\\****### Phase 5 Complete\\\\***\*\\\\\****\*\***\*\*

Final filter policies:

``` text

INPUT    DROP

FORWARD  DROP

OUTPUT   ACCEPT

```

---

*\*\****\*\*\\\\***\*\\\\\****## Phase 6 -- Linux Services & Service Exposure\\\\***\*\\\\\****\*\***\*\*

*\*\****\*\*\\\\***\*\\\\\****### OpenRC Service Management\\\\***\*\\\\\****\*\***\*\*

-   Reviewed the distinction between service definitions and daemon

    processes.

-   Used `rc-status`, `rc-service`, and `rc-update`.

-   Demonstrated with `crond` that runtime state and boot enablement are

    independent.

-   Established:

``` text

rc-service start/stop   -> runtime state NOW

rc-update add/del       -> behavior on future boots

```

-   Inspected OpenRC init scripts including `/etc/init.d/sshd` and

    `/etc/init.d/nginx`.

*\*\****\*\*\\\\***\*\\\\\****### nginx Installation\\\\***\*\\\\\****\*\***\*\*

-   Installed nginx on Alpine-Lab-02:

``` bash

sudo apk add nginx

```

-   Inspected nginx configuration, executable, OpenRC files, and

    document root.

-   Installed `iproute2` for `ss`.

-   Started nginx and inspected its master and worker processes.

-   Observed initial wildcard listeners:

``` text

0.0.0.0:80

[::]:80

```

*\*\****\*\*\\\\***\*\\\\\****### First HTTP Service\\\\***\*\\\\\****\*\***\*\*

-   Replaced the default deliberate 404 configuration with a normal

    static document root.

-   Created `/var/lib/nginx/html/lab.html`.

-   Validated configuration with `nginx -t`.

-   Verified HTTP `200 OK` locally and remotely.

-   Used `curl`, `curl -i`, and `curl -I` to distinguish body,

    headers+body, and HEAD requests.

*\*\****\*\*\\\\***\*\\\\\****### nginx Logging\\\\***\*\\\\\****\*\***\*\*

-   Inspected `/var/log/nginx/access.log`.

-   Inspected `/var/log/nginx/error.log`.

-   Observed HTTP 200, 304, and 404 behavior.

-   Generated deliberate missing-resource requests.

-   Confirmed firewall-dropped requests never reach nginx access

    logging.

*\*\****\*\*\\\\***\*\\\\\****### Filesystem Permissions and Least Privilege\\\\***\*\\\\\****\*\***\*\*

-   Inspected `/var/lib/nginx/html/lab.html` with `namei -l`.

-   Identified nginx master and worker process privileges.

-   Verified the nginx user can read static content.

-   Verified the nginx user cannot modify the root-owned static page.

-   Reinforced directory traversal (`x`) permissions and least

    privilege.

*\*\****\*\*\\\\***\*\\\\\****### Specific Service Binding\\\\***\*\\\\\****\*\***\*\*

-   Backed up the original server configuration.

-   Changed nginx to:

``` nginx

server {

    listen 10.10.10.2:80 default_server;

    # listen [::]:80 default_server;

    root /var/lib/nginx/html;

    index index.html;

}

```

-   Validated the configuration.

-   Investigated graceful reload/socket reuse behavior.

-   Confirmed no active configuration still requested wildcard binding.

-   Performed a full restart.

-   Verified the final listener:

``` text

10.10.10.2:80

```

-   Verified `127.0.0.1:80` no longer accepts connections.

-   Verified `10.10.10.2:80` continues returning HTTP 200.

Established:

``` text

Binding   -> WHERE the service listens

Routing   -> CAN the client find a network path

Firewall  -> IS the traffic permitted

```

*\*\****\*\*\\\\***\*\\\\\****### Remote Connectivity and DNS\\\\***\*\\\\\****\*\***\*\*

-   Verified HTTP access from Linux Mint (`10.10.10.254`).

-   Verified HTTP access from Alpine-Lab-03 (`10.10.10.3`).

-   Correlated both clients with nginx access-log source addresses.

-   Verified hostname-based HTTP access from Alpine-Lab-01.

-   Determined Linux Mint can route to Alpine-Lab-02 but normally does

    not use Alpine-Lab-01 for DNS.

-   Verified lab DNS directly:

``` bash

dig @10.10.10.1 alpine-lab-02.lab.local

```

-   Received `10.10.10.2`.

-   Deferred split-DNS configuration because it was outside Phase 6.

-   Recorded `.local` as an mDNS-reserved naming consideration.

*\*\****\*\*\\\\***\*\\\\\****### Alpine-Lab-02 Host Firewall\\\\***\*\\\\\****\*\***\*\*

-   Installed iptables.

-   Confirmed the initial empty firewall used ACCEPT policies.

*\*\****\*\*\\\\***\*\\\\\****### DROP vs REJECT vs No Listener\\\\***\*\\\\\****\*\***\*\*

-   Added a temporary TCP/80 DROP rule and observed client timeout.

-   Verified firewall counters increased.

-   Confirmed nginx received no request.

-   Replaced DROP with `REJECT --reject-with tcp-reset` and observed

    immediate failure.

-   Stopped nginx with TCP/80 permitted and observed immediate failure

    because no service was listening.

Final comparison:

``` text

Listening + allowed   -> HTTP 200

Listening + DROP      -> timeout

Listening + REJECT    -> immediate failure

Not listening         -> immediate failure

```

*\*\****\*\*\\\\***\*\\\\\****### Default-Deny Host Firewall\\\\***\*\\\\\****\*\***\*\*

Built the permanent INPUT firewall:

``` text

1  lo                                  ACCEPT

2  ESTABLISHED,RELATED                 ACCEPT

3  10.10.10.0/24 -> TCP/22 NEW        ACCEPT

4  10.10.10.0/24 -> TCP/80 NEW        ACCEPT

   INPUT policy                        DROP

```

-   Verified SSH remained available.

-   Verified HTTP remained available.

-   Tested TCP/9999 and confirmed it timed out under the default DROP

    policy.

*\*\****\*\*\\\\***\*\\\\\****### Firewall Persistence\\\\***\*\\\\\****\*\***\*\*

-   Saved the rules with:

``` bash

sudo iptables-save | sudo tee /etc/iptables/rules-save >/dev/null

```

-   Enabled boot restoration:

``` bash

sudo rc-update add iptables default

```

-   Reinforced that Netfilter rules live in the kernel and the OpenRC

    service restores the saved state at boot.

*\*\****\*\*\\\\***\*\\\\\****### nginx Persistence\\\\***\*\\\\\****\*\***\*\*

-   Enabled nginx in the default OpenRC runlevel.

-   Rebooted Alpine-Lab-02.

-   Verified nginx automatically returned.

-   Verified the specific `10.10.10.2:80` listener.

-   Verified HTTP access after reboot.

*\*\****\*\*\\\\***\*\\\\\****### Full Phase 6 Reboot Verification\\\\***\*\\\\\****\*\***\*\*

After reboot, the firewall returned as:

``` text

INPUT policy DROP

lo                                  ACCEPT

RELATED,ESTABLISHED                 ACCEPT

10.10.10.0/24 -> TCP/22 NEW        ACCEPT

10.10.10.0/24 -> TCP/80 NEW        ACCEPT

```

-   Observed the SSH NEW-rule counter increment after reconnecting.

-   Verified firewall persistence.

-   Verified nginx persistence.

-   Verified TCP/80 listener persistence.

-   Verified remote HTTP `200 OK`.

*\*\****\*\*\\\\***\*\\\\\****### Firewall Logging Decision\\\\***\*\\\\\****\*\***\*\*

-   Deliberately did not add persistent firewall logging to

    Alpine-Lab-02.

-   Phase 5 already covered firewall logging in depth.

-   Temporary logging remains available for future DNAT troubleshooting.

*\*\****\*\*\\\\***\*\\\\\****### Documentation\\\\***\*\\\\\****\*\***\*\*

-   Added `docs/11-linux-services-nginx.md`.

-   Updated `README.md`.

-   Updated `LAB_STATUS.md`.

-   Updated `CHANGELOG.md`.

*\*\****\*\*\\\\***\*\\\\\****### Phase 6 Complete\\\\***\*\\\\\****\*\***\*\*

Final Alpine-Lab-02 capabilities:

-   nginx web server managed through OpenRC

-   persistent service startup

-   custom static HTTP content

-   access and error logging

-   least-privilege filesystem access

-   specific binding to `10.10.10.2:80`

-   remote HTTP access from the lab network

-   default-deny host INPUT firewall

-   stateful conntrack handling

-   SSH restricted to `10.10.10.0/24`

-   HTTP restricted to `10.10.10.0/24`

-   unapproved inbound ports dropped

-   firewall persistence

-   full reboot/recovery verification

Final nginx exposure:

``` text

10.10.10.2:80 -> nginx

```

---

*\*\****\*\*\\\\***\*\\\\\****## Phase 7 -- DNAT / Service Publishing\\\\***\*\\\\\****\*\***\*\*

*\*\****\*\*\\\\***\*\\\\\****### Service Publishing Architecture\\\\***\*\\\\\****\*\***\*\*

-   Published the nginx service running on Alpine-Lab-02 through

    Alpine-Lab-01.

-   Established the external-to-internal mapping:

```text

192.168.122.252:8080 -> DNAT -> 10.10.10.2:80

```

-   Connected routing, NAT, firewalling, conntrack, backend host

    firewalling, and nginx into one complete packet flow.

Final architecture:

```text

Upstream client

192.168.122.1

      |

      | TCP :8080

      v

Alpine-Lab-01

192.168.122.252

Router / Firewall

      |

      | DNAT

      v

Alpine-Lab-02

10.10.10.2:80

      |

      v

    nginx

```

*\*\****\*\*\\\\***\*\\\\\****### DNAT / PREROUTING\\\\***\*\\\\\****\*\***\*\*

-   Added destination NAT on Alpine-Lab-01:

```bash

sudo iptables -t nat -A PREROUTING   

  -i eth0   

  -p tcp   

  --dport 8080   

  -j DNAT   

  --to-destination 10.10.10.2:80

```

-   Verified that DNAT occurs before the routing decision.

-   Demonstrated that without DNAT, traffic addressed to

    `192.168.122.252:8080` remains local to Alpine-Lab-01 and follows

    the INPUT path.

-   Demonstrated that after DNAT changes the destination to

    `10.10.10.2:80`, Linux routes the packet toward `eth1` and the

    packet follows the FORWARD path.

Established:

```text

No DNAT:

eth0 -> PREROUTING -> local routing decision -> INPUT

With DNAT:

eth0 -> PREROUTING/DNAT -> route to 10.10.10.2 -> FORWARD -> eth1

```

*\*\****\*\*\\\\***\*\\\\\****### Published-Service FORWARD Rule\\\\***\*\\\\\****\*\***\*\*

-   Added an explicit NEW forwarding rule on Alpine-Lab-01:

```bash

sudo iptables -I FORWARD 3   

  -i eth0   

  -o eth1   

  -p tcp   

  -d 10.10.10.2   

  --dport 80   

  -m conntrack --ctstate NEW   

  -j ACCEPT

```

-   Preserved the existing stateful `RELATED,ESTABLISHED` return rule.

-   Reinforced that DNAT does not automatically authorize traffic

    through the filter table.

Final FORWARD model:

```text

RELATED,ESTABLISHED                              ACCEPT

eth1 -> eth0 + source 10.10.10.0/24 + NEW       ACCEPT

eth0 -> eth1 + destination 10.10.10.2:80 + NEW  ACCEPT

rate-limited FORWARD-DROP logging

policy DROP

```

-   Reinforced iptables rule ordering and that the `LOG` target is

    non-terminating.

*\*\****\*\*\\\\***\*\\\\\****### Alpine-Lab-02 Backend Firewall\\\\***\*\\\\\****\*\***\*\*

-   Confirmed that DNAT changes the destination but preserves the

    original source address.

-   Observed published traffic arriving at Alpine-Lab-02 as:

```text

SRC 192.168.122.1

DST 10.10.10.2:80

```

-   Added a scoped INPUT rule for upstream published HTTP traffic:

```bash

sudo iptables -I INPUT 4   

  -p tcp   

  -s 192.168.122.0/24   

  -d 10.10.10.2   

  --dport 80   

  -m conntrack --ctstate NEW   

  -j ACCEPT

```

-   Kept the existing internal HTTP rule for `10.10.10.0/24`.

-   Reinforced that publishing a service through a router does not

    bypass the backend host firewall.

*\*\****\*\*\\\\***\*\\\\\****### End-to-End HTTP Verification\\\\***\*\\\\\****\*\***\*\*

-   Verified the published service from Linux Mint:

```bash

curl -I http\\\\://192.168.122.252:8080/

```

-   Received:

```text

HTTP/1.1 200 OK

Server: nginx

Content-Type: text/html

Content-Length: 896

```

-   Confirmed nginx itself continues listening on `10.10.10.2:80`;

    no process needs to listen locally on Alpine-Lab-01 TCP/8080.

*\*\****\*\*\\\\***\*\\\\\****### conntrack and Reverse NAT\\\\***\*\\\\\****\*\***\*\*

-   Installed `conntrack-tools` on Alpine-Lab-01.

-   Inspected live TCP connection tracking entries.

-   Observed the original client tuple:

```text

192.168.122.1  \:CLIENT_PORT -> 192.168.122.252:8080

```

-   Observed the backend/reply tuple:

```text

10.10.10.2:80 -> 192.168.122.1  \:CLIENT_PORT

```

-   Verified conntrack maintains the NAT relationship and allows the

    return traffic to be reverse-translated so that the client continues

    seeing `192.168.122.252:8080`.

-   Observed `[ASSURED]` and `TIME_WAIT` states.

*\*\****\*\*\\\\***\*\\\\\****### Nmap Through the Published Endpoint\\\\***\*\\\\\****\*\***\*\*

-   Tested service discovery against TCP/8080.

-   Observed normal Nmap host discovery report the target as apparently

    down because discovery probes were blocked by the restrictive

    firewall.

-   Repeated the test with:

```bash

nmap -Pn -sV -p 8080 192.168.122.252

```

-   Identified:

```text

8080/tcp open  http  nginx

```

-   Confirmed Nmap can fingerprint the application reached through DNAT

    without revealing the private backend address by itself.

*\*\****\*\*\\\\***\*\\\\\****### tcpdump Packet-Flow Analysis\\\\***\*\\\\\****\*\***\*\*

-   Captured the external/client-facing flow on Alpine-Lab-01 `eth0`:

```bash

sudo tcpdump -ni eth0 'tcp port 8080'

```

-   Observed:

```text

192.168.122.1  \:CLIENT_PORT -> 192.168.122.252:8080

192.168.122.252:8080 -> 192.168.122.1  \:CLIENT_PORT

```

-   Captured the internal/backend-facing flow on `eth1`:

```bash

sudo tcpdump -ni eth1 'host 10.10.10.2 and tcp port 80'

```

-   Observed:

```text

192.168.122.1  \:CLIENT_PORT -> 10.10.10.2:80

10.10.10.2:80 -> 192.168.122.1  \:CLIENT_PORT

```

-   Used the `any` pseudo-interface to observe both forms of the same

    connection simultaneously.

-   Directly visualized DNAT on the request path and reverse NAT on the

    reply path.

*\*\****\*\*\\\\***\*\\\\\****### TCP Flags and Sequence Analysis\\\\***\*\\\\\****\*\***\*\*

-   Reviewed TCP flags visible in tcpdump:

```text

[S]   SYN

[S.]  SYN + ACK

[.]   ACK

[P.]  PSH + ACK

[F.]  FIN + ACK

[R.]  RST + ACK

```

-   Traced a complete TCP three-way handshake.

-   Traced HTTP request and response data.

-   Traced graceful FIN/ACK connection closure.

-   Studied TCP sequence-number ranges and acknowledgement numbers.

-   Verified that:

```text

seq 1:84 length 83

```

means 83 payload bytes.

-   Verified ACK semantics: an ACK value represents the next sequence

    number expected.

-   Reinforced that SYN and FIN each consume one TCP sequence number.

-   Connected repeated identical SYN sequence numbers with TCP

    retransmissions.

*\*\****\*\*\\\\***\*\\\\\****### Controlled Failure Experiments\\\\***\*\\\\\****\*\***\*\*

Four major failure modes were deliberately reproduced.

*\*\****\*\*\\\\***\*\\\\\****1. nginx stopped\\\\***\*\\\\\****\*\***\*\*

-   Stopped nginx while networking and firewall rules remained valid.

-   Observed the SYN reach Alpine-Lab-02.

-   Observed Alpine-Lab-02 return `RST+ACK`.

-   Observed reverse NAT convert the reply source back to

    `192.168.122.252:8080`.

Diagnostic signature:

```text

SYN -> RST

```

Interpretation:

```text

Network path works, but no service is listening.

```

*\*\****\*\*\\\\***\*\\\\\****2. Alpine-Lab-02 published HTTP INPUT rule removed\\\\***\*\\\\\****\*\***\*\*

-   Left nginx running.

-   Observed SYN packets on both Lab-01 `eth0` and `eth1`.

-   Observed no reply from Alpine-Lab-02.

-   Observed repeated SYN retransmissions and timeout behavior.

Interpretation:

```text

DNAT works

FORWARD works

packet reaches backend

backend INPUT DROP blocks it

```

*\*\****\*\*\\\\***\*\\\\\****3. Alpine-Lab-01 published-service FORWARD rule removed\\\\***\*\\\\\****\*\***\*\*

-   Kept DNAT active.

-   Observed repeated SYNs arriving on `eth0`.

-   Observed no corresponding packet leaving `eth1`.

Interpretation:

```text

packet reaches router

DNAT/routing path exists

FORWARD DROP blocks transmission to backend

```

*\*\****\*\*\\\\***\*\\\\\****4. DNAT rule removed\\\\***\*\\\\\****\*\***\*\*

-   Removed the PREROUTING DNAT rule while keeping the remaining service

    configuration intact.

-   Observed repeated SYNs only on `eth0`.

-   Confirmed that the destination remained `192.168.122.252:8080`.

-   Established that the packet was therefore considered local and

    followed the INPUT chain rather than FORWARD.

Interpretation:

```text

No DNAT -> local destination -> INPUT -> DROP

```

Final troubleshooting matrix:

```text

DNAT missing:

    SYN on eth0 only

    packet follows INPUT

FORWARD allow missing:

    SYN on eth0 only

    translated flow blocked before eth1

Backend INPUT allow missing:

    SYN on eth0 and eth1

    no reply

    repeated retransmissions

nginx stopped:

    SYN reaches backend

    RST returns

Everything working:

    SYN / SYN-ACK / ACK

    HTTP 200

    clean FIN / ACK close

```

*\*\****\*\*\\\\***\*\\\\\****### Persistence and Recovery\\\\***\*\\\\\****\*\***\*\*

-   Saved the Alpine-Lab-01 DNAT and FORWARD rules to

    `/etc/iptables/rules-save`.

-   Saved the Alpine-Lab-02 published HTTP INPUT rule to

    `/etc/iptables/rules-save`.

-   Verified the relevant rules directly in the saved files.

-   Performed a reboot/recovery test.

-   Confirmed nginx, firewall rules, DNAT, forwarding, and HTTP

    publishing returned correctly.

-   During controlled failure testing, removed only runtime rules and

    deliberately avoided overwriting the known-good persistent files.

-   Restored every temporarily removed runtime rule.

-   Performed the final external HTTP test:

```bash

curl -I http\\\\://192.168.122.252:8080/

```

-   Received final `HTTP/1.1 200 OK`.

*\*\****\*\*\\\\***\*\\\\\****### Documentation\\\\***\*\\\\\****\*\***\*\*

-   Added `docs/12-service-publishing-dnat.md`.

-   Updated `README.md`.

-   Updated `LAB_STATUS.md`.

-   Updated `CHANGELOG.md`.

*\*\****\*\*\\\\***\*\\\\\****### Phase 7 Complete\\\\***\*\\\\\****\*\***\*\*

Final published service:

```text

192.168.122.252:8080

        |

        | DNAT

        v

10.10.10.2:80

        |

        v

      nginx

```

Phase 7 now connects:

-   routing

-   DNAT

-   stateful forwarding

-   backend host firewalling

-   nginx

-   conntrack

-   reverse NAT

-   Nmap service discovery

-   tcpdump packet analysis

-   TCP connection analysis

-   controlled failure troubleshooting

-   persistence and reboot recovery

---

---

*\*\****\*\*## Phase 8 -- Storage Management\*\***\*\*

*\*\****\*\*### Storage Inspection\*\***\*\*

\- Inspected existing VM disks with \`virsh domblklist\` and \`virsh domblkinfo\`.

\- Inspected qcow2 virtual capacity, physical allocation, and metadata with \`qemu-img info\`.

\- Compared host file views using \`ls\`, \`du\`, and libvirt volume information.

\- Inspected the \`default\` libvirt storage pool and its volumes.

\- Reinforced the storage stack:

\`\`\`text

qcow2 image

    ↓

QEMU / libvirt

    ↓

virtual block device

    ↓

partition

    ↓

filesystem

    ↓

mount point

\`\`\`

*\*\****\*\*### qcow2 and RAW\*\***\*\*

\- Created temporary RAW and qcow2 images.

\- Verified that RAW images can also be sparse.

\- Compared virtual size with actual host allocation.

\- Used \`qemu-io\` to write safely to qcow2 virtual contents.

\- Checked qcow2 integrity with \`qemu-img check\`.

\- Converted RAW to qcow2.

\- Practiced:

\`\`\`bash

qemu-img info IMAGE

qemu-img create -f FORMAT IMAGE SIZE

qemu-img check IMAGE

qemu-img resize IMAGE SIZE

qemu-img convert -f INPUT -O OUTPUT SOURCE DESTINATION

\`\`\`

*\*\****\*\*### Additional Alpine-Lab-02 Data Disk\*\***\*\*

\- Created a 2 GiB qcow2 libvirt volume:

\`\`\`text

Alpine-Lab-02-data.qcow2

\`\`\`

\- Attached it persistently to Alpine-Lab-02.

\- Observed that libvirt target names and guest \`/dev/vdX\` enumeration did not match.

\- Reinforced that persistent configuration should not rely solely on \`/dev/vda\` or \`/dev/vdb\`.

*\*\****\*\*### Partition, Filesystem, and Persistent Mount\*\***\*\*

\- Partitioned the new data disk using \`fdisk\`.

\- Created an ext4 filesystem with \`mkfs.ext4\`.

\- Identified its filesystem UUID with \`blkid\`.

\- Created the mount point:

\`\`\`text

/srv/data

\`\`\`

\- Added the filesystem to \`/etc/fstab\` using UUID.

\- Verified \`mount -a\`.

\- Rebooted Alpine-Lab-02 and confirmed automatic mounting.

\- Created persistent test data and verified it after reboot.

*\*\****\*\*### Storage Resize\*\***\*\*

\- Expanded \`Alpine-Lab-02-data.qcow2\` from 2 GiB to 3 GiB with \`qemu-img resize\`.

\- Verified that increasing qcow2 capacity did not automatically enlarge the guest partition or filesystem.

\- Expanded the MBR partition while preserving its original starting sector.

\- Installed the required ext4 resize utility on Alpine.

\- Expanded the mounted ext4 filesystem online with:

\`\`\`bash

sudo resize2fs /dev/vda1

\`\`\`

Final result:

\`\`\`text

qcow2 volume     3 GiB

partition        3 GiB

ext4             \~2.9 GiB

mount point      /srv/data

\`\`\`

\- Verified that existing data remained intact throughout the resize.

*\*\****\*\*### Storage Troubleshooting\*\***\*\*

\- Deliberately introduced an incorrect \`/srv/data\` UUID in \`/etc/fstab\`.

\- Observed:

\`\`\`text

mount: /srv/data: can't find UUID=...

\`\`\`

\- Used \`blkid\` to identify the correct filesystem UUID.

\- Restored the valid \`/etc/fstab\`.

\- Verified recovery with \`mount -a\`, \`df\`, and the persistent test file.

*\*\****\*\*### Documentation\*\***\*\*

\- Added \`docs/13-storage-management.md\`.

\- Updated \`README.md\`.

\- Updated \`LAB\_STATUS.md\`.

\- Updated \`CHANGELOG.md\`.

*\*\****\*\*### Phase 8 Complete\*\***\*\*

Phase 8 established practical understanding of:

\- libvirt storage pools and volumes

\- qcow2 and RAW images

\- sparse allocation

\- guest virtual block devices

\- partitioning

\- ext4 filesystems

\- persistent UUID-based mounts

\- multi-layer virtual disk resizing

\- storage troubleshooting and recovery

Final Alpine-Lab-02 data-storage path:

\`\`\`text

Alpine-Lab-02-data.qcow2 (3 GiB)

        ↓

QEMU / VirtIO

        ↓

guest data disk

        ↓

/dev/vda1 (3 GiB)

        ↓

ext4 (\~2.9 GiB)

        ↓

/srv/data

        ↓

persistent via UUID in /etc/fstab

\`\`\`

---

\*\*\*\*

---

**## Phase 9 -- Containers & Automation**

**### Podman Fundamentals**

\- Installed and used Podman on Linux Mint.

\- Practiced container image and lifecycle management with \`run\`, \`ps\`, \`exec\`, \`stop\`, \`start\`, and \`rm\`.

\- Demonstrated that containers share the host kernel while providing isolated userspace and processes.

\- Introduced rootless and daemonless container operation.

\- Distinguished images from running/stopped container instances.

**### Port Publishing and nginx**

\- Ran nginx in a container.

\- Published container TCP/80 through Linux Mint TCP/8080.

\- Verified HTTP access with \`curl\`.

\- Reinforced the mapping:

\`\`\`text

0.0.0.0:8080 -> container:80/tcp

\`\`\`

\- Established that \`0.0.0.0\` publishes on all host IPv4 interfaces.

**### Bind Mounts and Named Volumes**

\- Mounted \`\~/container-web\` into nginx as a read-only bind mount.

\- Verified host file changes immediately appeared inside the served container content.

\- Verified bind-mounted data survived container deletion.

\- Created the Podman-managed \`web-data\` named volume.

\- Recreated an nginx container with the same named volume and verified persistent content.

Established:

\`\`\`text

container = disposable runtime

volume    = persistent data

\`\`\`

**### Logs and Environment Variables**

\- Inspected container output with \`podman logs\`.

\- Observed nginx and Flask HTTP requests and status codes.

\- Passed runtime configuration using \`-e\`.

\- Reinforced the separation between reusable images and runtime configuration.

**### Custom Container Images**

\- Created a custom nginx image with a \`Containerfile\`.

\- Built images using:

\`\`\`bash

podman build -t IMAGE .

\`\`\`

\- Created a custom Python application image based on \`python:3.13-slim\`.

\- Installed Flask and \`psycopg[binary]\` during the image build.

**### Container Networking**

\- Created the dedicated \`app-net\` network.

\- Connected containers to the custom network.

\- Verified container-to-container communication using container names.

\- Demonstrated that service discovery avoids relying on changing container IP addresses.

\- Observed the Podman network using the \`10.89.0.0/24\` range during the lab.

**### PostgreSQL and Persistent Database Storage**

\- Created the \`postgres-data\` named volume.

\- Started PostgreSQL on \`app-net\`.

\- Configured the database through environment variables:

\`\`\`text

POSTGRES\_DB=labdb

POSTGRES\_USER=labuser

POSTGRES\_PASSWORD=labpass

\`\`\`

\- Used the PostgreSQL 18 storage layout with the persistent volume mounted at \`/var/lib/postgresql\`.

\- Deliberately kept PostgreSQL TCP/5432 internal to \`app-net\`.

\- Verified local \`psql\` access inside the database container.

\- Verified TCP database access from another container using \`postgres-db\` as the hostname.

\- Created the \`notes\` table and inserted persistent data.

\- Deleted and recreated the PostgreSQL container.

\- Verified the database row survived through the \`postgres-data\` volume.

**### Flask + PostgreSQL Multi-Container Application**

\- Built a Flask application that reads database configuration from environment variables.

\- Connected Flask to PostgreSQL through \`app-net\` using the hostname \`postgres-db\`.

\- Published Flask TCP/5000 through Linux Mint:

\`\`\`text

0.0.0.0:5000 -> python-app:5000/tcp

\`\`\`

Final application path:

\`\`\`text

Client

  ↓

Linux Mint :5000

  ↓

Podman port publishing

  ↓

python-app / Flask

  ↓

app-net

  ↓

postgres-db :5432

  ↓

postgres-data

\`\`\`

**### CRUD API**

Implemented and verified:

\`\`\`text

GET    /notes       -> READ

POST   /notes       -> CREATE

PUT    /notes/\<id>  -> UPDATE

DELETE /notes/\<id>  -> DELETE

\`\`\`

Observed HTTP results included:

\`\`\`text

GET     -> 200

POST    -> 201

PUT     -> 200

DELETE  -> 204

\`\`\`

A complete create/read/update/delete cycle was performed successfully.

**### KVM Lab Integration**

\- Published the Flask API through Linux Mint's \`virbr10\` address:

\`\`\`text

10.10.10.254:5000

\`\`\`

\- Verified successful API access from Alpine-Lab-01.

\- Verified successful API access from Alpine-Lab-02.

\- Reinforced that \`10.10.10.254\` is directly reachable from the Alpine guests because all are on \`10.10.10.0/24\`.

\- Distinguished the KVM subnet from Podman's internal \`app-net\`: the container itself is not directly on \`10.10.10.0/24\`; Mint publishes the service into that subnet.

**### Security Observations**

\- Confirmed Flask was reachable through all Mint IPv4 interfaces because of \`0.0.0.0:5000\`.

\- Identified that the lab CRUD API has no authentication.

\- Noted that database credentials are supplied through environment variables.

\- Observed Flask's expected Werkzeug development-server warning.

\- Kept PostgreSQL unexposed to the Mint host by not publishing TCP/5432.

\- Deliberately deferred production WSGI deployment and further application hardening as outside the phase scope.

**### Basic Automation**

Created \`start-stack.sh\`:

\`\`\`sh

\#!/bin/sh

echo "Starting PostgreSQL..."

podman start postgres-db

echo "Starting Python API..."

podman start python-app

echo

echo "Running containers:"

podman ps

\`\`\`

\- Made the script executable.

\- Stopped the application containers.

\- Successfully restarted the stack with \`./start-stack.sh\`.

\- Verified the complete application path afterward with \`curl\`.

**### Documentation**

\- Added \`docs/14-containers-automation.md\`.

\- Updated \`README.md\`.

\- Updated \`LAB\_STATUS.md\`.

\- Updated \`CHANGELOG.md\`.

**### Phase 9 Complete**

Phase 9 established practical understanding of:

\- container lifecycle and images

\- rootless and daemonless operation

\- port publishing

\- bind mounts and named volumes

\- container logging

\- environment variables

\- Containerfiles and custom image builds

\- container networking and name resolution

\- persistent PostgreSQL storage

\- multi-container application architecture

\- CRUD API operation

\- KVM-to-container service access

\- basic shell automation

Final container application:

\`\`\`text

KVM lab / Linux Mint clients

          ↓

10.10.10.254:5000

          ↓

     python-app

       Flask

          ↓

       app-net

          ↓

    postgres-db

          ↓

   postgres-data

\`\`\`

---

---

## Phase 10 -- VM Provisioning & Automation

### Clean Alpine Template

- Created a new reusable `Alpine-Template-v2` VM.
- Installed Alpine Linux as the base operating system.
- Deliberately avoided booting the installed system before sealing the template.
- Installed:
  - `cloud-init`
  - `cloud-init-openrc`
  - `cloud-init-datasource-nocloud`
  - `e2fsprogs-extra`
- Enabled the cloud-init OpenRC services:
  - `cloud-init-local`
  - `cloud-init`
  - `cloud-config`
  - `cloud-final`
- Restricted cloud-init datasource discovery to NoCloud.
- Removed previous cloud-init state from `/var/lib/cloud/*`.
- Removed SSH host keys so new instances generate unique host identities.

### NoCloud Seed Troubleshooting

- Initially attempted the standard external NoCloud seed-device workflow.
- Tested ISO9660 `CIDATA` seed media.
- Tested the seed as a virtual CD-ROM.
- Tested the seed as a virtio block device.
- Rebuilt the template to eliminate contamination from previous experiments.
- Tested VFAT `CIDATA` seed media.
- Confirmed Alpine/OpenRC cloud-init continued failing to mount the external seed during early boot.
- Identified the seed-device mount as the real blocker rather than cloud-init user-data syntax or libvirt networking.

The unsuccessful approach was:

```text
user-data + meta-data
        ↓
external CIDATA seed
        ↓
virtual CD-ROM / virtio disk
        ↓
early-boot mount failure
        ↓
NoCloud configuration not applied
```

### Direct NoCloud Seed Injection

- Replaced the external seed device with direct filesystem injection.
- Used `qemu-nbd` to expose a qcow2 instance disk as `/dev/nbd0`.
- Refreshed its partition table with `partprobe`.
- Identified the Alpine root filesystem as `/dev/nbd0p3`.
- Mounted the guest root filesystem from the Linux Mint host.
- Created:

```text
/var/lib/cloud/seed/nocloud/
```

- Injected:

```text
user-data
meta-data
```

directly into the guest filesystem before first boot.

Established the working provisioning path:

```text
qcow2 overlay
      ↓
qemu-nbd
      ↓
/dev/nbd0
      ↓
mount /dev/nbd0p3
      ↓
/var/lib/cloud/seed/nocloud/
      ↓
user-data + meta-data
      ↓
first boot
      ↓
DataSourceNoCloud
```

### qcow2 Copy-on-Write Provisioning

- Created VM instance disks as qcow2 overlays backed by:

```text
/var/lib/libvirt/images/Alpine-Template-v2.qcow2
```

- Used:

```bash
qemu-img create \
  -f qcow2 \
  -F qcow2 \
  -b TEMPLATE \
  INSTANCE.qcow2
```

- Reinforced the distinction between:
  - the reusable base/template image
  - per-VM copy-on-write changes
- Preserved the template unchanged while creating independent VM instances.
- Connected Phase 10 provisioning directly with the qcow2/storage concepts practiced in Phase 8.

### cloud-init Instance Metadata

- Generated unique `meta-data` for every VM.
- Used the VM name as the source for:
  - `instance-id`
  - `local-hostname`
- Used Bash lowercase expansion:

```bash
${VM_NAME,,}
```

Example:

```yaml
instance-id: alpine-auto-04
local-hostname: alpine-auto-04
```

### Automated User and SSH Provisioning

- Configured cloud-init to create the `airgon` user.
- Added the user to `adm` and `wheel`.
- Configured `/bin/ash` as the login shell.
- Installed the SSH Ed25519 public key through `ssh_authorized_keys`.
- Disabled SSH password authentication with:

```yaml
ssh_pwauth: false
```

- Disabled root access through the cloud-init configuration.

### Locked Account Troubleshooting

- Diagnosed a case where the SSH public key existed correctly but OpenSSH still rejected the user.
- Found the decisive SSH server message:

```text
User airgon not allowed because account is locked
```

- Confirmed the cloud-init-created account had a locked password field.
- Verified manually that setting a valid password unlocked the account and immediately allowed SSH public-key authentication.
- Updated `user-data` with:

```yaml
lock_passwd: false
passwd: '<SHA-512 crypt hash>'
```

- Kept `ssh_pwauth: false`, so the password hash exists to keep the Unix account unlocked rather than to enable SSH password login.
- Learned to generate SHA-512 crypt hashes with:

```bash
openssl passwd -6
```

### cloud-init Configuration Cleanup

- Disabled automatic partition growth for the current template:

```yaml
growpart:
  mode: 'off'

resize_rootfs: false
```

- Quoted `'off'` to prevent YAML from interpreting the value as a boolean.
- Verified cloud-init reached:

```text
status: done
```

with:

```text
DataSourceNoCloud [seed=/var/lib/cloud/seed/nocloud]
```

- Observed a non-fatal Alpine packaging warning related to `keys_to_console`.
- Confirmed the warning did not prevent hostname configuration, user creation, SSH-key installation, or SSH access.

### Manual Provisioning Proof

- Successfully provisioned `Alpine-Auto-02` using the direct NoCloud injection method.
- Verified:
  - hostname configuration
  - cloud-init completion
  - NoCloud datasource detection
  - `airgon` account creation
  - SSH public-key login
- Established that no manual guest configuration was required after first boot.

### Bash Provisioning Automation

Created:

```text
~/kvm-cloud-init/provision-vm.sh
```

The script automates:

```text
VM name
   ↓
safety checks
   ↓
cloud-init metadata
   ↓
qcow2 overlay
   ↓
qemu-nbd attachment
   ↓
guest root mount
   ↓
NoCloud injection
   ↓
sync / unmount / NBD disconnect
   ↓
virt-install --import
   ↓
DHCP discovery
   ↓
SSH command
```

### Script Safety Checks

- Added:

```bash
set -e
```

so command failures stop the script.

- Added a required VM-name argument.
- Added a check preventing creation if the libvirt domain already exists.
- Added a check preventing overwrite if the target qcow2 disk already exists.

### Failure-Safe Cleanup

- Introduced state variables:

```bash
NBD_CONNECTED=false
MOUNTED=false
```

- Created a `cleanup()` function.
- Registered it with:

```bash
trap cleanup EXIT
```

- Ensured temporary resources are released if the script exits unexpectedly.
- Updated the state after normal unmount/disconnect so the EXIT trap does not repeat successful cleanup.
- Verified the final script produces only the expected NBD disconnect during a successful run.

### Automated VM Creation

- Used `virt-install --import` to define and start the prepared instance.
- Configured:
  - 1024 MiB RAM
  - 2 vCPUs
  - virtio qcow2 disk
  - libvirt `default` network
  - virtio NIC
  - no graphical console requirement
- Eliminated the operating-system installation step for each new VM.

### Automatic DHCP Discovery

- Added a loop that waits for a libvirt DHCP lease.
- Polls up to 30 times with a two-second delay.
- Uses:

```bash
virsh net-dhcp-leases default
```

- Matches the exact VM hostname with `awk`.
- Extracts the IP address while removing the CIDR suffix.

Final matching logic:

```bash
awk -v host="${VM_NAME,,}" \
  '$6 == host {split($5,a,"/"); print a[1]}'
```

- Prints the assigned address and ready-to-use SSH command.

### Final Automated Verification

Ran:

```bash
./provision-vm.sh Alpine-Auto-04
```

The script completed:

```text
VM Alpine-Auto-04 created successfully.
IP address: 192.168.122.237
SSH:
  ssh airgon@192.168.122.237
```

Then verified:

```bash
ssh airgon@192.168.122.237
```

Result:

```text
Welcome to Alpine!

alpine-auto-04:~$
```

This confirmed the complete workflow:

```text
template
   ↓
qcow2 overlay
   ↓
offline cloud-init injection
   ↓
automated libvirt deployment
   ↓
first-boot provisioning
   ↓
DHCP
   ↓
SSH-ready VM
```

### Documentation

- Added `docs/15-vm-provisioning-automation.md`.
- Updated `README.md`.
- Updated `LAB_STATUS.md`.
- Updated `CHANGELOG.md`.

### Phase 10 Complete

Phase 10 established practical understanding of:

- golden/template VM images
- cloud-init
- NoCloud
- first-boot instance configuration
- qcow2 copy-on-write overlays
- backing images
- offline guest customization
- `qemu-nbd`
- automatic hostname generation
- automatic user provisioning
- SSH key provisioning
- Bash scripting
- functions and `trap`
- defensive cleanup
- `virt-install --import`
- libvirt DHCP discovery
- repeatable VM provisioning

Final automated provisioning architecture:

```text
Alpine-Template-v2.qcow2
          ↓
qcow2 overlay
          ↓
qemu-nbd
          ↓
NoCloud seed injection
          ↓
virt-install --import
          ↓
cloud-init
          ↓
libvirt DHCP
          ↓
SSH-ready Alpine VM
```

---

## Next -- Phase 11

Phase 11 will be the **Final Integration / Capstone** phase.

Planned work:

- integrate the infrastructure built throughout the lab
- perform final verification and recovery testing
- clean and review documentation
- create/finalize the architecture diagram
- perform the final README and lab-status review
- create final snapshots or known-good checkpoints where appropriate
- create the final Git checkpoint
