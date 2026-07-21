# virt-clone Automation

## Objective

Learn how to clone a KVM virtual machine using **virt-clone** and
compare it with the manual cloning process.

## Lab Topology

``` text
Linux Mint
└── KVM / libvirt
    ├── Alpine-Lab-01 (Golden Image)
    ├── Alpine-Lab-02 (Manual Clone)
    └── Alpine-Lab-03 (virt-clone)
```

## Why use virt-clone?

`virt-clone` automates the hypervisor-side cloning tasks:

-   Copy the virtual disk
-   Create a new libvirt domain
-   Generate a new UUID
-   Generate a new MAC address
-   Define the new virtual machine

It does **not** modify the operating system inside the guest.

## Verify the tool

``` bash
which virt-clone
virt-clone --version
```

## Clone the VM

Ensure the source VM is powered off:

``` bash
virsh domstate Alpine-Lab-01
```

Clone it:

``` bash
sudo virt-clone \
  --original Alpine-Lab-01 \
  --name Alpine-Lab-03 \
  --file /var/lib/libvirt/images/Alpine-Lab-03.qcow2
```

## Verification

``` bash
virsh list --all
virsh dominfo Alpine-Lab-03
virsh domiflist Alpine-Lab-03
virsh domblklist Alpine-Lab-03
```

Expected:

-   New domain
-   New UUID
-   New MAC address
-   Independent qcow2 disk

## Disk behaviour

Inspect the image:

``` bash
qemu-img info /var/lib/libvirt/images/Alpine-Lab-03.qcow2
du -h /var/lib/libvirt/images/Alpine-Lab-03.qcow2
```

Observed in this lab:

``` text
Virtual size: 10 GiB
Disk size: ~251 MiB
```

Unlike the manual clone, `virt-clone` created a clean qcow2 image
without copying the internal snapshot history.

Verification:

``` bash
virsh snapshot-list Alpine-Lab-03
```

Result:

``` text
(no snapshots)
```

## Dynamic ownership

Observed during the lab:

``` text
Before start : root:root
Running      : libvirt-qemu:kvm
After stop   : root:root
```

Libvirt temporarily changes ownership while the VM is running.

## Guest customization

After the first boot the guest still had:

-   Hostname: alpine-lab-01
-   Original SSH host keys
-   Existing SSH hardening
-   Existing user accounts

The following customization was performed:

``` bash
vi /etc/hostname
hostname alpine-lab-03

rm /etc/ssh/ssh_host_*
ssh-keygen -A
```

SSH connectivity was then verified from Linux Mint.

## Manual clone vs virt-clone

  Feature                    Manual           virt-clone
  -------------------------- ---------------- ------------
  Disk copy                  `cp`             Automatic
  XML editing                Manual           Automatic
  UUID                       Manual/libvirt   Automatic
  MAC                        Manual/libvirt   Automatic
  Internal qcow2 snapshots   Preserved        Not copied
  Guest hostname             Manual           Manual
  SSH host keys              Manual           Manual

## Key takeaway

A clone has two identities.

### Hypervisor identity

-   Domain name
-   UUID
-   MAC address
-   Virtual disk

Handled by `virt-clone`.

### Guest identity

-   Hostname
-   SSH host keys
-   Guest configuration

Must still be customized after cloning.
