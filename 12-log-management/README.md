# Lab 12 — Linux Log Management & System Troubleshooting

## Overview

This lab focuses on Linux system log management, service investigation, SSH troubleshooting, and basic incident analysis using Red Hat Enterprise Linux 9.

The lab was performed on a **Red Hat Enterprise Linux 9.8 AWS EC2 t3.micro instance**.

The objective was to develop a practical troubleshooting workflow using `systemctl`, `journalctl`, `sshd`, and `ss`.

---

## Objectives

By completing this lab, I practiced:

* Checking overall Linux system health
* Identifying failed systemd services
* Inspecting the system journal
* Filtering journal entries by priority
* Filtering logs by boot and time
* Investigating SSH service logs
* Searching logs for errors and failures
* Validating SSH configuration
* Troubleshooting a controlled SSH configuration error
* Verifying SSH service availability
* Checking listening network sockets
* Collecting troubleshooting evidence
* Documenting system findings

---

## Environment

| Component            | Details                      |
| -------------------- | ---------------------------- |
| Operating System     | Red Hat Enterprise Linux 9.8 |
| Platform             | Amazon EC2                   |
| Instance Type        | t3.micro                     |
| Architecture         | x86_64                       |
| Kernel               | 5.14.0-687.10.1.el9_8.x86_64 |
| Service Investigated | sshd                         |
| Logging System       | systemd-journald             |
| Access Method        | SSH                          |

---

# 1. Check System Health

The initial system state was checked using:

```bash
hostnamectl
uptime
who
date
systemctl --failed
```

### Result

The system was running normally.

```text
Uptime: 2 days, 5 hours
Failed systemd units: 0
```

The command:

```bash
systemctl --failed
```

returned:

```text
0 loaded units listed.
```

This confirmed that there were no currently failed systemd services.

---

# 2. Inspect the System Journal

Recent system logs were reviewed using:

```bash
sudo journalctl -n 50
```

The current boot journal was examined with:

```bash
sudo journalctl -b
```

The available boots were checked using:

```bash
sudo journalctl --list-boots
```

Live journal monitoring was also practiced:

```bash
sudo journalctl -f
```

---

# 3. Filter Logs by Priority

System errors were investigated using:

```bash
sudo journalctl -p err
```

Warnings were reviewed using:

```bash
sudo journalctl -p warning
```

Errors from the current boot were filtered with:

```bash
sudo journalctl -b -p err
```

This helped identify error-level events without manually reviewing the complete journal.

---

# 4. Investigate SSH Service

SSH is a critical service because the AWS EC2 instance is administered remotely.

The service status was checked:

```bash
sudo systemctl status sshd
```

The SSH journal was investigated:

```bash
sudo journalctl -u sshd
```

Recent SSH events were reviewed:

```bash
sudo journalctl -u sshd -n 30
```

SSH logs from the current boot were also inspected:

```bash
sudo journalctl -b -u sshd
```

### Result

The SSH service was found to be active.

```bash
sudo systemctl is-active sshd
```

Result:

```text
active
```

---

# 5. Time-Based Log Investigation

Journal entries were filtered using different time ranges:

```bash
sudo journalctl --since "1 hour ago"
```

```bash
sudo journalctl --since "30 minutes ago"
```

```bash
sudo journalctl --since today
```

```bash
sudo journalctl --since yesterday --until today
```

Time-based filtering is useful when troubleshooting an incident where the approximate failure time is known.

---

# 6. Search System Logs

The journal was searched for common troubleshooting keywords:

```bash
sudo journalctl | grep -i error
```

```bash
sudo journalctl | grep -i failed
```

```bash
sudo journalctl | grep -i ssh
```

```bash
sudo journalctl | grep -i authentication
```

This demonstrated how standard Linux text-processing tools can be combined with `journalctl`.

---

# 7. SSH Configuration Validation

Before making changes to SSH configuration, a backup was created:

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.lab12-backup
```

The SSH configuration was validated using:

```bash
sudo sshd -t
```

A successful validation produced no output.

This is an important administrative practice because SSH configuration errors can potentially prevent remote access after a service restart.

---

# 8. Controlled SSH Troubleshooting Scenario

A controlled configuration error was introduced into the SSH configuration to simulate a real troubleshooting scenario.

The configuration was then tested with:

```bash
sudo sshd -t
```

The validation process identified the configuration problem.

The invalid configuration was removed and the configuration was tested again.

```bash
sudo sshd -t
```

The configuration then passed validation.

The SSH service was subsequently reloaded and verified.

---

# 9. Verify SSH Service

The SSH service was verified using:

```bash
sudo systemctl is-active sshd
```

Result:

```text
active
```

The SSH configuration was validated again:

```bash
sudo sshd -t
```

No errors were returned.

---

# 10. Verify SSH Network Socket

The listening SSH port was checked using:

```bash
sudo ss -tlnp | grep :22
```

The server was listening on:

```text
0.0.0.0:22
[::]:22
```

This confirmed that SSH was listening on both IPv4 and IPv6 sockets.

---

# 11. Security Observation

During log investigation, repeated SSH authentication attempts against invalid usernames were observed.

Examples included attempts using usernames such as:

```text
admin
oracle
usuario
test
ubuntu
root
```

The journal entries showed failed/pre-authentication attempts.

These observations demonstrate why publicly accessible SSH services should be monitored and appropriately secured.

The external IP addresses observed during the investigation were intentionally **not included in this repository documentation**.

---

# 12. Troubleshooting Evidence

Evidence was collected locally under:

```text
~/lab12-evidence/
```

The following files were created:

```text
system-info.txt
sshd-status.txt
sshd-logs.txt
system-errors.txt
troubleshooting-report.txt
```

These files contain the system state, service information, relevant logs, and troubleshooting findings from the lab.

Raw log files containing external connection information were kept locally rather than uploaded to the public repository.

---

# Troubleshooting Workflow

The following workflow was practiced during the lab:

```text
Check system state
       ↓
systemctl --failed
       ↓
Identify affected service
       ↓
systemctl status <service>
       ↓
journalctl -u <service>
       ↓
Filter by time / priority
       ↓
Identify the problem
       ↓
Make a controlled correction
       ↓
Validate configuration
       ↓
Verify service
       ↓
Check network socket
       ↓
Collect evidence
       ↓
Document findings
```

---

# Key Commands

### System health

```bash
hostnamectl
uptime
systemctl --failed
```

### Service management

```bash
systemctl status sshd
systemctl is-active sshd
```

### Journal investigation

```bash
journalctl
journalctl -b
journalctl -u sshd
journalctl -p err
journalctl -b -p err
```

### Time filtering

```bash
journalctl --since "1 hour ago"
journalctl --since today
```

### SSH validation

```bash
sshd -t
```

### Network socket verification

```bash
ss -tlnp
```

### Log searching

```bash
journalctl | grep -i error
journalctl | grep -i failed
journalctl | grep -i ssh
```

---

# Skills Practiced

* RHEL 9 system administration
* systemd service management
* `journalctl`
* Linux log analysis
* SSH administration
* SSH configuration validation
* Service troubleshooting
* Time-based incident investigation
* Network socket verification
* Basic security log analysis
* Troubleshooting documentation

---

# Outcome

The server was successfully investigated and returned to a verified healthy state.

Final checks confirmed:

* No failed systemd units
* SSH service active
* SSH configuration valid
* SSH listening on port 22
* System logs successfully investigated
* Troubleshooting evidence collected
* Security-related SSH log activity identified and documented

The main troubleshooting principle practiced in this lab was:

> **Investigate first, make a controlled change, validate the change, and verify the service afterward.**

---


