# Service Publishing with DNAT

## Phase 7 — Publishing an Internal nginx Service Through Alpine-Lab-01

This phase extends the KVM lab from internal routing and firewalling into **service publishing**.

The objective is to make the nginx web server running on **Alpine-Lab-02 (`10.10.10.2:80`)** reachable from the upstream `192.168.122.0/24` network through **Alpine-Lab-01 (`192.168.122.252:8080`)**.

The final published service is:

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
nginx
```

The phase was deliberately built and tested layer by layer so that the complete packet path could be observed:

```text
client
  -> eth0
  -> PREROUTING / DNAT
  -> routing decision
  -> FORWARD
  -> eth1
  -> Alpine-Lab-02 INPUT
  -> nginx
  -> return traffic
  -> conntrack / reverse NAT
  -> client
```

---

## 1. Starting Architecture

### Alpine-Lab-01 — router/firewall

Interfaces:

```text
eth0  192.168.122.252/24
eth1  10.10.10.1/24
```

Routing:

```text
default via 192.168.122.1 dev eth0
10.10.10.0/24 dev eth1
192.168.122.0/24 dev eth0
```

IPv4 forwarding is enabled:

```bash
sysctl net.ipv4.ip_forward
```

Expected:

```text
net.ipv4.ip_forward = 1
```

The existing outbound NAT rule provides Internet access to the isolated LAN:

```bash
-A POSTROUTING -s 10.10.10.0/24 -o eth0 -j MASQUERADE
```

The Phase 5 FORWARD baseline is stateful:

```text
RELATED,ESTABLISHED                         ACCEPT
NEW eth1 -> eth0 from 10.10.10.0/24        ACCEPT
FORWARD-DROP logging
policy DROP
```

### Alpine-Lab-02 — nginx server

Address:

```text
10.10.10.2/24
```

nginx listens on TCP port 80 and is managed through OpenRC.

Useful checks:

```bash
sudo rc-service nginx status
sudo ss -ltnp | grep ':80'
```

Lab-02 uses a restrictive INPUT firewall. HTTP from the isolated LAN was already allowed:

```text
10.10.10.0/24 -> 10.10.10.2:80/tcp NEW ACCEPT
```

---

## 2. DNAT Concept

**DNAT** means **Destination Network Address Translation**.

For this lab the desired translation is:

```text
192.168.122.252:8080
        |
        | DNAT
        v
10.10.10.2:80
```

The source address is not changed by this DNAT rule.

For example:

```text
Before DNAT:

SRC 192.168.122.1:CLIENT_PORT
DST 192.168.122.252:8080


After DNAT:

SRC 192.168.122.1:CLIENT_PORT
DST 10.10.10.2:80
```

This is important because the backend server can still see the original client source address.

### Service publishing vs port forwarding vs DNAT

These terms describe related ideas at different levels:

- **Service publishing** — architectural goal: expose an internal service through another network endpoint.
- **Port forwarding** — practical description: traffic arriving at one address/port is forwarded elsewhere.
- **DNAT** — the netfilter/NAT mechanism used here to rewrite the destination.

---

## 3. Baseline Test Before DNAT

From Linux Mint:

```bash
curl -v http://192.168.122.252:8080/
```

Before DNAT existed, the request failed.

The packet still had destination:

```text
192.168.122.252:8080
```

Because `192.168.122.252` belongs to Alpine-Lab-01 itself, Linux treated the packet as **local traffic**.

The path was therefore:

```text
eth0
  |
PREROUTING
  |
no destination translation
  |
routing decision
  |
destination is local
  |
INPUT
  |
DROP
```

The INPUT policy counter increased while FORWARD did not.

This established an important rule:

> A packet addressed to the router itself traverses INPUT unless an earlier operation such as DNAT changes the routing decision.

---

## 4. Add the DNAT Rule

On Alpine-Lab-01:

```bash
sudo iptables -t nat -A PREROUTING \
  -i eth0 \
  -p tcp \
  --dport 8080 \
  -j DNAT \
  --to-destination 10.10.10.2:80
```

Meaning:

```text
table:       nat
chain:       PREROUTING
input:       eth0
protocol:    TCP
destination port: 8080
action:      DNAT
new destination: 10.10.10.2:80
```

Verification:

```bash
sudo iptables -t nat -L PREROUTING -n -v --line-numbers
```

After another client attempt, the DNAT counter increased.

The packet now became:

```text
192.168.122.1:CLIENT_PORT -> 10.10.10.2:80
```

before Linux performed its routing decision.

Because `10.10.10.2` is reached through `eth1`, the packet was no longer local to Lab-01.

Its path changed from:

```text
INPUT
```

to:

```text
FORWARD
```

This is one of the central lessons of the phase:

> DNAT changes not only the destination fields but also the routing decision and therefore the firewall path.

---

## 5. Allow the Published Flow Through FORWARD

DNAT alone was insufficient.

After translation, the packet reached Lab-01's FORWARD chain, whose default policy is DROP.

The publishing rule was added:

```bash
sudo iptables -I FORWARD 3 \
  -i eth0 \
  -o eth1 \
  -p tcp \
  -d 10.10.10.2 \
  --dport 80 \
  -m conntrack --ctstate NEW \
  -j ACCEPT
```

The intended FORWARD order became:

```text
1  RELATED,ESTABLISHED                                  ACCEPT
2  NEW eth1 -> eth0 source 10.10.10.0/24               ACCEPT
3  NEW eth0 -> eth1 destination 10.10.10.2 tcp/80      ACCEPT
4  FORWARD-DROP logging
   policy DROP
```

### Rule ordering lesson

The rule was initially appended after the logging rule.

Because a `LOG` target records a packet but does **not** terminate traversal, a packet could be logged as `FORWARD-DROP` and then accepted by a later rule.

The allow rule was therefore moved above the final logging rule.

This reinforced an important iptables principle:

> Rule order matters, and LOG is normally non-terminating.

---

## 6. Backend Firewall Requirement

Even after Lab-01 allowed forwarding, Lab-02 initially still dropped the connection.

Its existing HTTP rule allowed only:

```text
source 10.10.10.0/24 -> destination 10.10.10.2:80
```

But DNAT changes the **destination**, not the source.

The forwarded packet still arrived at Lab-02 as:

```text
SRC 192.168.122.1
DST 10.10.10.2:80
```

Therefore it did not match the internal-LAN HTTP rule.

A scoped rule was added on Alpine-Lab-02:

```bash
sudo iptables -I INPUT 4 \
  -p tcp \
  -s 192.168.122.0/24 \
  -d 10.10.10.2 \
  --dport 80 \
  -m conntrack --ctstate NEW \
  -j ACCEPT
```

The backend now explicitly accepts published HTTP connections originating from the upstream libvirt network.

This demonstrates another key principle:

> DNAT does not automatically bypass or open the destination host's firewall.

---

## 7. Successful End-to-End Publishing

From Linux Mint:

```bash
curl -v http://192.168.122.252:8080/
```

Result:

```text
Connected to 192.168.122.252 port 8080
HTTP/1.1 200 OK
Server: nginx
```

The complete path was now:

```text
Linux Mint / upstream client
192.168.122.1
        |
        | TCP -> 192.168.122.252:8080
        v
Alpine-Lab-01 eth0
        |
        v
nat PREROUTING
        |
        | DNAT
        | 192.168.122.252:8080
        |        ->
        | 10.10.10.2:80
        v
routing decision
        |
        v
FORWARD ACCEPT
        |
        v
Alpine-Lab-01 eth1
        |
        v
Alpine-Lab-02
        |
        v
INPUT ACCEPT
        |
        v
nginx :80
```

---

## 8. Persistence

The Phase 7 rules were saved after the working configuration was confirmed.

### Alpine-Lab-01

```bash
sudo iptables-save | sudo tee /etc/iptables/rules-save > /dev/null
```

Verification:

```bash
sudo grep -E '8080|10\.10\.10\.2.*80' /etc/iptables/rules-save
```

Relevant persistent rules:

```text
-A FORWARD -d 10.10.10.2/32 -i eth0 -o eth1 -p tcp -m tcp --dport 80 -m conntrack --ctstate NEW -j ACCEPT
-A PREROUTING -i eth0 -p tcp -m tcp --dport 8080 -j DNAT --to-destination 10.10.10.2:80
```

### Alpine-Lab-02

```bash
sudo iptables-save | sudo tee /etc/iptables/rules-save > /dev/null
```

Verification:

```bash
sudo grep '192.168.122.0/24' /etc/iptables/rules-save
```

Relevant rule:

```text
-A INPUT -s 192.168.122.0/24 -d 10.10.10.2/32 -p tcp -m tcp --dport 80 -m conntrack --ctstate NEW -j ACCEPT
```

A reboot test confirmed that the DNAT, FORWARD, backend firewall, and nginx configuration survived restart.

After reboot:

```bash
curl -I http://192.168.122.252:8080/
```

returned:

```text
HTTP/1.1 200 OK
Server: nginx
```

---

## 9. Conntrack and Reverse NAT

`conntrack-tools` was installed on Alpine-Lab-01:

```bash
sudo apk add conntrack-tools
```

Verification:

```bash
which conntrack
conntrack -V
```

A connection was then generated from Mint and inspected:

```bash
sudo conntrack -L -p tcp
```

A representative entry was:

```text
tcp ... TIME_WAIT \
src=192.168.122.1 dst=192.168.122.252 sport=48072 dport=8080 \
src=10.10.10.2 dst=192.168.122.1 sport=80 dport=48072 \
[ASSURED]
```

This exposes both views of the connection.

Client/original direction:

```text
192.168.122.1:48072
        ->
192.168.122.252:8080
```

Backend/reply direction:

```text
10.10.10.2:80
        ->
192.168.122.1:48072
```

Conntrack remembers the NAT relationship for the connection.

When the backend replies:

```text
10.10.10.2:80 -> client
```

the NAT/conntrack state performs the corresponding reverse translation so that the client sees:

```text
192.168.122.252:8080 -> client
```

The client therefore continues to perceive a single consistent TCP endpoint.

### Conntrack states observed

`[ASSURED]` indicates that conntrack observed bidirectional traffic and considers the flow established/confirmed.

`TIME_WAIT` was observed after successful short HTTP requests. This is a normal TCP state after connection closure.

---

## 10. What the Client Can Discover

The client sees the published endpoint:

```text
192.168.122.252:8080
```

It does not normally learn the private backend address from TCP/IP alone.

The backend may still leak information at the application layer, for example through:

- HTTP headers
- redirects
- error messages
- generated URLs
- DNS names
- TLS certificates
- application configuration mistakes

In this lab nginx reveals:

```text
Server: nginx
```

but not:

```text
10.10.10.2
```

---

## 11. Nmap Service Detection Through DNAT

An initial scan:

```bash
nmap -sV -p 8080 192.168.122.252
```

reported:

```text
Host seems down.
```

This occurred because host discovery was blocked by the restrictive firewall.

Repeating with `-Pn`:

```bash
nmap -Pn -sV -p 8080 192.168.122.252
```

produced:

```text
PORT     STATE SERVICE VERSION
8080/tcp open  http    nginx
```

`-Pn` tells Nmap to skip host discovery and treat the target as online.

This is an important interpretation:

> Nmap determined that interacting with `192.168.122.252:8080` behaves like nginx. It did not prove that nginx is locally listening on Lab-01.

Indeed, Lab-01 does not need a process listening on TCP 8080. Netfilter intercepts and translates the traffic before local socket delivery.

---

## 12. Packet Capture with tcpdump

Packet capture made the DNAT transformation directly observable.

### External side — eth0

On Alpine-Lab-01:

```bash
sudo tcpdump -ni eth0 'tcp port 8080'
```

A successful request showed traffic such as:

```text
192.168.122.1.33580 > 192.168.122.252.8080
192.168.122.252.8080 > 192.168.122.1.33580
```

This is the **client-facing view**.

### Internal side — eth1

```bash
sudo tcpdump -ni eth1 'host 10.10.10.2 and tcp port 80'
```

The corresponding internal flow appeared as:

```text
192.168.122.1.58928 > 10.10.10.2.80
10.10.10.2.80 > 192.168.122.1.58928
```

This is the **backend-facing view**.

The two captures demonstrate:

```text
EXTERNAL VIEW                     INTERNAL VIEW

client -> 192.168.122.252:8080    client -> 10.10.10.2:80
          DNAT ----------------->

client <- 192.168.122.252:8080    client <- 10.10.10.2:80
          <---- reverse NAT
```

### Capture both interfaces

The `any` pseudo-interface was also useful:

```bash
sudo tcpdump -ni any 'tcp port 8080 or (host 10.10.10.2 and tcp port 80)'
```

The warning:

```text
any: That device doesn't support promiscuous mode
```

is expected for the Linux cooked `any` capture interface and did not prevent the capture.

With a working connection, each significant packet could be seen entering or leaving both sides of the router.

---

## 13. TCP Flags Observed

The captures provided a useful review of TCP flags.

Common tcpdump notation:

```text
[S]   SYN
[S.]  SYN + ACK
[.]   ACK
[P.]  PSH + ACK
[F.]  FIN + ACK
[R]   RST
[R.]  RST + ACK
```

Important TCP flags:

| Flag | Name | Purpose |
|---|---|---|
| S | SYN | Begin/synchronize a TCP connection |
| A / `.` | ACK | Acknowledge received sequence data |
| P | PSH | Promptly deliver received data to the application |
| F | FIN | Gracefully close one direction of a TCP connection |
| R | RST | Reset/abort/refuse a TCP connection |
| U | URG | Urgent pointer is significant |
| E | ECE | ECN congestion indication |
| W | CWR | Congestion Window Reduced |

A normal successful HTTP connection showed:

```text
SYN
SYN-ACK
ACK
PSH-ACK request
ACK
PSH-ACK response
ACK
FIN-ACK
FIN-ACK
ACK
```

---

## 14. TCP Sequence and Acknowledgement Numbers

TCP tracks a byte stream rather than simply numbering packets.

A captured HTTP request contained:

```text
seq 1:84
length 83
```

because:

```text
84 - 1 = 83 bytes
```

The server then replied:

```text
ack 84
```

meaning:

> Bytes through 83 have been received; sequence 84 is expected next.

A full GET response contained:

```text
seq 1:1128
length 1127
```

and the client acknowledged:

```text
ack 1128
```

The HTTP response body itself had:

```text
Content-Length: 896
```

while the TCP payload was 1127 bytes because TCP carried the HTTP headers plus the 896-byte body.

### SYN and FIN consume sequence numbers

SYN and FIN each consume one sequence number even when:

```text
length 0
```

For example:

```text
SYN seq 1570777123
```

is acknowledged as:

```text
ack 1570777124
```

Similarly:

```text
FIN seq 84
```

is acknowledged with:

```text
ack 85
```

A useful simplified rule is:

```text
next ACK =
    sequence number
  + payload length
  + 1 if SYN is present
  + 1 if FIN is present
```

---

## 15. Controlled Failure Experiments

The working publishing path was deliberately broken one layer at a time.

These experiments created a practical troubleshooting model.

### Experiment A — nginx stopped

nginx was stopped on Alpine-Lab-02:

```bash
sudo rc-service nginx stop
```

Nothing was listening on port 80.

A simultaneous capture showed:

```text
eth0 In:
client -> 192.168.122.252:8080   SYN

eth1 Out:
client -> 10.10.10.2:80          SYN

eth1 In:
10.10.10.2:80 -> client          RST+ACK

eth0 Out:
192.168.122.252:8080 -> client   RST+ACK
```

The actual captured reset included:

```text
Flags [R.]
```

This proved:

- DNAT worked.
- FORWARD worked.
- Lab-02 firewall allowed the packet.
- The packet reached Lab-02's TCP stack.
- No application was listening.

The client received an immediate connection refusal.

Diagnostic signature:

```text
SYN -> RST
```

usually means:

> The destination is reachable, but the TCP port is closed/not listening.

nginx was then restarted.

---

### Experiment B — Lab-02 INPUT rule removed

The published HTTP INPUT allow rule was temporarily removed from Lab-02 while nginx remained running.

The capture showed repeated:

```text
eth0 In:
client -> 192.168.122.252:8080   SYN

eth1 Out:
client -> 10.10.10.2:80          SYN
```

with **no reply**.

The same sequence number was retransmitted repeatedly.

This proved:

- DNAT worked.
- Lab-01 FORWARD worked.
- The packet left Lab-01 through eth1.
- The packet reached Lab-02.
- Lab-02's INPUT policy silently dropped it before nginx could receive it.

Diagnostic signature:

```text
SYN
SYN
SYN
...
```

with no response.

This is characteristic of silent filtering/DROP.

The Lab-02 rule was restored afterward.

---

### Experiment C — Lab-01 FORWARD rule removed

The publishing-specific FORWARD rule was temporarily removed from Lab-01.

The capture showed only:

```text
eth0 In:
client -> 192.168.122.252:8080   SYN
```

repeated several times.

There was no:

```text
eth1 Out -> 10.10.10.2:80
```

This proved:

- The packet reached Lab-01.
- DNAT could translate the destination.
- Routing selected the internal network.
- FORWARD policy DROP stopped the packet before transmission through eth1.

Diagnostic signature:

```text
eth0 sees SYN
eth1 does not
```

The FORWARD rule was restored afterward.

---

### Experiment D — DNAT rule removed

Finally, the DNAT PREROUTING rule itself was temporarily removed.

The capture again showed repeated:

```text
eth0 In:
192.168.122.1:PORT -> 192.168.122.252:8080   SYN
```

and nothing on eth1.

Although this capture resembles the FORWARD failure, the internal reason is different.

Without DNAT:

```text
destination = 192.168.122.252
```

Lab-01 recognizes that address as its own.

The path therefore becomes:

```text
eth0
  |
PREROUTING
  |
no DNAT
  |
routing decision
  |
LOCAL
  |
INPUT
  |
DROP
```

With DNAT present but FORWARD blocked:

```text
eth0
  |
PREROUTING / DNAT
  |
destination = 10.10.10.2
  |
routing decision
  |
FORWARD
  |
DROP
```

This demonstrates why firewall counters are useful in addition to packet captures: two failures can look similar on an interface capture while traversing different netfilter chains.

The DNAT rule was restored after the experiment.

---

## 16. Failure Matrix

| Failure | eth0 | eth1 | Response | Main interpretation |
|---|---|---|---|---|
| DNAT missing | SYN visible | No forwarded SYN | Silence | Packet remains local and reaches Lab-01 INPUT |
| Lab-01 FORWARD allow missing | SYN visible | No forwarded SYN | Silence | Forwarding firewall blocks translated packet |
| Lab-02 INPUT allow missing | SYN visible | SYN visible | Silence | Backend host firewall drops packet |
| nginx stopped | SYN visible | SYN visible | RST | Network path works; no service is listening |
| Everything working | Full TCP flow | Full TCP flow | HTTP 200 | Published service operational |

This matrix provides a repeatable troubleshooting strategy:

1. Is the packet arriving at the router?
2. Is DNAT occurring?
3. Does it leave through the expected interface?
4. Does the backend respond?
5. Does the TCP handshake complete?
6. Does the application return valid data?

---

## 17. Final Working Rules

### Alpine-Lab-01 — DNAT

```bash
sudo iptables -t nat -A PREROUTING \
  -i eth0 \
  -p tcp \
  --dport 8080 \
  -j DNAT \
  --to-destination 10.10.10.2:80
```

### Alpine-Lab-01 — forwarding

```bash
sudo iptables -I FORWARD 3 \
  -i eth0 \
  -o eth1 \
  -p tcp \
  -d 10.10.10.2 \
  --dport 80 \
  -m conntrack --ctstate NEW \
  -j ACCEPT
```

Return packets are accepted by the existing stateful rule:

```text
RELATED,ESTABLISHED ACCEPT
```

### Alpine-Lab-02 — published HTTP input

```bash
sudo iptables -I INPUT 4 \
  -p tcp \
  -s 192.168.122.0/24 \
  -d 10.10.10.2 \
  --dport 80 \
  -m conntrack --ctstate NEW \
  -j ACCEPT
```

### Persistence

On each machine after confirming the intended runtime rules:

```bash
sudo iptables-save | sudo tee /etc/iptables/rules-save > /dev/null
```

---

## 18. Final Validation

After all destructive experiments, every removed rule was restored.

Final test from Linux Mint:

```bash
curl -I http://192.168.122.252:8080/
```

Result:

```text
HTTP/1.1 200 OK
Server: nginx
Content-Type: text/html
Content-Length: 896
```

The published nginx service was therefore restored to a healthy state.

---

## 19. Key Lessons

1. **DNAT rewrites the destination before the routing decision.**
2. **Changing the destination can change a packet from INPUT traffic into FORWARD traffic.**
3. **DNAT does not automatically create a firewall exception.**
4. **Both the router firewall and backend firewall must permit the intended flow.**
5. **DNAT preserves the original source address unless source NAT is also performed.**
6. **Conntrack maintains state and enables the corresponding reverse NAT.**
7. **The client sees the published endpoint, not normally the private backend address.**
8. **Nmap can fingerprint the backend application through a DNAT endpoint without discovering its internal IP.**
9. **tcpdump can show the same flow before and after translation by observing different router interfaces.**
10. **TCP flags, sequence numbers, acknowledgements, and retransmissions provide precise clues about where a connection is failing.**
11. **SYN -> RST suggests a reachable host with a closed/unavailable service.**
12. **Repeated SYNs with no response suggest filtering, dropping, or another silent path failure.**
13. **Seeing a SYN on eth0 but not eth1 points toward the router's translation/routing/FORWARD path.**
14. **Seeing the SYN on both eth0 and eth1 but no response shifts investigation toward the backend host.**
15. **Controlled failure testing is a powerful way to learn and validate network architecture.**

---

## Phase 7 Result

The lab now publishes an internal nginx service through the router/firewall:

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

The configuration is persistent, reboot-tested, packet-captured, service-scanned, conntrack-observed, and tested under multiple deliberate failure conditions.

**Phase 7 — Service Publishing with DNAT: COMPLETE.**
