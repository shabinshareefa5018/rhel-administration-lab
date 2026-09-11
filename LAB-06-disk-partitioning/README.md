# Lab 06 — Disk Partitioning, Filesystem Creation & Persistent Mount

## 📌 Overview

This lab demonstrates practical Linux storage administration using a secondary disk attached to a RHEL 9 system running on AWS EC2.

The lab covers disk identification, partitioning, filesystem creation, mounting, filesystem verification, testing file access, and configuring a persistent mount using `/etc/fstab`.

## 🎯 Objectives

* Identify attached disks and partitions
* Create a partition on a secondary disk
* Create an XFS filesystem
* Create and use a mount point
* Mount a filesystem manually
* Verify disk usage and mount status
* Test read/write access
* Configure persistent mounting using `/etc/fstab`
* Reload systemd after modifying `fstab`
* Verify the mount configuration

## 🖥️ Environment

| Component        | Details          |
| ---------------- | ---------------- |
| Operating System | RHEL 9           |
| Platform         | AWS EC2          |
| User             | `ec2-user`       |
| Additional Disk  | 2 GB             |
| Partition        | `/dev/nvme1n1p1` |
| Filesystem       | XFS              |
| Mount Point      | `/data`          |

## 🧪 Lab Tasks

### 1. Identify Storage Devices

The available disks were inspected using:

```bash
lsblk
```

The additional 2 GB disk was identified as:

```text
nvme1n1
└─nvme1n1p1
```

### 2. Create a Partition

A partition was created on the secondary disk:

```bash
sudo fdisk /dev/nvme1n1
```

The resulting partition was:

```text
/dev/nvme1n1p1
```

### 3. Create an XFS Filesystem

An XFS filesystem was created on the new partition:

```bash
sudo mkfs.xfs /dev/nvme1n1p1
```

The filesystem UUID was checked using:

```bash
sudo blkid /dev/nvme1n1p1
```

### 4. Create the Mount Point

The `/data` directory was created as the mount point:

```bash
sudo mkdir /data
```

### 5. Mount the Filesystem

The new filesystem was mounted manually:

```bash
sudo mount /dev/nvme1n1p1 /data
```

The mount was verified with:

```bash
df -h /data
```

Result:

```text
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme1n1p1  2.0G   47M  1.9G   3% /data
```

### 6. Test File Creation

A test file was created on the mounted filesystem:

```bash
sudo touch /data/testfile.txt
```

The file was verified using:

```bash
ls -l /data
```

Result:

```text
-rw-r--r--. 1 root root 0 Sep 11 12:00 testfile.txt
```

This confirmed that the mounted filesystem was accessible for file operations.

### 7. Configure Persistent Mounting

The filesystem was configured in:

```bash
/etc/fstab
```

The filesystem UUID was used to identify the partition rather than relying only on the device name.

The configuration follows this format:

```text
UUID=<filesystem-UUID>  /data  xfs  defaults  0  0
```

### 8. Reload systemd

After modifying `/etc/fstab`, systemd was reloaded:

```bash
sudo systemctl daemon-reload
```

The filesystem was then tested using:

```bash
sudo mount /data
```

The system reported that:

```text
/dev/nvme1n1p1 already mounted on /data
```

confirming that the filesystem was already mounted.

### 9. Verify Storage Configuration

The final storage configuration was checked using:

```bash
lsblk
```

```bash
df -h
```

The resulting configuration showed:

```text
nvme1n1     2G
└─nvme1n1p1 2G  /data
```

and:

```text
/dev/nvme1n1p1  2.0G   47M  1.9G   3% /data
```

## 🔧 Commands Practiced

```bash
lsblk
sudo fdisk -l
sudo fdisk /dev/nvme1n1
sudo mkfs.xfs /dev/nvme1n1p1
sudo blkid /dev/nvme1n1p1
sudo mkdir /data
sudo mount /dev/nvme1n1p1 /data
df -h /data
mount | grep /data
findmnt /data
sudo touch /data/testfile.txt
ls -l /data
sudo systemctl daemon-reload
sudo mount /data
```

## 🧠 Key Concepts Learned

### Filesystem

A filesystem provides the structure required to store and manage files on a storage device. XFS is commonly used in RHEL environments.

### Mount Point

A mount point is a directory where a filesystem becomes accessible within the Linux directory hierarchy.

### UUID

A UUID uniquely identifies a filesystem and can be used in `/etc/fstab` for more reliable persistent mounting.

### `/etc/fstab`

The `/etc/fstab` file contains filesystem mounting configurations that Linux can use during system startup.

### `mount -a`

The command:

```bash
sudo mount -a
```

can be used to mount filesystems configured in `/etc/fstab`.

### `findmnt`

The `findmnt` command provides information about currently mounted filesystems.

## ✅ Verification

The lab successfully demonstrated:

* [x] Secondary disk identification
* [x] Disk partitioning
* [x] XFS filesystem creation
* [x] Mount point creation
* [x] Manual filesystem mounting
* [x] Filesystem verification
* [x] Test file creation
* [x] UUID identification
* [x] `/etc/fstab` configuration
* [x] systemd reload
* [x] Persistent mount configuration testing

TROUBLESHOOTING TIP

When `/etc/fstab` is modified on a system using systemd, the following command can be used to reload the configuration:

```bash
sudo systemctl daemon-reload
```

Before rebooting, `/etc/fstab` entries should always be tested carefully to avoid boot-time mounting problems.

## 📚 Skills Demonstrated

**Linux System Administration • RHEL 9 • AWS EC2 • Disk Management • Partitioning • XFS • Filesystem Management • Mounting • ****`/etc/fstab`**** • UUID • systemd • Storage Troubleshooting • Command-Line Administration**
