# Snapshot Management with `virsh`

> Phase 1.2 of the KVM Virtualization Lab

## Objective

Learn how to create, inspect, revert, and manage KVM snapshots using
`virsh`.

------------------------------------------------------------------------

# What is a Snapshot?

A snapshot is a restore point for a virtual machine.

It can preserve:

-   Disk state
-   Memory state
-   VM metadata
-   Device configuration

Snapshots let you return the VM to an earlier point in time after
experiments, upgrades, or configuration changes.

------------------------------------------------------------------------

# Snapshot Types

## Internal Snapshots

Stored inside the `.qcow2` disk image.

Example from this lab:

``` xml
<disk name='vda' snapshot='internal'/>
```

**Pros**

-   Simple
-   One disk file
-   Excellent for labs

**Cons**

-   Disk grows over time
-   Less flexible for production

## External Snapshots

Changes are stored in overlay qcow2 files.

``` text
base.qcow2
└── snap1.qcow2
    └── snap2.qcow2
```

Better suited to production workflows.

------------------------------------------------------------------------

# Disk vs Memory Snapshots

Our snapshots contain only the disk:

``` xml
<memory snapshot='no'/>
```

The VM boots normally after a revert.

------------------------------------------------------------------------

# Commands

## List snapshots

``` bash
virsh snapshot-list Alpine-Lab-01
```

Tree view:

``` bash
virsh snapshot-list Alpine-Lab-01 --tree
```

## Current snapshot

``` bash
virsh snapshot-current Alpine-Lab-01 --name
```

## Snapshot information

``` bash
virsh snapshot-info Alpine-Lab-01 SNAPSHOT_NAME
```

## Snapshot XML

``` bash
virsh snapshot-dumpxml Alpine-Lab-01 SNAPSHOT_NAME
```

## Create

``` bash
virsh snapshot-create-as \
Alpine-Lab-01 \
02-virsh-fundamentals \
"Completed Phase 1: virsh fundamentals and snapshot management."
```

## Revert

``` bash
virsh snapshot-revert Alpine-Lab-01 01-fresh-alpine-install
```

## Delete

``` bash
virsh snapshot-delete Alpine-Lab-01 SNAPSHOT_NAME
```

------------------------------------------------------------------------

# Lab Exercise

Performed successfully:

1.  Created a file inside `/root`.
2.  Reverted to `01-fresh-alpine-install`.
3.  Verified the file disappeared.

This proved that reverting restored the previous disk state.

------------------------------------------------------------------------

# Best Practices

-   Snapshot before risky changes.
-   Keep milestone snapshots.
-   Delete temporary snapshots.
-   Use consistent names.

Suggested milestones:

``` text
01-fresh-alpine-install
02-virsh-fundamentals
03-networking
04-ssh
05-storage
06-package-management
```

------------------------------------------------------------------------

# Common Mistake

Missing the space before the description:

Incorrect:

``` bash
virsh snapshot-create-as Alpine-Lab-01 02-test"Description"
```

Correct:

``` bash
virsh snapshot-create-as Alpine-Lab-01 02-test "Description"
```

------------------------------------------------------------------------

# Phase 1.2 Complete

-   Listed snapshots
-   Inspected snapshot metadata
-   Created snapshots
-   Reverted snapshots
-   Deleted snapshots
-   Verified snapshot restoration

------------------------------------------------------------------------

# Next

**SSH Access to the Alpine VM**
