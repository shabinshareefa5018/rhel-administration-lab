# Lab 15 — Bash Scripting Foundations

## Overview

This lab documents my journey into Bash scripting for Linux administration and DevOps.

I started with basic Bash commands and gradually progressed to variables, conditions, loops, functions, script arguments, and Linux system checks.

As a final practical exercise, I developed a basic **Linux Server Health Check** script on an AWS EC2 Linux instance.

The objective was to understand Bash fundamentals and learn how to combine Linux commands to automate repetitive administration tasks.

## Learning Environment

- Cloud Platform: AWS EC2
- Operating System: Linux
- Shell: Bash
- Access: SSH
- Editor: Vim / Nano

## Learning Objectives

- Understand Bash script structure and execution.
- Work with variables and user input.
- Implement conditions and comparisons.
- Use loops to automate repetitive operations.
- Create reusable functions.
- Pass arguments to scripts.
- Validate files and directories.
- Check disk and memory usage.
- Verify Linux service status.
- Combine Linux commands into practical scripts.

---

## Lessons Covered

### Lesson 1 — Bash Fundamentals

Topics:
- Shebang (`#!/bin/bash`)
- `echo` and displaying output
- Variables
- User input using `read`
- Command substitution
- Executing Bash scripts

### Lesson 2 — Conditions and Decision Making

Topics:
- `if`, `else`, and `elif`
- Numeric and string comparisons
- File and directory tests
- Exit status (`$?`)
- Basic service checks

### Lesson 3 — Loops

Topics:
- `for` loops
- `while` loops
- Counters
- `break` and `continue`
- Processing multiple files and paths

### Lesson 4 — Functions and Script Arguments

Topics:
- Defining and calling functions
- Passing arguments
- `$1`, `$2`, `$#`, and `$@`
- Local variables using `local`
- `return` and `exit`
- Basic input validation

**Mini Project: Linux Administration Toolkit**

Created a script that:
- Displays the system hostname.
- Displays the current user.
- Checks whether supplied paths are files or directories.
- Identifies missing paths.

Example:

```bash
bash lesson4.sh /etc/passwd /var/log /missing/path
```

### Lesson 5 — Linux Server Health Check

Created a basic Bash script to collect system information and check important server resources.

**Script:** `lesson5.sh`

Functions included:

| Function | Purpose |
|---|---|
| `show_system_info()` | Displays hostname, current user, and date |
| `check_disk()` | Displays filesystem usage using `df -h` |
| `check_disk_warning()` | Identifies filesystems with usage of 80% or higher |
| `check_memory()` | Displays memory usage using `free -h` |
| `check_ssh()` | Checks whether the SSH service is active |

Linux commands and utilities practiced:

```bash
hostname
whoami
date
df -h
df -P
free -h
systemctl is-active sshd
awk
```

The disk warning check uses `awk` to extract filesystem utilization and identify filesystems that reach the configured threshold.

---

## Final Project — Linux Server Health Check

### Execution

```bash
bash lesson5.sh
```

### Observed Results

The script executed successfully on my EC2 instance.

| Check | Observed Result |
|---|---|
| Hostname | Displayed successfully |
| Current user | ec2-user |
| Root filesystem usage | 20% |
| Disk warnings | No warnings |
| Total memory | 906 MiB |
| Available memory | 538 MiB |
| Swap | 0 B |
| SSH service | Active |

**Result:** Successfully executed a basic Linux server health-check script.

Note: These are observations from one execution, not permanent server health guarantees.

---

## Useful Bash Commands

### Syntax validation

```bash
bash -n lesson5.sh
```

Checks for syntax errors without executing the script.

### Execute a script

```bash
bash lesson5.sh
```

### Debug a script

```bash
bash -x lesson5.sh
```

Displays commands as Bash executes them, helping with troubleshooting.

---

## Key Learnings

Through these exercises, I gained practical experience with:

- Bash scripting fundamentals.
- Conditions, loops, and functions.
- Script arguments and input validation.
- Automating repetitive Linux checks.
- Using `awk` to process command output.
- Checking disk, memory, and service status.
- Debugging and validating Bash scripts.

## Future Learning Goals

- Bash error handling and input validation.
- Output redirection and logging.
- Text processing using `grep`, `awk`, and `sed`.
- Linux log analysis automation.
- More practical Linux administration scripts.
- Integration with Ansible automation.

## Medium Documentation

Read the complete learning journey and practical implementation:

**[RHEL 9 Linux Administration Lab — Part 3: Bash Scripting & Linux Server Health Check](https://medium.com/@shabinshareefainfo/rhel-9-linux-administration-lab-part-3-bash-scripting-linux-server-health-check-05e16de1ee66)**

## Repository

This lab is part of my ongoing Linux administration practice.

**GitHub:** https://github.com/shabinshareefa5018/rhel-administration-lab

---

*Learning Linux administration through hands-on practice, troubleshooting, and automation — one lab at a time.*
