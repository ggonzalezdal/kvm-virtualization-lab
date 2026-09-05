# Phase 8 — Storage Management

## Objective

Learn how KVM/libvirt storage maps from host disk images to guest block devices, create and attach additional storage, configure persistent filesystems, resize storage safely, and troubleshoot common storage-layer problems.

---

## 1. Inspecting Existing KVM Storage

### VM block devices

```bash
virsh domblklist Alpine-Lab-01
virsh domblkinfo Alpine-Lab-01 vda
```

The main system disk is stored on the Linux Mint host as a qcow2 image under:

```text
/var/lib/libvirt/images/
```

Example:

```text
/var/lib/libvirt/images/Alpine-Lab-01.qcow2
```

Inside the guest, QEMU/libvirt exposes that image as a virtual block device such as `/dev/vda`.

### Inspecting qcow2 images

```bash
sudo ls -lh /var/lib/libvirt/images/Alpine-Lab-01.qcow2
sudo du -h /var/lib/libvirt/images/Alpine-Lab-01.qcow2
sudo qemu-img info /var/lib/libvirt/images/Alpine-Lab-01.qcow2
```

Important distinction:

- **Virtual size** — capacity presented to the VM.
- **Disk size / allocation** — actual host storage currently consumed.
- qcow2 images can therefore provide a large virtual disk while consuming much less physical storage.

### Guest view

```bash
lsblk
```

Storage stack:

```text
qcow2 image
    ↓
QEMU / libvirt
    ↓
virtual block device (/dev/vdX)
    ↓
partition
    ↓
filesystem
    ↓
mount point
```

---

## 2. libvirt Storage Pools and Volumes

Inspect pools:

```bash
virsh pool-list --all
virsh pool-info default
virsh pool-dumpxml default
```

The `default` pool is directory-backed:

```text
/var/lib/libvirt/images
```

Inspect volumes:

```bash
virsh vol-list default
virsh vol-info --pool default Alpine-Lab-01.qcow2
```

A libvirt **storage pool** manages storage resources. A **volume** is an individual managed storage object, such as a qcow2 disk image or ISO.

---

## 3. qcow2 and RAW Experiments

Temporary test directory:

```text
/tmp/kvm-storage-lab
```

### RAW

```bash
qemu-img create -f raw test-raw.img 1G
```

A raw image can also be sparse when supported by the host filesystem.

Writing 100 MiB:

```bash
dd if=/dev/zero of=test-raw.img bs=1M count=100 conv=notrunc
```

### qcow2

```bash
qemu-img create -f qcow2 test-qcow2.qcow2 1G
qemu-io -f qcow2 -c "write 0 100M" test-qcow2.qcow2
qemu-img check test-qcow2.qcow2
```

`qemu-io` writes to the virtual contents of the qcow2 image without overwriting its container metadata.

### Resize and conversion

```bash
qemu-img resize test-qcow2.qcow2 2G
qemu-img convert -f raw -O qcow2 test-raw.img converted.qcow2
```

Core commands learned:

```bash
qemu-img info IMAGE
qemu-img create -f FORMAT IMAGE SIZE
qemu-img check IMAGE
qemu-img resize IMAGE SIZE
qemu-img convert -f INPUT -O OUTPUT SOURCE DESTINATION
```

---

## 4. Creating a Second Disk for Alpine-Lab-02

Created a 2 GiB qcow2 volume:

```bash
virsh vol-create-as default Alpine-Lab-02-data.qcow2 2G --format qcow2
```

Verified:

```bash
virsh vol-info --pool default Alpine-Lab-02-data.qcow2
```

Attached persistently:

```bash
virsh attach-disk   Alpine-Lab-02   /var/lib/libvirt/images/Alpine-Lab-02-data.qcow2   vdb   --persistent   --subdriver qcow2
```

Host mapping:

```bash
virsh domblklist Alpine-Lab-02
```

### Important device-enumeration lesson

libvirt configured:

```text
vda → Alpine-Lab-02.qcow2
vdb → Alpine-Lab-02-data.qcow2
```

However, inside Alpine the kernel enumerated them as:

```text
/dev/vda → 2 GiB data disk
/dev/vdb → 10 GiB operating-system disk
```

Therefore `/dev/vda`, `/dev/vdb`, etc. should not be treated as permanent identities.

Prefer UUIDs, labels, and `/dev/disk/by-*` identifiers where persistence matters.

---

## 5. Partitioning and Formatting the Data Disk

The new 2 GiB disk appeared inside Alpine-Lab-02 as `/dev/vda`.

Partitioned it:

```bash
sudo fdisk /dev/vda
```

Created one primary Linux partition occupying the entire disk:

```text
/dev/vda1
```

Created an ext4 filesystem:

```bash
sudo mkfs.ext4 /dev/vda1
```

Filesystem UUID:

```text
7e131cc4-bf19-4274-b69a-43311d204295
```

Verified with:

```bash
sudo blkid /dev/vda1
```

---

## 6. Mounting the Data Disk Persistently

Created the mount point:

```bash
sudo mkdir -p /srv/data
```

Mounted manually:

```bash
sudo mount /dev/vda1 /srv/data
```

Verified:

```bash
lsblk
df -h /srv/data
```

Added the following entry to `/etc/fstab`:

```text
UUID=7e131cc4-bf19-4274-b69a-43311d204295  /srv/data  ext4  defaults  0 2
```

Tested without rebooting:

```bash
sudo umount /srv/data
sudo mount -a
```

Then rebooted Alpine-Lab-02 and confirmed `/srv/data` mounted automatically.

Created persistent test data:

```bash
echo "Persistent data from Alpine-Lab-02" | sudo tee /srv/data/test.txt
cat /srv/data/test.txt
```

---

## 7. Resizing the Data Disk

Goal:

```text
2 GiB → 3 GiB
```

### Layer 1 — qcow2 virtual disk

With Alpine-Lab-02 shut down, on Linux Mint:

```bash
sudo qemu-img resize   /var/lib/libvirt/images/Alpine-Lab-02-data.qcow2   3G
```

Verified:

```bash
sudo qemu-img info   /var/lib/libvirt/images/Alpine-Lab-02-data.qcow2
```

After booting the guest:

```text
/dev/vda  = 3G
/dev/vda1 = 2G
ext4      ≈ 1.9G
```

This demonstrated that enlarging the virtual disk does **not** automatically enlarge its partition or filesystem.

### Layer 2 — partition

Inspected:

```bash
sudo fdisk -l /dev/vda
```

The partition still ended at the old 2 GiB boundary.

After unmounting `/srv/data`, `fdisk` was used to delete and recreate partition 1 with:

- the **same starting sector: 2048**
- a new ending sector at the end of the 3 GiB disk
- the existing ext4 signature preserved

Result:

```text
/dev/vda1 = 3G
```

The unchanged starting sector was critical because the existing filesystem data begins there.

### Layer 3 — ext4 filesystem

Installed the required Alpine utility package:

```bash
sudo apk add e2fsprogs-extra
```

Expanded ext4 online:

```bash
sudo resize2fs /dev/vda1
```

Final result:

```text
/dev/vda1 filesystem ≈ 2.9G
```

Verified:

```bash
df -h /srv/data
cat /srv/data/test.txt
```

The original data remained intact.

### Resize model

```text
qcow2
  ↓ qemu-img resize
virtual disk
  ↓ fdisk
partition
  ↓ resize2fs
ext4 filesystem
  ↓
/srv/data
```

Each layer must be considered independently.

---

## 8. Storage Troubleshooting Exercise

A controlled `/etc/fstab` failure was introduced by changing one character in the `/srv/data` UUID.

After:

```bash
sudo umount /srv/data
sudo mount -a
```

the system reported:

```text
mount: /srv/data: can't find UUID=...
```

Diagnosis:

```text
virtual disk     OK
partition        OK
filesystem       OK
mount point      OK
/etc/fstab UUID  WRONG
```

The correct UUID was identified with:

```bash
sudo blkid /dev/vda1
```

The known-good `/etc/fstab` was restored and verified:

```bash
sudo mount -a
df -h /srv/data
cat /srv/data/test.txt
```

Final state:

```text
/dev/vda1 → ext4 → /srv/data
Size: approximately 2.9 GiB
Persistent data intact
```

---

## Key Lessons

1. A qcow2 file, virtual disk, partition, filesystem, and mount point are separate storage layers.
2. qcow2 virtual capacity and actual host allocation are different values.
3. RAW images can also be sparse.
4. libvirt storage pools contain managed volumes.
5. Guest `/dev/vdX` names are not reliable persistent identities.
6. UUIDs are preferable for persistent `/etc/fstab` configuration.
7. Enlarging a qcow2 image does not automatically enlarge its partition or filesystem.
8. Storage troubleshooting should identify the exact failing layer rather than treating “the disk” as one object.

## Phase 8 Final Storage Stack

```text
Linux Mint host
    ↓
libvirt default storage pool
    ↓
Alpine-Lab-02-data.qcow2 (3 GiB)
    ↓
QEMU / virtio
    ↓
guest data disk
    ↓
/dev/vda1 (3 GiB)
    ↓
ext4 (~2.9 GiB)
    ↓
/srv/data
    ↓
persistent via UUID in /etc/fstab
```

**Phase 8 — Storage Management: COMPLETE**
