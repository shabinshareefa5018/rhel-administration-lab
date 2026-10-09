# Lab 11 — Linux Memory, OOM Troubleshooting & Swap Management

## Overview

This lab focuses on Linux memory management, troubleshooting processes terminated due to insufficient memory, and configuring swap space on a Red Hat Enterprise Linux server.

The lab was performed on a **Red Hat Enterprise Linux 9.8 AWS EC2 t3.micro instance** with limited available memory.

The practical scenario was based on a real package installation problem where the `dnf` process was terminated while attempting to install Apache HTTP Server.

---

## Objectives

By completing this lab, I practiced:

* Monitoring Linux memory usage
* Identifying available and used memory
* Investigating processes terminated by the Linux OOM mechanism
* Reviewing kernel messages
* Searching system logs for memory-related events
* Understanding the impact of limited RAM
* Creating a swap file
* Setting secure swap-file permissions
* Initializing swap space
* Enabling swap
* Making swap persistent through `/etc/fstab`
* Verifying swap availability
* Re-running package installation after increasing available virtual memory

---

## Environment

| Component        | Details                      |
| ---------------- | ---------------------------- |
| Operating System | Red Hat Enterprise Linux 9.8 |
| Platform         | Amazon EC2                   |
| Instance Type    | t3.micro                     |
| Architecture     | x86_64                       |
| RAM              | ~717 MiB                     |
| Swap             | Initially unavailable        |
| Package Manager  | DNF                          |
| Package Tested   | Apache HTTP Server (`httpd`) |

---

# 1. Identify the Problem

While attempting to install Apache HTTP Server using:

```bash
sudo dnf install httpd -y
```

the package-management process was terminated.

The server had very limited available memory.

Memory usage was checked using:

```bash
free -h
```

The system reported approximately:

```text
Mem:    717Mi
```

with no swap initially available.

This indicated that the small EC2 instance had very limited memory resources for package-management operations.

---

# 2. Investigate Memory Usage

The following command was used to inspect memory:

```bash
free -h
```

The output was used to determine:

* Total memory
* Used memory
* Free memory
* Available memory
* Buffer/cache usage

This provided the initial indication that the server had insufficient memory resources.

---

# 3. Investigate Kernel Messages

Kernel messages were checked using:

```bash
sudo dmesg | tail -30
```

Memory-related journal entries were also searched:

```bash
sudo journalctl -k --since "10 minutes ago" | grep -i -E 'killed|oom|out of memory'
```

This is useful when investigating processes that unexpectedly terminate because of memory pressure.

The investigation focused on determining whether the Linux kernel's Out-Of-Memory mechanism had been involved.

---

# 4. Create a Swap File

Because the EC2 instance had limited RAM, a swap file was created.

A 1 GB swap file was allocated:

```bash
sudo fallocate -l 1G /swapfile
```

The resulting file was checked:

```bash
ls -lh /swapfile
```

---

# 5. Secure the Swap File

Swap files should not be writable by ordinary users.

The permissions were therefore restricted:

```bash
sudo chmod 600 /swapfile
```

The permissions were verified:

```bash
ls -l /swapfile
```

Expected permissions:

```text
-rw------- 
```

---

# 6. Initialize the Swap Area

The file was formatted as swap space:

```bash
sudo mkswap /swapfile
```

The command initialized the file so that Linux could use it as swap space.

---

# 7. Enable Swap

The swap file was enabled using:

```bash
sudo swapon /swapfile
```

The active swap areas were checked:

```bash
swapon --show
```

Memory and swap usage were then checked:

```bash
free -h
```

The system now had additional virtual memory available through the swap file.

---

# 8. Configure Persistent Swap

To ensure the swap file would be automatically enabled after reboot, the following entry was added to `/etc/fstab`:

```text
/swapfile none swap sw 0 0
```

It was added using:

```bash
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

The configuration was verified:

```bash
tail -n 1 /etc/fstab
```

---

# 9. Verify Swap

Swap availability was verified using:

```bash
swapon --show
```

and:

```bash
free -h
```

These commands confirmed that the swap file was active.

---

# 10. Re-test Package Installation

After configuring swap, the Apache HTTP Server package installation was attempted again:

```bash
sudo dnf install httpd -y
```

The purpose of this step was to determine whether the additional virtual memory helped the memory-constrained system complete the package-management operation.

---

# 11. Troubleshooting Workflow

The troubleshooting process followed this sequence:

```text
Package installation fails
          ↓
Check memory with free -h
          ↓
Inspect kernel messages
          ↓
Search journal for OOM events
          ↓
Identify memory pressure
          ↓
Create 1 GB swap file
          ↓
Secure swap permissions
          ↓
Initialize swap
          ↓
Enable swap
          ↓
Configure /etc/fstab
          ↓
Verify swap
          ↓
Retry package installation
```

This represents a practical Linux troubleshooting workflow.

---

# Key Commands

### Memory

```bash
free -h
```

### Kernel messages

```bash
sudo dmesg | tail -30
```

### Search for OOM events

```bash
sudo journalctl -k --since "10 minutes ago" | grep -i -E 'killed|oom|out of memory'
```

### Create swap

```bash
sudo fallocate -l 1G /swapfile
```

### Secure swap

```bash
sudo chmod 600 /swapfile
```

### Initialize swap

```bash
sudo mkswap /swapfile
```

### Enable swap

```bash
sudo swapon /swapfile
```

### Verify swap

```bash
swapon --show
free -h
```

### Persistent configuration

```text
/swapfile none swap sw 0 0
```

---

# Important Concepts Learned

## RAM vs Swap

RAM is physical memory used directly by running processes.

Swap is disk space that Linux can use as additional virtual memory when RAM becomes constrained.

Swap is significantly slower than RAM, so it should not be considered a replacement for sufficient physical memory.

---

## Out-Of-Memory Conditions

When a Linux system experiences severe memory pressure, the kernel may terminate processes to recover memory.

This is commonly referred to as an **Out-Of-Memory (OOM) event**.

Useful commands for investigating these situations include:

```bash
free -h
dmesg
journalctl
```

---

## Why Swap Permissions Matter

The swap file was configured with:

```bash
chmod 600 /swapfile
```

This prevents other users from modifying the swap file.

---

# Skills Practiced

* RHEL memory management
* Linux resource monitoring
* OOM troubleshooting
* Kernel log investigation
* systemd journal investigation
* Swap management
* Filesystem permissions
* `/etc/fstab` configuration
* DNF troubleshooting
* AWS EC2 resource troubleshooting

---

# Outcome

A memory-constrained RHEL 9.8 AWS EC2 instance was investigated after a package installation process was terminated.

The investigation included memory monitoring, kernel-message analysis, and journal filtering.

A **1 GB persistent swap file** was configured and verified to provide additional virtual memory for the instance.

This lab demonstrated how Linux administrators can investigate resource-related failures and apply an appropriate corrective action.

---

## Practical Takeaway

> **When a Linux process unexpectedly dies, don't immediately rerun the command. Check system resources and logs first.**

A useful troubleshooting sequence is:

```text
Observe → Investigate → Identify cause → Correct → Verify
```


