# Linux Services & nginx

## Phase 6 --- Linux Services & Service Exposure

This phase introduces Linux service management and real application
exposure using **Alpine-Lab-02** as a web server. The goal is to connect
service management, processes, sockets, HTTP, logs, permissions, network
binding, firewalling, and persistence.

------------------------------------------------------------------------

## 1. Lab Role

**Alpine-Lab-02**

-   Hostname: `alpine-lab-02`
-   Lab IP: `10.10.10.2/24`
-   Gateway: `10.10.10.1`
-   Web server: nginx
-   HTTP service: TCP/80
-   nginx binding: `10.10.10.2:80`
-   Host firewall: iptables
-   Init/service manager: OpenRC

Relevant topology:

``` text
Linux Mint              Alpine-Lab-01             Alpine-Lab-02
10.10.10.254            10.10.10.1                10.10.10.2
     │                  Router / DNS                    │
     └────────────── 10.10.10.0/24 ────────────────────┘
                                                   nginx :80
```

------------------------------------------------------------------------

## 2. Service Management with OpenRC

A Linux **service** is the operating-system-managed definition used to
start, stop, restart, enable, and inspect some functionality.

A **daemon** is the actual background process providing that
functionality.

Useful OpenRC commands:

``` bash
rc-status
rc-service <service> status
rc-service <service> start
rc-service <service> stop
rc-service <service> restart
rc-update add <service> default
rc-update del <service> default
rc-update show
```

Important distinction:

``` text
rc-service start/stop   → controls the service NOW
rc-update add/del       → controls what happens on future boots
```

A service can therefore be:

-   installed but stopped;
-   running but not enabled at boot;
-   enabled at boot but currently stopped;
-   running and enabled.

This was demonstrated using `crond` before configuring nginx.

------------------------------------------------------------------------

## 3. Installing nginx

On Alpine-Lab-02:

``` bash
sudo apk add nginx
```

The package provides, among other files:

``` text
/etc/nginx/nginx.conf
/etc/init.d/nginx
/etc/conf.d/nginx
/usr/sbin/nginx
/var/lib/nginx/html/
```

The main configuration loads server definitions from:

``` nginx
include /etc/nginx/http.d/*.conf;
```

The default server configuration is:

``` text
/etc/nginx/http.d/default.conf
```

Useful validation command:

``` bash
sudo nginx -t
```

OpenRC's nginx service also validates the configuration when appropriate
before starting or reloading nginx.

------------------------------------------------------------------------

## 4. Processes and Listening Sockets

After starting nginx:

``` bash
sudo rc-service nginx start
```

Processes can be inspected with:

``` bash
ps
```

nginx uses a privileged master process and unprivileged worker
processes.

Listening sockets can be inspected with:

``` bash
ss -lntp
```

Initially nginx listened on wildcard addresses:

``` text
0.0.0.0:80
[::]:80
```

`0.0.0.0:80` means nginx accepts TCP/80 connections addressed to any
local IPv4 address.

`[::]:80` is the equivalent IPv6 wildcard listener.

------------------------------------------------------------------------

## 5. Creating the Lab Web Page

The nginx document root is:

``` text
/var/lib/nginx/html
```

A test page was created as:

``` text
/var/lib/nginx/html/lab.html
```

with:

``` html
<!DOCTYPE html>
<html>
<head>
    <title>KVM Virtualization Lab</title>
</head>
<body>
    <h1>Alpine-Lab-02</h1>
    <p>My first nginx web server is working!</p>
</body>
</html>
```

The file was created using a heredoc and `tee`:

``` bash
sudo tee /var/lib/nginx/html/lab.html >/dev/null <<'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>KVM Virtualization Lab</title>
</head>
<body>
    <h1>Alpine-Lab-02</h1>
    <p>My first nginx web server is working!</p>
</body>
</html>
EOF
```

For static content, changing the HTML file does not require an nginx
reload.

------------------------------------------------------------------------

## 6. HTTP Testing with curl

Useful forms:

``` bash
curl http://10.10.10.2/lab.html
```

Displays the response body.

``` bash
curl -i http://10.10.10.2/lab.html
```

Displays response headers and body.

``` bash
curl -I http://10.10.10.2/lab.html
```

Uses an HTTP `HEAD` request and displays headers only.

Successful response:

``` text
HTTP/1.1 200 OK
Server: nginx
Content-Type: text/html
Content-Length: 179
```

Remote tests from Linux Mint (`10.10.10.254`) and Alpine-Lab-03
(`10.10.10.3`) both returned HTTP `200 OK`.

------------------------------------------------------------------------

## 7. nginx Logs

Important log files:

``` text
/var/log/nginx/access.log
/var/log/nginx/error.log
```

### Access log

Records requests handled by nginx.

Examples included:

``` text
10.10.10.254 ... "HEAD /lab.html HTTP/1.1" 200 ...
10.10.10.3   ... "HEAD /lab.html HTTP/1.1" 200 ...
```

This proved that nginx could distinguish requests from Linux Mint and
Alpine-Lab-03.

Relevant HTTP status codes observed:

-   `200 OK` --- request succeeded.
-   `304 Not Modified` --- browser used its cached copy.
-   `404 Not Found` --- requested resource did not exist.

### Error log

The error log showed underlying problems such as requests for missing
files, including a missing favicon and a deliberately requested
nonexistent page.

Important troubleshooting distinction:

``` text
access.log → what requests nginx handled
error.log  → server/filesystem problems associated with requests
```

If a firewall drops a packet before nginx receives it, nginx produces no
access-log entry for that attempt.

------------------------------------------------------------------------

## 8. Document-Root Permissions

Inspection showed:

``` text
/var/lib/nginx
```

was owned by `nginx:nginx` with restrictive directory permissions.

Path permissions were inspected with:

``` bash
namei -l /var/lib/nginx/html/lab.html
```

and, when privileges were required:

``` bash
sudo namei -l /var/lib/nginx/html/lab.html
```

The nginx worker processes run as user `nginx`.

Reading the page as nginx succeeded:

``` bash
sudo -u nginx cat /var/lib/nginx/html/lab.html
```

Attempting to append to it as nginx failed:

``` bash
sudo -u nginx sh -c 'echo test >> /var/lib/nginx/html/lab.html'
```

with `Permission denied`.

This demonstrated **least privilege**:

``` text
nginx needs to read static content
nginx does not need to modify that content
```

Directory execute permission (`x`) means **traverse/search**, which is
required to reach files farther down a pathname.

------------------------------------------------------------------------

## 9. Binding nginx to the Lab Address

The original configuration used:

``` nginx
listen 80 default_server;
listen [::]:80 default_server;
```

This was changed to:

``` nginx
server {
    listen 10.10.10.2:80 default_server;
    # listen [::]:80 default_server;

    root /var/lib/nginx/html;
    index index.html;
}
```

Configuration validation:

``` bash
sudo nginx -t
```

A graceful reload initially retained the old wildcard listening sockets.
A full restart was used:

``` bash
sudo rc-service nginx restart
```

Afterward:

``` bash
sudo ss -lntp | grep ':80'
```

showed only:

``` text
10.10.10.2:80
```

Tests proved:

``` text
127.0.0.1:80  → no listener
10.10.10.2:80 → nginx → HTTP 200
```

### Binding vs routing vs firewalling

These are separate layers:

``` text
Binding   → WHERE does the service listen?
Routing   → CAN the client find a network path?
Firewall  → IS the traffic permitted?
```

Binding nginx to `10.10.10.2:80` means the service is specifically
exposed through that local IPv4 address rather than every local IPv4
address.

------------------------------------------------------------------------

## 10. DNS and Service Names

The lab DNS server is dnsmasq on Alpine-Lab-01 (`10.10.10.1`).

From a lab machine able to use that DNS information:

``` bash
curl -I http://alpine-lab-02/lab.html
```

successfully reached nginx.

The FQDN is:

``` text
alpine-lab-02.lab.local
```

Linux Mint could reach `10.10.10.2` directly but could not normally
resolve the lab hostname because Mint's `systemd-resolved` configuration
used external DNS servers such as `1.1.1.1`.

A direct DNS query from Mint proved that the lab DNS server itself
worked:

``` bash
dig @10.10.10.1 alpine-lab-02.lab.local
```

returned:

``` text
alpine-lab-02.lab.local. ... IN A 10.10.10.2
```

Therefore:

``` text
routing to Lab-02       → works
lab DNS                 → works
nginx                   → works
Mint DNS selection      → does not normally query Lab-01 for lab.local
```

No split-DNS configuration was added to Mint because it was unnecessary
for the objectives of this phase.

### Note about `.local`

`dig` warned that `.local` is reserved for Multicast DNS (mDNS). The
existing `lab.local` domain was retained to avoid unnecessary lab
reconfiguration, but this is recorded as a design lesson for future
environments.

------------------------------------------------------------------------

## 11. Host Firewall on Alpine-Lab-02

Unlike Alpine-Lab-01, which acts as the lab router/firewall,
Alpine-Lab-02 now also has its own **host firewall**.

Installed:

``` bash
sudo apk add iptables
```

Initial state:

``` text
INPUT   ACCEPT
FORWARD ACCEPT
OUTPUT  ACCEPT
```

with no custom rules.

This provided a clean baseline for firewall experiments.

------------------------------------------------------------------------

## 12. DROP vs REJECT vs No Listener

A temporary DROP rule was added:

``` bash
sudo iptables -A INPUT -p tcp --dport 80 -j DROP
```

From Linux Mint:

``` bash
curl -I --max-time 5 http://10.10.10.2/lab.html
```

timed out.

iptables counters increased, while nginx produced no new access-log
entry.

This proved:

``` text
client → firewall DROP → nginx never receives request
```

The DROP rule was then replaced with:

``` bash
sudo iptables -A INPUT -p tcp --dport 80 \
    -j REJECT --reject-with tcp-reset
```

The client failed almost immediately because the firewall explicitly
returned a TCP reset.

Finally, with the firewall block removed and nginx stopped:

``` bash
sudo rc-service nginx stop
```

the client also received an immediate connection failure because nothing
was listening on TCP/80.

Comparison:

``` text
nginx listening + ACCEPT  → HTTP 200
nginx listening + DROP    → timeout
nginx listening + REJECT  → immediate connection failure
nginx stopped             → immediate connection failure
```

This demonstrates that similar client symptoms can have different
causes. Troubleshooting therefore requires inspecting multiple layers,
including:

``` bash
ss -lntp
iptables -L -n -v
rc-service nginx status
```

------------------------------------------------------------------------

## 13. Final Host-Firewall Policy

The permanent INPUT rules were built **before** changing the default
policy to DROP, avoiding accidental lockout.

Final logical order:

``` text
1. loopback                         ACCEPT
2. ESTABLISHED,RELATED              ACCEPT
3. NEW SSH from 10.10.10.0/24      ACCEPT
4. NEW HTTP from 10.10.10.0/24     ACCEPT
------------------------------------------
   INPUT policy                     DROP
```

Commands:

``` bash
sudo iptables -A INPUT -i lo -j ACCEPT

sudo iptables -A INPUT \
    -m conntrack --ctstate ESTABLISHED,RELATED \
    -j ACCEPT

sudo iptables -A INPUT \
    -p tcp \
    -s 10.10.10.0/24 \
    --dport 22 \
    -m conntrack --ctstate NEW \
    -j ACCEPT

sudo iptables -A INPUT \
    -p tcp \
    -s 10.10.10.0/24 \
    --dport 80 \
    -m conntrack --ctstate NEW \
    -j ACCEPT

sudo iptables -P INPUT DROP
```

The final ruleset was verified with:

``` bash
sudo iptables -L INPUT -n -v --line-numbers
```

Result:

``` text
INPUT policy DROP
lo                                  ACCEPT
RELATED,ESTABLISHED                 ACCEPT
10.10.10.0/24 → TCP/22 NEW         ACCEPT
10.10.10.0/24 → TCP/80 NEW         ACCEPT
```

### Why this order?

iptables evaluates rules from top to bottom and acts on the first
matching rule.

Loopback traffic is accepted immediately. Established connections are
then handled efficiently by conntrack. New connections are permitted
only for explicitly exposed services.

The default DROP **policy is not another numbered rule**. It is the
fallback applied when no explicit rule matches.

------------------------------------------------------------------------

## 14. Firewall Validation

From Linux Mint:

``` bash
ssh airgon@10.10.10.2
```

succeeded.

HTTP:

``` bash
curl -I --max-time 5 http://10.10.10.2/lab.html
```

returned:

``` text
HTTP/1.1 200 OK
```

An unapproved TCP port was tested:

``` bash
nc -vz -w 3 10.10.10.2 9999
```

and:

``` bash
curl --max-time 3 http://10.10.10.2:9999
```

Both timed out because TCP/9999 matched no ACCEPT rule and therefore
fell through to the INPUT DROP policy.

The resulting security boundary is:

``` text
TCP/22    → allowed from 10.10.10.0/24
TCP/80    → allowed from 10.10.10.0/24
TCP/9999  → DROP
other unsolicited INPUT → DROP
```

------------------------------------------------------------------------

## 15. Firewall Persistence

Installing iptables also installed `iptables-openrc`.

Before persistence was configured:

``` text
rc-service iptables status       → stopped
rc-update show | grep iptables   → no entry
/etc/iptables/                   → empty
```

The live rules were saved with:

``` bash
sudo iptables-save | sudo tee /etc/iptables/rules-save >/dev/null
```

The resulting file contained:

``` text
*filter
:INPUT DROP
:FORWARD ACCEPT
:OUTPUT ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -s 10.10.10.0/24 -p tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT
-A INPUT -s 10.10.10.0/24 -p tcp --dport 80 -m conntrack --ctstate NEW -j ACCEPT
COMMIT
```

iptables was enabled in the default OpenRC runlevel:

``` bash
sudo rc-update add iptables default
```

verified with:

``` bash
rc-update show | grep iptables
```

### iptables is not a firewall daemon

The firewall rules themselves live in the Linux kernel.

Therefore:

``` text
iptables rules in kernel       → active firewall state
/etc/iptables/rules-save       → persistent stored rules
OpenRC iptables service        → restores those rules at boot
```

Unlike nginx or sshd, iptables does not require a continuously running
userspace daemon to enforce the loaded rules.

------------------------------------------------------------------------

## 16. Persistence Test

Alpine-Lab-02 was rebooted.

After reboot:

``` bash
sudo iptables -L INPUT -n -v --line-numbers
```

showed the complete ruleset restored automatically:

``` text
Chain INPUT (policy DROP)

1  ACCEPT  lo
2  ACCEPT  RELATED,ESTABLISHED
3  ACCEPT  TCP/22 NEW from 10.10.10.0/24
4  ACCEPT  TCP/80 NEW from 10.10.10.0/24
```

A new SSH connection incremented the TCP/22 NEW counter.

nginx service persistence, its TCP/80 listener, HTTP access, and
firewall persistence were all verified after reboot.

**Phase 6 service and firewall configuration therefore survives
reboot.**

------------------------------------------------------------------------

## 17. Firewall Logging Decision

No persistent firewall logging was added to Alpine-Lab-02.

Firewall logging was studied extensively on Alpine-Lab-01 during Phase
5. Repeating the complete logging configuration here would add little to
the objectives of this phase.

Lab-02 firewall behavior can currently be diagnosed with:

``` bash
sudo iptables -L INPUT -n -v --line-numbers
sudo ss -lntp
sudo tail /var/log/nginx/access.log
sudo tail /var/log/nginx/error.log
```

If later DNAT troubleshooting requires determining whether forwarded
traffic reached Lab-02 but was dropped locally, a temporary LOG rule can
be added.

------------------------------------------------------------------------

## 18. Final Security Model

Alpine-Lab-02 now applies multiple independent controls:

``` text
CLIENT
   │
   ▼
NETWORK / ROUTING
   │
   ▼
HOST FIREWALL
iptables INPUT
   │
   ├── SSH :22 from lab network
   ├── HTTP :80 from lab network
   └── everything else DROP
   │
   ▼
SERVICE BINDING
nginx → 10.10.10.2:80
   │
   ▼
APPLICATION / STATIC CONTENT
/var/lib/nginx/html
```

nginx determines **where the web service listens**.

iptables determines **which incoming traffic may reach services on the
host**.

Routing determines **whether a client has a path to the host**.

These mechanisms complement rather than replace one another.

------------------------------------------------------------------------

## 19. Key Commands

  ----------------------------------------------------------------------------------------------------------
  Purpose                             Command
  ----------------------------------- ----------------------------------------------------------------------
  Service status                      `rc-service nginx status`

  Start nginx                         `sudo rc-service nginx start`

  Stop nginx                          `sudo rc-service nginx stop`

  Restart nginx                       `sudo rc-service nginx restart`

  Enable nginx at boot                `sudo rc-update add nginx default`

  Validate nginx config               `sudo nginx -t`

  Inspect sockets                     `ss -lntp`

  HTTP request                        `curl http://10.10.10.2/lab.html`

  Headers only                        `curl -I http://10.10.10.2/lab.html`

  Access log                          `sudo tail /var/log/nginx/access.log`

  Error log                           `sudo tail /var/log/nginx/error.log`

  Firewall rules                      `sudo iptables -L INPUT -n -v --line-numbers`

  Firewall rule syntax                `sudo iptables -S`

  Save firewall                       `sudo iptables-save \| sudo tee /etc/iptables/rules-save >/dev/null`

  Enable firewall restore             `sudo rc-update add iptables default`

  Direct lab DNS query                `dig @10.10.10.1 alpine-lab-02.lab.local`
  ----------------------------------------------------------------------------------------------------------

------------------------------------------------------------------------

## 20. Phase 6 Result

At the end of Phase 6, Alpine-Lab-02 is a persistent, deliberately
exposed web server:

-   nginx installed and managed through OpenRC;
-   custom static web page served successfully;
-   HTTP requests and status codes inspected with curl;
-   access and error logging understood;
-   nginx process privileges and filesystem permissions examined;
-   nginx bound specifically to `10.10.10.2:80`;
-   remote access verified from the isolated lab network;
-   DNS name resolution tested;
-   host firewall installed and configured;
-   SSH and HTTP explicitly permitted from `10.10.10.0/24`;
-   unsolicited inbound traffic dropped by default;
-   DROP, REJECT, and no-listener behavior compared experimentally;
-   nginx and firewall persistence verified by reboot;
-   firewall logging deliberately omitted to avoid duplicating Phase 5.

The server is now ready for the next phase: **publishing the internal
HTTP service through Alpine-Lab-01 using DNAT/port forwarding**.
