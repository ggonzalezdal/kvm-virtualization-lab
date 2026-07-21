# Manual KVM Virtual Machine Cloning

## Objective

Create **Alpine-Lab-02** as a full, independent clone of
**Alpine-Lab-01** without using `virt-clone`.

This exercise demonstrates the separation between:

-   The libvirt domain definition (XML)
-   The qcow2 virtual disk
-   Hypervisor-level machine identity
-   Guest operating-system identity

------------------------------------------------------------------------

## Final Architecture

``` text
Windows 11
└── VirtualBox
    └── Linux Mint 22.2
        └── KVM / QEMU / libvirt
            ├── Alpine-Lab-01
            └── Alpine-Lab-02
```

## Summary of the Process

1.  Export the libvirt XML definition.
2.  Copy the qcow2 disk.
3.  Correct ownership of the copied disk.
4.  Create a new XML definition.
5.  Change the VM name.
6.  Remove the UUID (libvirt generates a new one).
7.  Point the XML to the new qcow2 disk.
8.  Define the new VM.
9.  Remove the copied MAC address and let libvirt generate a new one.
10. Boot the clone.
11. Change the hostname.
12. Regenerate SSH host keys.
13. Verify SSH connectivity.
14. Create a snapshot.

------------------------------------------------------------------------

# Commands Used

## Export the VM

``` bash
virsh dumpxml Alpine-Lab-01 > Alpine-Lab-01.xml
```

## Copy the disk

``` bash
sudo cp /var/lib/libvirt/images/Alpine-Lab-01.qcow2 \
         /var/lib/libvirt/images/Alpine-Lab-02.qcow2

sudo chown libvirt-qemu:kvm \
         /var/lib/libvirt/images/Alpine-Lab-02.qcow2
```

## Create a new XML

``` bash
cp Alpine-Lab-01.xml Alpine-Lab-02.xml
```

Modify:

-   `<name>` → Alpine-Lab-02
-   Remove `<uuid>`
-   Change `<source file='...'>` to Alpine-Lab-02.qcow2

## Define the clone

``` bash
virsh define Alpine-Lab-02.xml
```

## Generate a new MAC

``` bash
virsh edit Alpine-Lab-02
```

Remove only:

``` xml
<mac address='52:54:00:88:90:ca'/>
```

Libvirt automatically generated:

``` text
52:54:00:00:27:56
```

## Boot the clone

``` bash
virsh start Alpine-Lab-02
virsh console Alpine-Lab-02
```

## Change the hostname

``` bash
vi /etc/hostname
hostname alpine-lab-02
```

## Regenerate SSH host keys

``` bash
rm /etc/ssh/ssh_host_*
ssh-keygen -A
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

Old fingerprint:

``` text
SHA256:RgJxw2RXzvc2VMLjZrrxyfGCsH8ZQvBVtXsQIhCHV88
```

New fingerprint:

``` text
SHA256:7eEIAPkTwqnEbUvjD866i2nbdFfHMzNLRyQUS0EngWo
```

## Verify SSH policy

``` bash
sudo sshd -T | grep -E 'permitrootlogin|passwordauthentication|pubkeyauthentication'
```

Expected:

``` text
permitrootlogin no
passwordauthentication yes
pubkeyauthentication yes
```

## Create the snapshot

``` bash
virsh snapshot-create-as Alpine-Lab-02 \
  04-manual-clone-complete \
  --description "Manual full clone completed."
```

------------------------------------------------------------------------

# Final State

  Item              Alpine-Lab-02
  ----------------- -------------------
  Domain            Alpine-Lab-02
  UUID              Auto-generated
  Disk              Independent qcow2
  MAC               Unique
  Hostname          alpine-lab-02
  SSH Host Keys     Regenerated
  SSH Hardening     Preserved
  User SSH Access   Working

------------------------------------------------------------------------

# Lessons Learned

A VM has two identities.

## Hypervisor identity

-   Name
-   UUID
-   MAC address
-   Disk

## Guest identity

-   Hostname
-   SSH host keys
-   Configuration

When cloning a VM, both identities must be made unique.

The existing OpenSSH hardening and the user's authorized SSH key were
intentionally preserved because they are part of the operating-system
configuration stored inside the cloned qcow2 image.
