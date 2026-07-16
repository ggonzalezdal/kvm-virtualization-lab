# KVM Virtualization Lab

A hands-on learning repository for KVM, QEMU, libvirt, Alpine Linux, Linux administration, and LPIC-1 practice.

## Current Lab Architecture

```text
Windows 11 host
└── VirtualBox
    └── Linux Mint 22.2
        └── KVM / QEMU / libvirt
            └── Alpine-Lab-01
```

This is a nested virtualization lab. KVM runs inside a Linux Mint virtual machine, which itself runs inside VirtualBox.

## Current Progress

- Nested virtualization enabled
- KVM modules loaded
- `/dev/kvm` available
- libvirt installed and running
- virt-manager installed
- Alpine Linux 3.24.1 installed
- Default libvirt NAT networking working
- Alpine VM address assigned from `192.168.122.0/24`
- VirtIO disk detected as `/dev/vda`
- Baseline snapshot created:
  - `01-fresh-alpine-install`

## Roadmap

### Phase 1 — `virsh` Command-Line VM Management

Goal: manage virtual machines without relying on virt-manager.

Topics:

- libvirt architecture
- `virsh list`
- `virsh list --all`
- `virsh start`
- `virsh shutdown`
- `virsh reboot`
- `virsh destroy`
- `virsh suspend`
- `virsh resume`
- `virsh dominfo`
- `virsh domstate`
- `virsh domblklist`
- `virsh domiflist`
- `virsh console`
- `virsh dumpxml`

### Phase 2 — libvirt Networking

Goal: understand and manage virtual networks.

Topics:

- default NAT network
- DHCP and DNS
- isolated networks
- host-only networking
- bridge networking
- network XML
- `virsh net-list`
- `virsh net-info`
- `virsh net-dumpxml`

### Phase 3 — Snapshots and Cloning

Goal: create reusable VM templates and recover quickly from mistakes.

Topics:

- internal snapshots
- external snapshots
- snapshot creation
- snapshot restore
- snapshot deletion
- full clones
- linked clones
- VM backup
- VM export and import

### Phase 4 — SSH Access to Alpine

Goal: administer Alpine remotely instead of using the graphical console.

Topics:

- install OpenSSH
- enable and start SSH
- password authentication
- SSH keys
- `ssh-copy-id`
- SCP
- SFTP
- basic SSH hardening

### Phase 5 — Multiple VMs and Virtual Networking

Goal: build a small multi-machine Linux lab.

Planned machines:

```text
Linux Mint KVM host
├── Alpine-Lab-01
├── Alpine-Lab-02
├── Debian-Lab
└── Rocky-Lab
```

Exercises:

- ping between guests
- SSH between guests
- transfer files
- inspect ARP and routing tables
- test DNS and DHCP
- create isolated lab networks

### Phase 6 — LPIC-1 Administration Practice

Goal: use the virtual machines as a permanent Linux administration lab.

Topics:

- users and groups
- permissions
- ACLs
- filesystems
- mounting
- swap
- processes
- logs
- scheduled tasks
- package management
- shell scripting
- boot process
- networking
- OpenRC and systemd differences

### Phase 7 — Linux Server Services

Goal: configure practical server workloads.

Possible services:

- nginx
- Apache HTTP Server
- MariaDB
- PostgreSQL
- DNS
- NFS
- Samba
- containers

### Phase 8 — Advanced Virtualization

Topics:

- VirtIO
- qcow2 internals
- storage pools
- CPU models
- CPU pinning
- huge pages
- cloud-init
- PCI passthrough concepts
- nested virtualization
- performance tuning

## Final Project

Build and document a small virtual datacenter:

```text
Linux Mint KVM host
└── libvirt
    ├── Alpine Router
    ├── Debian Web Server
    │   └── nginx
    ├── Rocky Linux Database Server
    │   └── PostgreSQL
    └── Linux Client
```

The environment will be:

- managed using `virsh`
- accessed using SSH
- connected through custom libvirt networks
- protected with snapshots and backups
- documented with commands, diagrams, and troubleshooting notes

## Repository Structure

```text
kvm-virtualization-lab/
├── README.md
├── docs/
│   ├── 01-kvm-installation.md
│   ├── 02-alpine-installation.md
│   ├── 03-virsh-basics.md
│   ├── 04-libvirt-networking.md
│   ├── 05-snapshots-and-cloning.md
│   ├── 06-ssh-access.md
│   └── 07-multiple-vms.md
├── commands/
│   ├── virsh-cheatsheet.md
│   ├── networking-cheatsheet.md
│   └── troubleshooting.md
├── xml/
│   ├── vm-definitions/
│   └── network-definitions/
└── images/
```

## Documentation Style

Each lab document should contain:

1. Objective
2. Environment
3. Commands used
4. Explanation
5. Expected output
6. Problems encountered
7. Solution
8. Final verification
9. LPIC-1 relevance

## Important Safety Note

Do not publish:

- passwords
- private SSH keys
- public IP addresses unless necessary
- personal usernames when privacy matters
- API tokens
- cloud credentials
- files from `/etc/shadow`
- secrets embedded in XML or configuration files

Use placeholders such as:

```text
YOUR_USERNAME
VM_IP_ADDRESS
EXAMPLE_PASSWORD
```

## Next Session

Start with `virsh` command-line management.

Initial exercises:

```bash
virsh list --all
virsh domstate Alpine-Lab-01
virsh dominfo Alpine-Lab-01
virsh domblklist Alpine-Lab-01
virsh domiflist Alpine-Lab-01
virsh dumpxml Alpine-Lab-01
virsh start Alpine-Lab-01
virsh shutdown Alpine-Lab-01
```

## Learning Goal

The purpose of this repository is not only to collect commands. It is to demonstrate practical Linux virtualization skills, troubleshooting ability, structured documentation, and progress toward Linux system administration work.
