# Virsh Fundamentals

> Phase 1 of the KVM Virtualization Lab

## Objective

Learn the basic `virsh` commands required to manage KVM virtual machines
from the command line.

------------------------------------------------------------------------

# Lab Environment

``` text
Windows 11
└── VirtualBox
    └── Linux Mint 22.2
        └── KVM / QEMU / libvirt
            └── Alpine-Lab-01
```

VM used throughout this phase:

-   **Name:** Alpine-Lab-01
-   **Guest OS:** Alpine Linux 3.24
-   **Hypervisor:** KVM/QEMU
-   **Management tool:** libvirt (`virsh`)

------------------------------------------------------------------------

# Basic VM Management

## List running virtual machines

``` bash
virsh list
```

## List all virtual machines

``` bash
virsh list --all
```

## Show VM information

``` bash
virsh dominfo Alpine-Lab-01
```

Useful information:

-   UUID
-   State
-   vCPUs
-   Memory
-   Persistent
-   Autostart

## Show VM state

``` bash
virsh domstate Alpine-Lab-01
```

Typical states:

-   running
-   shut off
-   paused

------------------------------------------------------------------------

# CPU & Memory

## Show virtual CPU information

``` bash
virsh vcpuinfo Alpine-Lab-01
```

## Show memory statistics

``` bash
virsh dommemstat Alpine-Lab-01
```

> **Note:** The VM should be running to obtain runtime memory
> statistics.

  Field       Description
  ----------- -----------------------------------
  actual      Memory assigned to the VM
  rss         Host RAM currently used
  available   Memory available inside the guest
  unused      Unused guest memory

------------------------------------------------------------------------

# Storage

## List attached disks

``` bash
virsh domblklist Alpine-Lab-01
```

Example:

``` text
Target  Source
----------------------------------------
vda     alpine.qcow2
sda     alpine.iso
```

The Alpine installation disk appears inside the guest as:

``` text
/dev/vda
```

------------------------------------------------------------------------

# Networking

## List network interfaces

``` bash
virsh domiflist Alpine-Lab-01
```

Displays:

-   Interface type
-   Source network
-   Model
-   MAC address

Current lab network:

``` text
Default libvirt NAT
```

------------------------------------------------------------------------

# XML Configuration

Display the VM definition:

``` bash
virsh dumpxml Alpine-Lab-01
```

The XML contains:

-   Memory
-   CPUs
-   Storage
-   Network
-   Boot devices
-   Graphics
-   Console
-   UUID

Safely edit:

``` bash
virsh edit Alpine-Lab-01
```

------------------------------------------------------------------------

# Power Management

## Start

``` bash
virsh start Alpine-Lab-01
```

## Graceful shutdown

``` bash
virsh shutdown Alpine-Lab-01
```

## Force power off

``` bash
virsh destroy Alpine-Lab-01
```

`destroy` **does not delete the VM**. It is equivalent to pulling the
power cable on a physical computer.

------------------------------------------------------------------------

# Serial Console

Connect:

``` bash
virsh console Alpine-Lab-01
```

Documented escape sequence:

``` text
Ctrl + ]
```

### Lab Note

Current lab:

-   Linux Mint 22.2
-   libvirt 10.0.0
-   Spanish keyboard layout

The documented escape sequence could not be reproduced reliably.

For day-to-day administration, SSH will be preferred after it is
configured.

------------------------------------------------------------------------

# Runtime vs Configuration Commands

## Work when the VM is stopped

``` bash
virsh dominfo
virsh domstate
virsh domblklist
virsh domiflist
virsh dumpxml
```

## Require the VM to be running

``` bash
virsh dommemstat
virsh vcpuinfo
virsh cpu-stats
```

------------------------------------------------------------------------

# Commands Learned

  Command                 Purpose
  ----------------------- ---------------------------
  `virsh list`            Running VMs
  `virsh list --all`      All VMs
  `virsh dominfo VM`      VM information
  `virsh domstate VM`     VM state
  `virsh vcpuinfo VM`     CPU information
  `virsh dommemstat VM`   Memory statistics
  `virsh domblklist VM`   Virtual disks
  `virsh domiflist VM`    Network interfaces
  `virsh dumpxml VM`      VM configuration
  `virsh edit VM`         Edit VM configuration
  `virsh start VM`        Start VM
  `virsh shutdown VM`     Gracefully stop VM
  `virsh destroy VM`      Force stop VM
  `virsh console VM`      Connect to serial console

------------------------------------------------------------------------

# Progress

-   ✅ Learned the basic `virsh` command set
-   ✅ Inspected VM configuration
-   ✅ Managed VM power state
-   ✅ Explored CPU, memory, disks and networking
-   ✅ Examined the VM XML definition
-   ✅ Connected to the serial console
-   ✅ Identified the console escape-sequence issue

------------------------------------------------------------------------

# Next Phase

**Phase 1.2 --- Snapshot Management with `virsh`**

Topics:

-   List snapshots
-   Create snapshots
-   Inspect snapshots
-   Revert snapshots
-   Delete snapshots
-   Internal vs external snapshots
-   Live vs offline snapshots
