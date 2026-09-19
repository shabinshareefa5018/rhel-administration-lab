# Lab 07 — systemd & Service Management

## Overview

This lab focuses on **systemd service and system management in RHEL 9**.

The objective was to understand how systemd manages services, processes, targets, startup behavior, service logs, and troubleshooting.

The exercises were performed on an **AWS EC2 instance running Red Hat Enterprise Linux 9.8**.

---

## Objectives

By completing this lab, I practiced:

* Understanding systemd
* Listing and inspecting services
* Checking service status
* Starting, stopping, and restarting services
* Enabling and disabling services
* Using `enable --now`
* Using `mask` and `unmask`
* Understanding systemd targets
* Inspecting service dependencies
* Reading service logs with `journalctl`
* Identifying service-related problems from system logs
* Troubleshooting services without unnecessarily modifying a working system

---

## Environment

| Component          | Details                      |
| ------------------ | ---------------------------- |
| Cloud Platform     | AWS EC2                      |
| Operating System   | Red Hat Enterprise Linux 9.8 |
| Init System        | systemd                      |
| systemd Version    | 252                          |
| Access             | SSH                          |
| Main Service Used  | `chronyd`                    |
| Additional Service | `sshd`                       |
| Logging            | `journalctl`                 |

---

# 07.1 — systemd Basics

First, the systemd version and currently loaded services were inspected.

```bash
systemctl --version
systemctl list-units --type=service
systemctl list-units --type=service --state=running
systemctl --failed
systemctl get-default
```

### Observations

* systemd version: **252**
* Loaded services: approximately **40**
* Running services: approximately **17**
* Failed units: **0**
* Default target: `multi-user.target`

This established the current system state before performing service-management operations.

---

# 07.2 — Service Inspection

The SSH service was used to understand service properties.

```bash
systemctl status sshd
systemctl is-active sshd
systemctl is-enabled sshd
systemctl show sshd -p FragmentPath
systemctl show sshd -p MainPID
systemctl show sshd -p ActiveEnterTimestamp
```

The running process was also inspected:

```bash
ps -fp $(systemctl show -p MainPID --value sshd)
```

### Important Observation

The SSH service was:

```text
active
```

but:

```text
disabled
```

This demonstrates an important system administration concept:

> **Active** describes the current runtime state, while **enabled** describes whether a service is configured to start automatically during boot.

---

# 07.3 — Viewing Service Logs

Service-specific logs were examined using `journalctl`.

```bash
sudo journalctl -u sshd -n 10 --no-pager
```

The logs demonstrated SSH connection and authentication events.

`journalctl` provides an important troubleshooting interface for systemd-managed services.

---

# 07.4 — Service Lifecycle Management

The `chronyd` service was used to practice the service lifecycle.

Initial state:

```bash
systemctl is-active chronyd
systemctl is-enabled chronyd
```

The service process was inspected:

```bash
systemctl show chronyd -p MainPID
ps -fp $(systemctl show -p MainPID --value chronyd)
```

Service logs were reviewed:

```bash
sudo journalctl -u chronyd -n 10 --no-pager
```

### Restart

```bash
sudo systemctl restart chronyd
```

The service was then verified:

```bash
systemctl is-active chronyd
systemctl show chronyd -p MainPID
```

The Main PID changed after the restart, demonstrating that the service process had been recreated.

### Stop

The service was deliberately stopped:

```bash
sudo systemctl stop chronyd
```

Verification:

```bash
systemctl is-active chronyd
systemctl status chronyd --no-pager
systemctl show chronyd -p MainPID
```

The service entered:

```text
inactive (dead)
```

and its Main PID became:

```text
0
```

### Recovery

The service was started again:

```bash
sudo systemctl start chronyd
```

Verification:

```bash
systemctl is-active chronyd
systemctl status chronyd --no-pager
systemctl show chronyd -p MainPID
pgrep -a chronyd
```

The service returned to an active running state.

---

# 07.5 — Enable and Disable Services

The difference between runtime state and boot configuration was further demonstrated using `chronyd`.

Check the current state:

```bash
systemctl is-active chronyd
systemctl is-enabled chronyd
```

Disable automatic startup:

```bash
sudo systemctl disable chronyd
```

Verify:

```bash
systemctl is-active chronyd
systemctl is-enabled chronyd
```

The service remained **active** even though it became **disabled**.

It was then enabled again:

```bash
sudo systemctl enable chronyd
```

Verification:

```bash
systemctl is-active chronyd
systemctl is-enabled chronyd
```

### Key Concept

```text
active + enabled
```

means the service is currently running and configured to start automatically.

```text
active + disabled
```

means the service is currently running but is not configured for automatic startup.

---

# 07.6 — `enable --now`

The `enable --now` option was tested:

```bash
sudo systemctl enable --now chronyd
```

Verification:

```bash
systemctl is-active chronyd
systemctl is-enabled chronyd
```

This combines two operations:

```text
enable → configure service to start at boot
now    → start the service immediately
```

---

# 07.7 — Mask and Unmask Services

The stronger service-blocking mechanism `mask` was tested.

```bash
sudo systemctl mask chronyd
```

Verification:

```bash
systemctl is-enabled chronyd
```

Result:

```text
masked
```

An attempt was then made to start the service:

```bash
sudo systemctl start chronyd
```

systemd rejected the operation because the service was masked.

### Unmask

The service was restored:

```bash
sudo systemctl unmask chronyd
```

Then it was enabled and started:

```bash
sudo systemctl enable --now chronyd
```

Final verification:

```bash
systemctl is-active chronyd
systemctl is-enabled chronyd
```

### Key Concept

| Command        | Purpose                            |
| -------------- | ---------------------------------- |
| `start`        | Start service now                  |
| `stop`         | Stop service now                   |
| `restart`      | Restart service                    |
| `enable`       | Start automatically at boot        |
| `disable`      | Remove automatic boot startup      |
| `mask`         | Prevent service from being started |
| `unmask`       | Remove the mask                    |
| `enable --now` | Enable and start immediately       |

---

# 07.8 — systemd Targets

The default system target was checked:

```bash
systemctl get-default
```

Result:

```text
multi-user.target
```

Available targets were inspected:

```bash
systemctl list-units --type=target
systemctl list-unit-files --type=target
```

Dependencies of the multi-user target were examined:

```bash
systemctl list-dependencies multi-user.target --type=target
```

The target configuration was also inspected:

```bash
systemctl cat multi-user.target
```

### Important Observation

The dependency tree demonstrated how systemd builds the operating environment through multiple targets and services.

It included services such as:

* `auditd.service`
* `chronyd.service`
* `crond.service`
* `NetworkManager.service`
* `rsyslog.service`
* `tuned.service`

It also included targets related to:

* networking
* local filesystems
* remote filesystems
* cloud initialization
* login/getty services
* timers

The `data.mount` entry also connected this lab with **Lab 06 — Disk Partitioning & Storage**, where `/data` was configured as a persistent mount.

---

# 07.9 — Journal and System Troubleshooting

System logs were examined using `journalctl`.

Service-specific logs:

```bash
sudo journalctl -u chronyd --since "10 minutes ago" --no-pager
```

Warnings for a service:

```bash
sudo journalctl -u chronyd -p warning --no-pager
```

Boot errors:

```bash
sudo journalctl -b -p err --no-pager
```

The boot error logs demonstrated how historical system events can be investigated.

Among the historical events observed were:

* SSH connection failures
* failed authentication attempts
* kernel warnings
* memory pressure / OOM events
* a historical `dnf makecache` failure

These logs were treated as **troubleshooting evidence rather than automatically assuming the system was currently broken**.

---

# 07.10 — Troubleshooting Exercise

System memory was checked:

```bash
free -h
```

The system showed approximately:

```text
Memory: ~717 MiB
Swap:   0 B
```

This helped explain why memory pressure could be significant on this small EC2 instance.

The DNF timer was then inspected:

```bash
systemctl status dnf-makecache.timer --no-pager
```

The associated service was checked:

```bash
systemctl status dnf-makecache.service --no-pager
```

Historical logs were reviewed:

```bash
sudo journalctl -u dnf-makecache.service -n 20 --no-pager
```

The investigation showed that although older journal entries contained DNF-related memory pressure, the latest `dnf-makecache` execution had completed successfully.

### Troubleshooting Principle

A useful Linux administrator should distinguish between:

```text
Historical failure
```

and:

```text
Current failure
```

Instead of changing a working system unnecessarily, the current service state and recent logs were verified first.

---

# Key Commands Practiced

```bash
systemctl status <service>
systemctl is-active <service>
systemctl is-enabled <service>

systemctl start <service>
systemctl stop <service>
systemctl restart <service>

systemctl enable <service>
systemctl disable <service>

systemctl enable --now <service>

systemctl mask <service>
systemctl unmask <service>

systemctl list-units --type=service
systemctl list-unit-files --type=service

systemctl get-default
systemctl list-units --type=target
systemctl list-dependencies <target>
systemctl cat <target>

journalctl -u <service>
journalctl -b
journalctl -p err
journalctl -p warning
```

---

# What I Learned

Through this lab, I practiced:

* Managing systemd services
* Inspecting service processes and PIDs
* Understanding active vs enabled states
* Managing service startup behavior
* Using service masking for stronger service control
* Understanding systemd targets
* Inspecting service dependencies
* Connecting systemd with persistent storage
* Using `journalctl` for troubleshooting
* Investigating historical failures without assuming they are current
* Verifying service recovery after intentional failures

---

# Practical Troubleshooting Workflow

The following workflow was used during the troubleshooting exercises:

```text
1. Check current service status
        ↓
2. Check active/enabled state
        ↓
3. Inspect Main PID/process
        ↓
4. Review service logs
        ↓
5. Check related system resources
        ↓
6. Identify whether the issue is current or historical
        ↓
7. Make the smallest appropriate change
        ↓
8. Verify recovery
```

---

# Lab Status

**Status: Completed**

This lab is part of my ongoing **RHEL Administration Lab** portfolio.

Previous lab:

* [Lab 06 — Disk Partitioning & Storage](../Lab-06-Disk-Partitioning)

Next lab:

* **Lab 08 — Network Administration**

---

## Skills Demonstrated

`RHEL 9` `Linux Administration` `systemd` `systemctl` `journalctl` `Service Management` `Process Management` `Troubleshooting` `AWS EC2` `Linux Networking Basics` `System Monitoring`
