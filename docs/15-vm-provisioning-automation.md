# Phase 10 — VM Provisioning & Automation
## Objective
Build a repeatable VM provisioning workflow using:

- a clean Alpine Linux template

- qcow2 copy-on-write overlays

- cloud-init

- the NoCloud datasource

- offline seed injection with `qemu-nbd`

- automated VM creation with `virt-install`

- automatic DHCP discovery

- SSH key authentication

The final result is a reusable provisioning script that creates a new Alpine VM from a single command:

```bash

./provision-vm.sh Alpine-Auto-04

```

The script creates the disk overlay, injects cloud-init configuration, starts the VM, waits for DHCP, discovers the assigned IP address, and prints the SSH command.

---
## 1. Architecture
The final provisioning flow is:

```text

Alpine-Template-v2

        |

        | qcow2 backing image

        v

Alpine-Auto-XX.qcow2

        |

        | qemu-nbd

        v

/dev/nbd0

        |

        | mount root partition

        v

/var/lib/cloud/seed/nocloud/

        |

        | user-data + meta-data

        v

virt-install --import

        |

        v

first boot

        |

        v

cloud-init / NoCloud

        |

        +--> hostname

        +--> user creation

        +--> SSH public key

        +--> account configuration

        |

        v

libvirt DHCP

        |

        v

SSH-ready VM

```

---
## 2. Clean Alpine Template
A new clean template was created instead of continuing to modify the earlier experimental template.

```bash

sudo virt-install \

  --name Alpine-Template-v2 \

  --memory 1024 \

  --vcpus 2 \

  --disk pool=default,size=10,format=qcow2,bus=virtio \

  --cdrom /var/lib/libvirt/images/alpine-standard-3.24.1-x86_64.iso \

  --network network=default,model=virtio \

  --os-variant alpinelinux3.19 \

  --graphics spice \

  --noautoconsole

```

The Alpine installation was performed normally with `setup-alpine`.

Important template characteristics:

```text

Hostname: alpine-template-v2

Network: DHCP

SSH server: OpenSSH

Root SSH: prohibit-password

Disk: /dev/vda

Installation type: sys

```

After installation completed, the installed system was ****not booted****.

This allowed cloud-init to be installed and configured before the template's first real boot.

---
## 3. Installing cloud-init Inside the Template
The root filesystem was mounted from the installer environment and entered with `chroot`.

Packages installed:

```sh

apk add cloud-init cloud-init-openrc cloud-init-datasource-nocloud e2fsprogs-extra

```

The cloud-init OpenRC services were enabled:

```sh

rc-update add cloud-init-local boot

rc-update add cloud-init default

rc-update add cloud-config default

rc-update add cloud-final default

```

The datasource was restricted to NoCloud:

```sh

mkdir -p /etc/cloud/cloud.cfg.d

cat > /etc/cloud/cloud.cfg.d/90_dpkg.cfg <<'EOF'

datasource_list: [ NoCloud ]

EOF

```

Before sealing the template:

```sh

rm -rf /var/lib/cloud/*

rm -f /etc/ssh/ssh_host_*

```

Then the template was powered off.

This leaves the image clean so every clone can:

- generate its own SSH host keys

- receive a new cloud-init instance ID

- receive its own hostname

- receive its own user configuration

---
## 4. The External NoCloud Seed Problem
The first approach used an external NoCloud seed image created with `cloud-localds`.

Attempts included:

```text

ISO9660 CIDATA attached as CD-ROM

ISO9660 CIDATA attached as virtio disk

VFAT CIDATA attached as virtio disk

```

In all cases, Alpine/OpenRC cloud-init failed during early boot while attempting to mount the external seed device.

The VM therefore fell back without applying the intended configuration.

Symptoms included:

```text

hostname not changed

user not created correctly

SSH configuration not applied

NoCloud datasource not detected as intended

```

---
## 5. Final Solution — Inject NoCloud Seed Directly Into the qcow2 Overlay
Instead of presenting cloud-init with a separate seed device, the NoCloud files are written directly into the guest filesystem before first boot.

The target directory inside the Alpine guest is:

```text

/var/lib/cloud/seed/nocloud/

```

The required files are:

```text

user-data

meta-data

```

This completely avoids the problematic external seed mount.

---
## 6. qcow2 Overlay
Each automated VM uses a copy-on-write overlay backed by the clean template.

Example:

```bash

sudo qemu-img create \

  -f qcow2 \

  -F qcow2 \

  -b /var/lib/libvirt/images/Alpine-Template-v2.qcow2 \

  /var/lib/libvirt/images/Alpine-Auto-02.qcow2

```

Conceptually:

```text

Alpine-Template-v2.qcow2

        |

        | backing file

        v

Alpine-Auto-02.qcow2

```

The template remains unchanged.

Only changes made by the new VM are written into its overlay.

Benefits:

- fast VM creation

- low initial disk usage

- clean reusable base image

- independent guests

- easy disposable test systems

---
## 7. Exposing the qcow2 Disk With NBD
`qemu-nbd` allows a qcow2 image to appear temporarily as a Linux block device on the Mint host.

Load NBD support:

```bash

sudo modprobe nbd max_part=8

```

Connect the overlay:

```bash

sudo qemu-nbd \

  --connect=/dev/nbd0 \

  /var/lib/libvirt/images/Alpine-Auto-02.qcow2

```

Refresh partition information:

```bash

sudo partprobe /dev/nbd0

```

The mapping is:

```text

Inside VM             Mint host through NBD

/dev/vda        -->   /dev/nbd0

/dev/vda1       -->   /dev/nbd0p1

/dev/vda2       -->   /dev/nbd0p2

/dev/vda3       -->   /dev/nbd0p3

```

For this Alpine template:

```text

/dev/nbd0p3 = root filesystem

```

Mount it:

```bash

sudo mkdir -p /mnt/alpine-auto-02

sudo mount /dev/nbd0p3 /mnt/alpine-auto-02

```

---
## 8. Injecting NoCloud Data
Create the NoCloud seed directory inside the guest:

```bash

sudo mkdir -p \

  /mnt/alpine-auto-02/var/lib/cloud/seed/nocloud

```

Copy the cloud-init files:

```bash

sudo cp user-data \

  /mnt/alpine-auto-02/var/lib/cloud/seed/nocloud/user-data

sudo cp meta-data \

  /mnt/alpine-auto-02/var/lib/cloud/seed/nocloud/meta-data

```

Flush writes:

```bash

sudo sync

```

Unmount:

```bash

sudo umount /mnt/alpine-auto-02

```

Disconnect NBD:

```bash

sudo qemu-nbd --disconnect /dev/nbd0

```

The qcow2 overlay now contains its cloud-init configuration before the VM has ever booted.

---
## 9. cloud-init `meta-data`
Example:

```yaml

instance-id: alpine-auto-04

local-hostname: alpine-auto-04

```

Each VM receives a unique instance ID and hostname.

The provisioning script generates this dynamically from the VM name.

For example:

```bash

./provision-vm.sh Alpine-Auto-04

```

becomes:

```yaml

instance-id: alpine-auto-04

local-hostname: alpine-auto-04

```

The Bash expansion:

```bash

${VM_NAME,,}

```

converts the VM name to lowercase.

---
## 10. cloud-init `user-data`
Working configuration:

```yaml

#cloud-config

users:

  - name: airgon

    groups: [adm, wheel]

    shell: /bin/ash

    lock_passwd: false

    passwd: '<sha-512-crypt-hash>'

    ssh_authorized_keys:

      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINszckLSAJBL2aZcsaTfkWZ7Z2KMsMQtSUu25DVsr/Ta lab-admin

ssh_pwauth: false

disable_root: true

growpart:

  mode: 'off'

resize_rootfs: false

```

Important points:

### `lock_passwd: false`
Without this, the `airgon` account was created but remained locked.

The SSH daemon then rejected the user even though the public key had been installed.

Observed error:

```text

User airgon not allowed because account is locked

```

A valid password hash keeps the account unlocked.

SSH password authentication is still disabled globally by:

```yaml

ssh_pwauth: false

```

Therefore SSH access still uses the public key.

### Password hashes
A SHA-512 crypt hash can be generated with:

```bash

openssl passwd -6

```

The hash should be placed in:

```yaml

passwd: '<hash>'

```

The repository documentation uses a placeholder instead of publishing a reusable password hash. Generate a unique value for your own environment.

### `growpart`
This is intentionally disabled:

```yaml

growpart:

  mode: 'off'

```

The value is quoted because unquoted YAML `off` can be interpreted as a boolean.

---
## 11. Manual VM Import
After cloud-init seed injection, the VM can be created directly from the qcow2 overlay:

```bash

sudo virt-install \

  --name Alpine-Auto-02 \

  --memory 1024 \

  --vcpus 2 \

  --disk path=/var/lib/libvirt/images/Alpine-Auto-02.qcow2,format=qcow2,bus=virtio \

  --network network=default,model=virtio \

  --os-variant alpinelinux3.19 \

  --import \

  --graphics none \

  --noautoconsole

```

No installation ISO is needed.

The VM is already installed because its disk is based on the template.

---
## 12. Successful cloud-init Verification
Inside an automatically provisioned VM:

```sh

hostname

```

Example:

```text

alpine-auto-02

```

Cloud-init status:

```sh

cloud-init status --long

```

Successful result:

```text

status: done

extended_status: degraded done

boot_status_code: enabled-by-sysvinit

detail: DataSourceNoCloud [seed=/var/lib/cloud/seed/nocloud]

errors: []

```

A remaining Alpine-specific warning was observed:

```text

Unable to activate module keys_to_console,

helper tool not found at

/usr/lib/cloud-init/write-ssh-key-fingerprints

```

This warning did not affect:

- hostname configuration

- user creation

- SSH key installation

- SSH access

- NoCloud provisioning

---
## 13. Automated Provisioning Script
Final script:

```bash

#!/bin/bash

set -e

VM_NAME="$1"

if [ -z "$VM_NAME" ]; then

    echo "Usage: $0 <vm-name>"

    exit 1

fi

TEMPLATE="/var/lib/libvirt/images/Alpine-Template-v2.qcow2"

VM_DISK="/var/lib/libvirt/images/${VM_NAME}.qcow2"

CLOUD_DIR="$HOME/kvm-cloud-init/${VM_NAME}"

MOUNT_DIR="/mnt/${VM_NAME}"

NBD_DEVICE="/dev/nbd0"

NBD_CONNECTED=false

MOUNTED=false

# Safety checks

if virsh dominfo "$VM_NAME" &>/dev/null; then

    echo "Error: VM '$VM_NAME' already exists."

    exit 1

fi

if [ -e "$VM_DISK" ]; then

    echo "Error: disk '$VM_DISK' already exists."

    exit 1

fi

# Create cloud-init directory

mkdir -p "$CLOUD_DIR"

# Reuse our working user-data

cp "$HOME/kvm-cloud-init/Alpine-Auto-01/user-data" \

    "$CLOUD_DIR/user-data"

# Generate instance-specific metadata

cat > "$CLOUD_DIR/meta-data" <<EOF

instance-id: ${VM_NAME,,}

local-hostname: ${VM_NAME,,}

EOF

# Create qcow2 overlay from template

sudo qemu-img create \

    -f qcow2 \

    -F qcow2 \

    -b "$TEMPLATE" \

    "$VM_DISK"

cleanup() {

    if [ "$MOUNTED" = true ]; then

        sudo umount "$MOUNT_DIR" 2>/dev/null || true

    fi

    if [ "$NBD_CONNECTED" = true ]; then

        sudo qemu-nbd --disconnect "$NBD_DEVICE" 2>/dev/null || true

    fi

}

trap cleanup EXIT

# Load NBD support

sudo modprobe nbd max_part=8

# Attach qcow2 overlay as /dev/nbd0

sudo qemu-nbd \

  --connect="$NBD_DEVICE" \

  "$VM_DISK"

NBD_CONNECTED=true

# Detect partitions

sudo partprobe "$NBD_DEVICE"

# Mount Alpine root filesystem

sudo mkdir -p "$MOUNT_DIR"

sudo mount "${NBD_DEVICE}p3" "$MOUNT_DIR"

MOUNTED=true

# Create NoCloud seed directory inside guest

sudo mkdir -p "$MOUNT_DIR/var/lib/cloud/seed/nocloud"

# Inject cloud-init files

sudo cp "$CLOUD_DIR/user-data" \

  "$MOUNT_DIR/var/lib/cloud/seed/nocloud/user-data"

sudo cp "$CLOUD_DIR/meta-data" \

  "$MOUNT_DIR/var/lib/cloud/seed/nocloud/meta-data"

# Flush writes and disconnect cleanly

sudo sync

sudo umount "$MOUNT_DIR"

MOUNTED=false

sudo qemu-nbd --disconnect "$NBD_DEVICE"

NBD_CONNECTED=false

# Create and start VM

sudo virt-install \

  --name "$VM_NAME" \

  --memory 1024 \

  --vcpus 2 \

  --disk path="$VM_DISK",format=qcow2,bus=virtio \

  --network network=default,model=virtio \

  --os-variant alpinelinux3.19 \

  --import \

  --graphics none \

  --noautoconsole

# Wait for DHCP lease

echo

echo "Waiting for DHCP lease..."

VM_IP=""

for i in {1..30}; do

    VM_IP=$(virsh net-dhcp-leases default \

      | awk -v host="${VM_NAME,,}" '$6 == host {split($5,a,"/"); print a[1]}')

    if [ -n "$VM_IP" ]; then

        break

    fi

    sleep 2

done

if [ -n "$VM_IP" ]; then

    echo

    echo "VM $VM_NAME created successfully."

    echo "IP address: $VM_IP"

    echo "SSH:"

    echo "  ssh airgon@$VM_IP"

else

    echo

    echo "VM $VM_NAME created, but no DHCP lease was found."

    echo "Check with:"

    echo "  virsh net-dhcp-leases default"

fi

```

Make executable:

```bash

chmod +x provision-vm.sh

```

---
## 14. Safety Logic
The script uses:

```bash

set -e

```

This causes the script to stop if a command fails.

It also checks that the VM does not already exist:

```bash

if virsh dominfo "$VM_NAME" &>/dev/null; then

    echo "Error: VM '$VM_NAME' already exists."

    exit 1

fi

```

And verifies that the destination disk does not already exist:

```bash

if [ -e "$VM_DISK" ]; then

    echo "Error: disk '$VM_DISK' already exists."

    exit 1

fi

```

This helps prevent accidental overwrites.

---
## 15. `cleanup()` and `trap`
NBD and filesystem mounts are temporary host resources.

If the script failed after attaching `/dev/nbd0`, the device could otherwise remain connected.

The cleanup function is:

```bash

cleanup() {

    if [ "$MOUNTED" = true ]; then

        sudo umount "$MOUNT_DIR" 2>/dev/null || true

    fi

    if [ "$NBD_CONNECTED" = true ]; then

        sudo qemu-nbd --disconnect "$NBD_DEVICE" 2>/dev/null || true

    fi

}

```

It is registered with:

```bash

trap cleanup EXIT

```

Meaning:

```text

normal exit  --> cleanup

error exit   --> cleanup

```

State variables track whether resources are active:

```bash

NBD_CONNECTED=false

MOUNTED=false

```

After NBD connection:

```bash

NBD_CONNECTED=true

```

After mount:

```bash

MOUNTED=true

```

After normal cleanup:

```bash

MOUNTED=false

NBD_CONNECTED=false

```

Therefore, when the script finishes normally, the EXIT trap has nothing left to clean.

If the script fails halfway, the trap releases the remaining resources.

---
## 16. DHCP Discovery
The script waits for libvirt's DHCP server to assign an address.

```bash

for i in {1..30}; do

```

It retries up to 30 times.

Each failed attempt waits:

```bash

sleep 2

```

Maximum wait:

```text

30 × 2 seconds ≈ 60 seconds

```

The current lease is discovered with:

```bash

virsh net-dhcp-leases default

```

The VM hostname is matched exactly:

```bash

awk -v host="${VM_NAME,,}" \

  '$6 == host {split($5,a,"/"); print a[1]}'

```

For the lease table:

```text

$5 = IP address

$6 = Hostname

```

The `/24` prefix is removed before storing the result.

---
## 17. Final Automated Test
Command:

```bash

./provision-vm.sh Alpine-Auto-04

```

Successful output:

```text

Formatting '/var/lib/libvirt/images/Alpine-Auto-04.qcow2',

fmt=qcow2

backing_file=/var/lib/libvirt/images/Alpine-Template-v2.qcow2

/dev/nbd0 disconnected

Starting install...

Creating domain...

Domain creation completed.

Waiting for DHCP lease...

VM Alpine-Auto-04 created successfully.

IP address: 192.168.122.237

SSH:

  ssh airgon@192.168.122.237

```

SSH test:

```bash

ssh airgon@192.168.122.237

```

Result:

```text

Welcome to Alpine!

alpine-auto-04:~$

```

No manual configuration inside the VM was required.

---
## 18. Result
Phase 10 now provides a working reusable VM provisioning pipeline.

One command:

```bash

./provision-vm.sh Alpine-Auto-04

```

performs:

```text

VM name validation

        |

        v

cloud-init metadata generation

        |

        v

qcow2 overlay creation

        |

        v

NBD attachment

        |

        v

guest root filesystem mount

        |

        v

NoCloud seed injection

        |

        v

filesystem sync

        |

        v

NBD cleanup

        |

        v

virt-install --import

        |

        v

first boot + cloud-init

        |

        v

DHCP discovery

        |

        v

SSH-ready Alpine VM

```

This is a repeatable infrastructure provisioning workflow rather than a manual VM installation process.

---
## 19. Skills Practiced
Commands and technologies used in this phase:

```text

virt-install

virsh

qemu-img

qemu-nbd

partprobe

mount

umount

cloud-init

NoCloud

OpenRC

qcow2 backing images

DHCP

SSH keys

Bash variables

Bash functions

trap

heredocs

awk

loops

exit status handling

```

Key infrastructure concepts:

- golden images

- template-based provisioning

- copy-on-write storage

- instance metadata

- first-boot configuration

- unattended provisioning

- offline image customization

- disposable/repeatable VMs

- infrastructure automation

- defensive scripting

---
## 20. Current Limitations
The current script is intentionally lab-focused.

It assumes:

```text

Template:

  /var/lib/libvirt/images/Alpine-Template-v2.qcow2

NBD device:

  /dev/nbd0

Root partition:

  p3

Network:

  libvirt default

VM resources:

  1024 MiB RAM

  2 vCPUs

Guest user:

  airgon

```

Future improvements could include:

- dynamically selecting a free NBD device

- configurable RAM and vCPU values

- selectable libvirt networks

- reusable generic `user-data` template

- automatic SSH readiness checking

- configurable guest usernames

- stronger credential handling

- logging

- more complete rollback after provisioning failure

These are enhancements to the automation layer; the current workflow is already functional and repeatable.

---
## Final Takeaway
The main lesson of this phase is that VM provisioning can be separated into two concerns:

```text

Template image

    = operating system and reusable base configuration

Instance data

    = hostname, users, SSH keys and machine-specific settings

```

By combining qcow2 overlays with cloud-init, a single clean Alpine image can be reused to create many independent VMs without repeatedly installing the operating system.

The final workflow turns VM creation from a manual installation task into a reproducible infrastructure operation.
