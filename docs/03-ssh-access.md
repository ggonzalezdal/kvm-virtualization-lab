# SSH Access and Hardening

## Objective

Configure secure SSH access to an Alpine Linux virtual machine running
on KVM/libvirt using public key authentication.

This guide covers:

-   OpenSSH server configuration
-   ED25519 key authentication
-   SSH key deployment
-   SSH directory permissions
-   Root login hardening
-   OpenSSH configuration validation
-   Best practices for safe SSH administration

------------------------------------------------------------------------

# Lab Environment

## Host

-   Linux Mint 22.2
-   KVM / QEMU
-   libvirt

## Guest

-   Alpine Linux 3.24
-   OpenRC
-   OpenSSH

------------------------------------------------------------------------

# Creating a Normal User

Never use `root` as your daily SSH user.

``` bash
adduser airgon
```

Verify administrative privileges:

``` bash
sudo whoami
```

Expected:

``` text
root
```

------------------------------------------------------------------------

# Copying the SSH Public Key

From the Linux Mint host:

``` bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub airgon@192.168.122.252
```

This copies your public key into:

``` text
/home/airgon/.ssh/authorized_keys
```

------------------------------------------------------------------------

# Testing Key Authentication

``` bash
ssh airgon@192.168.122.252
```

If configured correctly, the remote account password is no longer
required.

------------------------------------------------------------------------

# SSH Directory Permissions

``` bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Verify:

``` bash
ls -ld ~/.ssh
ls -l ~/.ssh/authorized_keys
```

Expected:

``` text
drwx------
-rw-------
```

------------------------------------------------------------------------

# Inspecting the Configuration

Configured directives:

``` bash
sudo grep -E '^(PermitRootLogin|PasswordAuthentication|PubkeyAuthentication)' \
/etc/ssh/sshd_config
```

Effective configuration:

``` bash
sudo sshd -T | grep -E '^(permitrootlogin|passwordauthentication|pubkeyauthentication)'
```

`grep` shows configuration entries.

`sshd -T` shows the effective configuration after defaults and included
files are processed.

------------------------------------------------------------------------

# Using sshd_config.d

Instead of editing the vendor configuration:

``` text
/etc/ssh/sshd_config
```

Create:

``` text
/etc/ssh/sshd_config.d/10-security.conf
```

Contents:

``` text
PermitRootLogin no
```

Advantages:

-   Vendor configuration remains untouched.
-   Easier package upgrades.
-   Cleaner configuration management.
-   Matches modern Linux administration practices.

------------------------------------------------------------------------

# Validate Before Restart

``` bash
sudo sshd -t
```

No output means the configuration is valid.

------------------------------------------------------------------------

# Restart SSH

``` bash
sudo rc-service sshd restart
```

------------------------------------------------------------------------

# Verify the Result

``` bash
sudo sshd -T | grep '^permitrootlogin'
```

Expected:

``` text
permitrootlogin no
```

------------------------------------------------------------------------

# Test the Configuration

Normal user:

``` bash
ssh airgon@192.168.122.252
```

Root:

``` bash
ssh root@192.168.122.252
```

Expected:

``` text
Permission denied
```

------------------------------------------------------------------------

# Best Practices

-   Use public key authentication.
-   Create a normal administrative user.
-   Use `sudo` instead of direct root login.
-   Validate configuration before restarting services.
-   Keep one SSH session open while testing changes.
-   Store local overrides inside `sshd_config.d`.
-   Create a VM snapshot after completing an important milestone.

------------------------------------------------------------------------

# Snapshot

**Name**

``` text
03-ssh-access
```

**Description**

``` text
SSH configured with ED25519 key authentication for airgon.
Root SSH login disabled using sshd_config.d override.
Password authentication intentionally left enabled for recovery during the lab.
```

------------------------------------------------------------------------

# LPIC-1 Notes

## Commands

``` text
ssh
sshd
ssh-copy-id
chmod
sudo
grep
vi
```

## Configuration Files

``` text
/etc/ssh/sshd_config
/etc/ssh/sshd_config.d/
/home/<user>/.ssh/
/home/<user>/.ssh/authorized_keys
```

## Important Directives

``` text
PermitRootLogin
PasswordAuthentication
PubkeyAuthentication
```

## sshd Testing

``` bash
sshd -t
sshd -T
```

-   `-t` → Validate configuration syntax.
-   `-T` → Print the effective configuration.
