# Lab Status

This file tracks the current state of the KVM Virtualization Lab.

---

# Current Phase

## Phase 10 --- VM Provisioning & Automation

Status:

✅ **Complete**

Current checkpoint:

**A clean Alpine template (`Alpine-Template-v2`) now supports repeatable VM provisioning through qcow2 copy-on-write overlays, cloud-init/NoCloud, offline seed injection with `qemu-nbd`, and automated deployment with `virt-install --import`. The final `provision-vm.sh` workflow generates instance metadata, injects cloud-init configuration, safely manages NBD/mount cleanup, starts the VM, waits for libvirt DHCP, discovers the assigned IP, and prints the SSH command. `Alpine-Auto-04` was provisioned end-to-end and accepted SSH key authentication without manual guest configuration.**

---

# Current Infrastructure

```text
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
            │   • Persistent ext4 Data Storage (/srv/data)
            │   • 3 GiB qcow2 Data Volume
            │   • SSH Server
            │   • 10.10.10.2/24
            │   • Gateway/DNS: 10.10.10.1
            │
            ├── Alpine-Lab-03
            │   • Internal Client
            │   • 10.10.10.3/24
            │   • Gateway/DNS: 10.10.10.1
            │
            └── Automated VM Provisioning
                • Alpine-Template-v2
                • qcow2 overlays
                • cloud-init / NoCloud
                • qemu-nbd offline injection
                • virt-install --import
                • automatic DHCP discovery
```

---

# Alpine-Lab-01 Firewall State

Phase 5 remains the security baseline, extended in Phase 7 with an explicit published-service FORWARD rule and DNAT.

```text
INPUT    DROP
FORWARD  DROP
OUTPUT   ACCEPT
```

Persisted INPUT model:

```text
INPUT DROP
│
├── lo                                             ACCEPT
├── RELATED,ESTABLISHED                            ACCEPT
├── eth1 + 10.10.10.0/24 + TCP/22 NEW             ACCEPT  SSH
├── eth1 + UDP/67                                  ACCEPT  DHCP
├── eth1 + 10.10.10.0/24 + UDP/53                 ACCEPT  DNS
├── eth1 + 10.10.10.0/24 + TCP/53 NEW             ACCEPT  DNS
├── eth1 + 10.10.10.0/24 + ICMP echo-request      ACCEPT
└── rate-limited IPTABLES-DROP logging
```

Persisted FORWARD model:

```text
FORWARD DROP
│
├── RELATED,ESTABLISHED                            ACCEPT
├── eth1 -> eth0 + source 10.10.10.0/24 + NEW     ACCEPT
├── eth0 -> eth1 + 10.10.10.2 + TCP/80 NEW        ACCEPT
└── rate-limited FORWARD-DROP logging
```

NAT:

```text
POSTROUTING:
10.10.10.0/24 -> eth0 -> MASQUERADE

PREROUTING:
eth0 TCP/8080 -> DNAT -> 10.10.10.2:80
```

Rules are persisted in `/etc/iptables/rules-save`. Firewall logging is handled by BusyBox `klogd` and `syslogd` and stored in `/var/log/messages`.

---

# Alpine-Lab-02 Service State

## nginx

nginx is installed, running, boot-enabled, and verified after reboot.

Current exposure:

```text
Internal:
10.10.10.2:80 -> nginx

Published:
192.168.122.252:8080 -> DNAT -> 10.10.10.2:80 -> nginx
```

Current server definition:

```nginx
server {
    listen 10.10.10.2:80 default_server;
    # listen [::]:80 default_server;

    root /var/lib/nginx/html;
    index index.html;
}
```

Custom page:

```text
/var/lib/nginx/html/lab.html
```

Important logs:

```text
/var/log/nginx/access.log
/var/log/nginx/error.log
```

---

# Alpine-Lab-02 Host Firewall

Current policies:

```text
INPUT    DROP
FORWARD  ACCEPT
OUTPUT   ACCEPT
```

Current INPUT model:

```text
INPUT DROP
│
├── lo                                             ACCEPT
├── RELATED,ESTABLISHED                            ACCEPT
├── 10.10.10.0/24 + TCP/22 NEW                    ACCEPT  SSH
├── 192.168.122.0/24 -> 10.10.10.2 + TCP/80 NEW  ACCEPT  Published HTTP
└── 10.10.10.0/24 + TCP/80 NEW                    ACCEPT  Internal HTTP
```

The Phase 7 rule for `192.168.122.0/24` is required because DNAT changes the destination address but preserves the original client source address.

Rules are saved in `/etc/iptables/rules-save` and restored at boot through OpenRC.

---

# Phase 6 Experiments Completed

## OpenRC and services

✔ Distinguished installed, running, and boot-enabled states  
✔ Used `rc-service` for current runtime state  
✔ Used `rc-update` for boot behavior  
✔ Demonstrated the distinction with `crond`  
✔ Inspected OpenRC init scripts

## nginx and HTTP

✔ Installed nginx  
✔ Inspected nginx configuration structure  
✔ Started and managed nginx through OpenRC  
✔ Inspected master and worker processes  
✔ Inspected listening sockets with `ss`  
✔ Created `/var/lib/nginx/html/lab.html`  
✔ Verified HTTP `200 OK`  
✔ Used `curl`, `curl -i`, and `curl -I`

## Logging

✔ Inspected nginx access and error logs  
✔ Observed HTTP 200, 304, and 404 behavior  
✔ Verified missing resources in the error log  
✔ Confirmed firewall-dropped requests do not reach nginx access logging

## Permissions and least privilege

✔ Inspected path permissions with `namei -l`  
✔ Identified nginx master and worker users  
✔ Verified nginx can read static content  
✔ Verified nginx cannot modify the root-owned static page  
✔ Reinforced directory traverse (`x`) permission behavior

## Service binding

Initial listeners:

```text
0.0.0.0:80
[::]:80
```

Final listener:

```text
10.10.10.2:80
```

✔ Changed nginx to a specific IPv4 binding  
✔ Validated configuration with `nginx -t`  
✔ Investigated graceful reload/socket behavior  
✔ Used a full restart to obtain the intended socket state  
✔ Verified `127.0.0.1:80` no longer listens  
✔ Verified `10.10.10.2:80` continues serving HTTP

Key distinction:

```text
Binding   -> WHERE the service listens
Routing   -> CAN the client find a path
Firewall  -> IS the traffic permitted
```

## DNS experiment

✔ Verified Lab-01 dnsmasq resolves `alpine-lab-02.lab.local -> 10.10.10.2`  
✔ Verified direct DNS query from Mint with `dig @10.10.10.1`  
✔ Determined Mint normally uses its own external DNS configuration  
✔ Deliberately deferred split-DNS configuration  
✔ Recorded `.local` as an mDNS-reserved naming consideration

## Host firewall experiments

✔ Installed iptables on Alpine-Lab-02  
✔ Confirmed the initial ACCEPT policies  
✔ Tested temporary TCP/80 DROP  
✔ Observed timeout and increasing firewall counters  
✔ Confirmed nginx received no request  
✔ Tested `REJECT --reject-with tcp-reset`  
✔ Observed immediate client failure  
✔ Stopped nginx while TCP/80 was permitted  
✔ Observed immediate failure with no listener

Comparison:

```text
Listening + allowed   -> HTTP 200
Listening + DROP      -> timeout
Listening + REJECT    -> immediate failure
Not listening         -> immediate failure
```

## Final default-deny firewall

✔ Loopback ACCEPT  
✔ `RELATED,ESTABLISHED` ACCEPT  
✔ NEW SSH from `10.10.10.0/24` ACCEPT  
✔ NEW HTTP from `10.10.10.0/24` ACCEPT  
✔ INPUT policy changed to DROP  
✔ SSH verified  
✔ HTTP verified  
✔ TCP/9999 verified as blocked  
✔ Rules saved with `iptables-save`  
✔ iptables enabled in OpenRC  
✔ Full reboot performed  
✔ Firewall rules restored after reboot  
✔ nginx restored after reboot  
✔ TCP/80 listener restored after reboot  
✔ HTTP connectivity verified after reboot

---

# Phase 7 Experiments Completed

## DNAT and service publishing

✔ Published `192.168.122.252:8080` to `10.10.10.2:80`  
✔ Added DNAT in the `nat` table `PREROUTING` chain  
✔ Verified that DNAT occurs before the routing decision  
✔ Demonstrated how DNAT changes the path from INPUT to FORWARD  
✔ Added the explicit eth0-to-eth1 NEW HTTP FORWARD rule  
✔ Added the backend INPUT rule for published upstream traffic  
✔ Verified original source-address preservation

## conntrack and reverse NAT

✔ Installed `conntrack-tools`  
✔ Inspected the original and reply tuples  
✔ Observed `[ASSURED]` connections and `TIME_WAIT`  
✔ Verified that conntrack maintains the NAT relationship  
✔ Confirmed reverse NAT makes replies appear to come from `192.168.122.252:8080`

## Nmap and service discovery

✔ Demonstrated why normal host discovery could report the target down  
✔ Used `nmap -Pn -sV -p 8080 192.168.122.252`  
✔ Identified nginx through the published endpoint  
✔ Confirmed service fingerprinting does not reveal the backend `10.10.10.2` address by itself

## tcpdump and TCP analysis

✔ Captured the external flow on `eth0`  
✔ Captured the translated flow on `eth1`  
✔ Captured both sides simultaneously using `-i any`  
✔ Observed SYN, SYN-ACK, ACK, PSH, FIN, and RST behavior  
✔ Analyzed TCP sequence numbers, acknowledgement numbers, and payload lengths  
✔ Connected repeated SYNs to TCP retransmission behavior

## Controlled failure testing

```text
DNAT missing
    -> SYN visible on eth0 only
    -> destination remains local
    -> packet takes Lab-01 INPUT path

Lab-01 FORWARD allow missing
    -> SYN visible on eth0 only
    -> translated packet blocked before eth1

Lab-02 INPUT allow missing
    -> SYN visible on eth0 and eth1
    -> no reply
    -> silent retransmissions / timeout

nginx stopped
    -> SYN reaches Lab-02
    -> RST+ACK returns
    -> immediate connection refusal

Everything working
    -> full TCP handshake
    -> HTTP exchange
    -> HTTP 200
    -> clean FIN/ACK close
```

✔ Restored every deliberately removed rule after testing  
✔ Verified persistent rules remained intact  
✔ Final request returned `HTTP/1.1 200 OK`

---

# Phase 8 Experiments Completed

## Storage inspection and image formats

✔ Inspected VM block devices with `virsh domblklist` and `domblkinfo`  
✔ Inspected libvirt storage pools and volumes  
✔ Compared qcow2 and sparse RAW allocation  
✔ Used `qemu-img info`, `create`, `check`, `convert`, and `resize`

## Additional data disk

✔ Created `Alpine-Lab-02-data.qcow2`  
✔ Attached the volume persistently to Alpine-Lab-02  
✔ Investigated guest `vda` / `vdb` enumeration differences  
✔ Partitioned the data disk with `fdisk`  
✔ Created an ext4 filesystem  
✔ Mounted it persistently at `/srv/data` using UUID in `/etc/fstab`  
✔ Verified automatic mounting after reboot

## Storage resizing and troubleshooting

✔ Expanded the qcow2 volume from 2 GiB to 3 GiB  
✔ Expanded the partition separately while preserving its starting sector  
✔ Expanded ext4 online with `resize2fs`  
✔ Verified existing data survived the complete resize  
✔ Deliberately introduced an incorrect `/etc/fstab` UUID  
✔ Diagnosed the mount failure and restored the correct configuration

Final storage path:

```text
Alpine-Lab-02-data.qcow2 (3 GiB)
        ↓
QEMU / VirtIO
        ↓
guest data disk
        ↓
/dev/vda1 (3 GiB)
        ↓
ext4 (~2.9 GiB)
        ↓
/srv/data
        ↓
UUID in /etc/fstab
```

---

# Phase 9 Experiments Completed

## Podman fundamentals

✔ Container images and lifecycle  
✔ Rootless and daemonless concepts  
✔ `podman run`, `ps`, `exec`, `stop`, `start`, and `rm`  
✔ Port publishing  
✔ Bind mounts and named volumes  
✔ Container logs and environment variables  
✔ Custom images built with Containerfiles

## Networking and persistence

✔ Dedicated `app-net` network  
✔ Container-to-container name resolution  
✔ Persistent `postgres-data` volume  
✔ PostgreSQL container recreation with data preserved  
✔ PostgreSQL kept internal without publishing TCP/5432

## Multi-container CRUD application

✔ Custom Flask/Python image  
✔ Flask connected to `postgres-db` by container name  
✔ `GET /notes` verified  
✔ `POST /notes` verified  
✔ `PUT /notes/<id>` verified  
✔ `DELETE /notes/<id>` verified  
✔ HTTP 200 / 201 / 204 behavior observed  
✔ Flask application logs inspected

Final application path:

```text
KVM client / Mint
      ↓
10.10.10.254:5000
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
```

✔ API reached successfully from Alpine-Lab-01 and Alpine-Lab-02  
✔ Basic startup automation verified with `start-stack.sh`

---

# Phase 10 Experiments Completed

## Template preparation

✔ Created clean `Alpine-Template-v2`  
✔ Installed Alpine Linux without booting the installed OS before sealing  
✔ Installed `cloud-init`, `cloud-init-openrc`, `cloud-init-datasource-nocloud`, and `e2fsprogs-extra`  
✔ Enabled cloud-init OpenRC services  
✔ Restricted cloud-init to the NoCloud datasource  
✔ Cleared `/var/lib/cloud/*`  
✔ Removed SSH host keys before sealing the template

## NoCloud troubleshooting

✔ Tested external ISO9660 CIDATA seed media  
✔ Tested external VFAT CIDATA seed media  
✔ Tested seed media through different virtual-device attachment methods  
✔ Identified Alpine/OpenRC early-boot seed-mount failure as the blocking issue  
✔ Replaced the external-seed approach with direct guest-filesystem injection

Final seed location:

```text
/var/lib/cloud/seed/nocloud/
├── user-data
└── meta-data
```

## qcow2 and offline customization

✔ Created copy-on-write overlays backed by `Alpine-Template-v2.qcow2`  
✔ Exposed overlays as `/dev/nbd0` with `qemu-nbd`  
✔ Detected guest partitions with `partprobe`  
✔ Mounted the Alpine root filesystem from `/dev/nbd0p3`  
✔ Injected NoCloud configuration before first boot  
✔ Flushed writes and disconnected NBD cleanly  
✔ Preserved the template as an unchanged reusable backing image

Provisioning path:

```text
Alpine-Template-v2.qcow2
        ↓
qcow2 overlay
        ↓
qemu-nbd -> /dev/nbd0
        ↓
mount /dev/nbd0p3
        ↓
inject NoCloud seed
        ↓
unmount / disconnect
        ↓
virt-install --import
```

## cloud-init account and SSH provisioning

✔ Generated instance-specific `instance-id` and `local-hostname`  
✔ Created the `airgon` user  
✔ Installed the SSH public key automatically  
✔ Diagnosed SSH rejection caused by a locked cloud-init account  
✔ Added `lock_passwd: false` with a valid password hash  
✔ Kept SSH password authentication disabled with `ssh_pwauth: false`  
✔ Verified key-based SSH access

## Provisioning script

✔ Created executable `provision-vm.sh`  
✔ Added VM-name argument handling  
✔ Added existing-domain and existing-disk safety checks  
✔ Generated per-instance `meta-data` automatically  
✔ Automated qcow2 overlay creation  
✔ Automated NBD attachment and filesystem mounting  
✔ Automated NoCloud seed injection  
✔ Added state-aware cleanup variables  
✔ Added `cleanup()` Bash function  
✔ Added `trap cleanup EXIT` for failure-safe resource cleanup  
✔ Automated `virt-install --import`  
✔ Added DHCP polling  
✔ Matched the exact libvirt DHCP hostname with `awk`  
✔ Extracted and printed the assigned IP address  
✔ Printed the ready-to-use SSH command

Final command:

```bash
./provision-vm.sh Alpine-Auto-04
```

Final result:

```text
VM Alpine-Auto-04 created successfully.
IP address: 192.168.122.237
SSH:
  ssh airgon@192.168.122.237
```

SSH verification:

```text
Welcome to Alpine!

alpine-auto-04:~$
```

No manual guest configuration was required.

---

# Completed Milestones

## Phase 1 --- KVM Fundamentals

✔ KVM installation  
✔ virsh fundamentals  
✔ Alpine installation

## Phase 2 --- Virtual Machine Management

✔ SSH key authentication  
✔ Manual VM cloning  
✔ virt-clone  
✔ XML editing  
✔ Snapshot strategy

## Phase 3 --- Networking Foundations

✔ Custom virtual network  
✔ Linux router  
✔ Static addressing  
✔ IP forwarding  
✔ NAT  
✔ Internet access  
✔ Inter-VM routing  
✔ Documentation complete

## Phase 4 --- Network Services

✔ dnsmasq  
✔ DHCP and reservations  
✔ DNS and forwarding  
✔ Local DNS zone and search domain  
✔ Automatic hostname resolution  
✔ Modular `/etc/dnsmasq.d` configuration  
✔ Documentation complete

## Phase 5 --- Firewall & Security

✔ Stateful INPUT and FORWARD firewalls  
✔ Default-deny policies  
✔ Logging and hardening  
✔ Nmap packet-analysis experiments  
✔ DROP vs REJECT  
✔ `rp_filter` investigation  
✔ NAT/MASQUERADE and conntrack verification  
✔ Persistence and reboot recovery  
✔ Documentation complete  
✔ Git checkpoint complete  
✔ Final snapshots complete

## Phase 6 --- Linux Services & Service Exposure

✔ OpenRC service-management fundamentals  
✔ nginx installation and service management  
✔ HTTP and static web content  
✔ nginx logs  
✔ Filesystem permissions and least privilege  
✔ Specific service binding  
✔ DNS/service-name experiment  
✔ Host firewall  
✔ DROP vs REJECT vs no-listener experiment  
✔ Default-deny INPUT  
✔ nginx persistence  
✔ iptables persistence  
✔ Reboot verification  
✔ Documentation complete  
✔ Git checkpoint complete  
✔ Final snapshots complete

## Phase 7 --- DNAT / Service Publishing

✔ PREROUTING DNAT  
✔ Published `192.168.122.252:8080 -> 10.10.10.2:80`  
✔ Stateful FORWARD rule for published HTTP  
✔ Backend firewall rule for upstream source network  
✔ Source-address behavior verified  
✔ conntrack and reverse NAT inspected  
✔ Nmap service fingerprinting through DNAT  
✔ tcpdump packet-flow verification  
✔ TCP flag / sequence / acknowledgement analysis  
✔ Controlled failure-mode troubleshooting  
✔ Persistence and reboot recovery  
✔ Documentation complete

## Phase 8 --- Storage Management

✔ libvirt storage pools and volumes  
✔ qcow2 and RAW sparse-allocation behavior  
✔ `qemu-img` image-management workflow  
✔ Additional persistent qcow2 data disk  
✔ Guest block-device identification  
✔ MBR partitioning and ext4 filesystem creation  
✔ Persistent `/srv/data` mount using UUID  
✔ Reboot persistence verification  
✔ End-to-end 2 GiB → 3 GiB resize  
✔ Online ext4 expansion  
✔ Data-integrity verification  
✔ Controlled `/etc/fstab` failure and recovery  
✔ Documentation complete

## Phase 9 --- Containers & Automation

✔ Podman fundamentals and lifecycle  
✔ Rootless / daemonless concepts  
✔ Port publishing  
✔ Bind mounts and named volumes  
✔ Container logs and environment variables  
✔ Custom Containerfile images  
✔ Dedicated container networking  
✔ PostgreSQL persistent storage  
✔ Flask + PostgreSQL multi-container application  
✔ Full CRUD API  
✔ KVM-network access to published API  
✔ Basic shell startup automation  
✔ Documentation complete  
✔ Git checkpoint complete

## Phase 10 --- VM Provisioning & Automation

✔ Clean Alpine template  
✔ cloud-init / NoCloud  
✔ qcow2 copy-on-write overlays  
✔ External seed failure investigation  
✔ Direct NoCloud filesystem injection  
✔ `qemu-nbd` offline customization  
✔ Automatic hostname and metadata generation  
✔ Automatic user and SSH key provisioning  
✔ Locked-account troubleshooting  
✔ Defensive Bash cleanup with `trap`  
✔ Automated `virt-install --import`  
✔ Automatic DHCP discovery  
✔ End-to-end `Alpine-Auto-04` test  
✔ Immediate SSH key access verified  
✔ Documentation complete

**Git checkpoint pending.**

---

# Client Health

## Alpine-Lab-02

```text
IP:       10.10.10.2/24
Gateway:  10.10.10.1
DNS:      10.10.10.1
Domain:   lab.local
Role:     nginx Web Server
```

Verified:

- ✔ Router connectivity
- ✔ Internet connectivity through Alpine-Lab-01
- ✔ DNS
- ✔ SSH
- ✔ nginx TCP/80
- ✔ HTTP access
- ✔ Host firewall
- ✔ nginx persistence
- ✔ Firewall persistence
- ✔ `/srv/data` persistent mount
- ✔ 3 GiB data volume and ext4 filesystem
- ✔ Persistent data after resize

## Alpine-Lab-03

```text
IP:       10.10.10.3/24
Gateway:  10.10.10.1
DNS:      10.10.10.1
Domain:   lab.local
Role:     Internal Client
```

Verified:

- ✔ Router connectivity
- ✔ Internet connectivity
- ✔ DNS
- ✔ SSH from Alpine-Lab-01
- ✔ HTTP access to Alpine-Lab-02

---

# Automated Provisioning Health

Template:

```text
Alpine-Template-v2
Status: clean reusable template
Disk:   Alpine-Template-v2.qcow2
```

Verified automated instances:

```text
Alpine-Auto-02   cloud-init / SSH verified
Alpine-Auto-03   scripted provisioning / SSH verified
Alpine-Auto-04   final automated workflow / SSH verified
```

Final Phase 10 verification target:

```text
Alpine-Auto-04
IP:       192.168.122.237/24
Hostname: alpine-auto-04
User:     airgon
Access:   SSH public key
```

---

# Next Goal

## Phase 11 --- Final Integration / Capstone

With Phase 10 technical work and documentation complete, the next phase will focus on:

1. Integrating the infrastructure built throughout the lab
2. Final verification and recovery testing
3. Documentation cleanup
4. Final architecture diagram
5. Final README and status review
6. Final snapshots / known-good checkpoints where appropriate
7. Final Git checkpoint

---

# Latest Checkpoint

Current known-good technical state:

```text
Phase 10 --- VM Provisioning & Automation complete
```

Final automated provisioning verification:

```text
./provision-vm.sh Alpine-Auto-04
        ↓
qcow2 overlay
        ↓
NoCloud injection
        ↓
virt-install --import
        ↓
DHCP: 192.168.122.237
        ↓
ssh airgon@192.168.122.237
        ↓
SUCCESS
```

Phase 10 changed the provisioning/template environment rather than the original Alpine-Lab-01/02/03 service stack. Snapshot decisions should therefore reflect which machines actually changed rather than creating snapshots only to preserve numbering.

---

# Repository Status

Current branch:

```text
main
```

Current documentation milestone:

```text
Phase 10 --- VM Provisioning & Automation complete
```

Files being updated:

```text
docs/15-vm-provisioning-automation.md
README.md
LAB_STATUS.md
CHANGELOG.md
```

Workflow:

```text
Documentation update
        ↓
git status / git diff
        ↓
Git commit
        ↓
Push main
        ↓
Clean shutdown / checkpoint review
        ↓
Phase 11 --- Final Integration / Capstone
```

The known-good Phase 10 provisioning workflow should remain unchanged while this checkpoint is documented and committed.
