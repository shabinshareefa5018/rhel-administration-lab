# Lab 04 — Process & Service Management

## Scenario

As a Linux administrator, managing running processes and system services is essential for maintaining system performance, availability, and stability.

## Objective

Practice process monitoring, process termination, signals, priorities, and systemd service management on **RHEL 9**.

## Environment

* OS: Red Hat Enterprise Linux 9
* Platform: AWS EC2
* User: `ec2-user`

---

## 1. Process Management

### View Current Processes

```bash
ps
```

```bash
ps -ef
```

The `ps` command displays running processes, while `ps -ef` provides a detailed list of processes running on the system.

### Identify Processes

```bash
ps -ef | grep sshd
```

This helps identify SSH-related processes and understand the parent-child process relationship.

### Monitor Processes in Real Time

```bash
top
```

`top` provides real-time information about CPU usage, memory usage, running processes, and system load.

---

## 2. Background Processes

Create a test background process:

```bash
sleep 300 &
```

Find the process:

```bash
ps -ef | grep '[s]leep'
```

Terminate the process:

```bash
kill <PID>
```

Verify that it has stopped:

```bash
ps -ef | grep '[s]leep'
```

---

## 3. Linux Signals

View available signals:

```bash
kill -l
```

Common signals:

* `SIGTERM (15)` — Requests graceful termination
* `SIGKILL (9)` — Forces immediate termination
* `SIGSTOP (19)` — Stops a process
* `SIGCONT (18)` — Continues a stopped process
* `SIGINT (2)` — Interrupts a process

Example:

```bash
kill -15 <PID>
```

Force termination:

```bash
kill -9 <PID>
```

---

## 4. Process States and Priorities

View process states:

```bash
ps -eo pid,ppid,user,stat,cmd
```

Process states can include:

* `R` — Running
* `S` — Sleeping
* `T` — Stopped
* `Z` — Zombie

Run a process with a modified priority:

```bash
nice -n 10 sleep 300 &
```

Change the priority of an existing process:

```bash
renice 5 -p <PID>
```

---

## 5. Service Management with systemd

Check the SSH service:

```bash
systemctl status sshd
```

Start a service:

```bash
sudo systemctl start <service>
```

Restart a service:

```bash
sudo systemctl restart <service>
```

Enable a service at boot:

```bash
sudo systemctl enable <service>
```

Disable a service:

```bash
sudo systemctl disable <service>
```

Check whether a service is enabled:

```bash
systemctl is-enabled <service>
```

Check whether a service is running:

```bash
systemctl is-active <service>
```

List running services:

```bash
systemctl --type=service --state=running
```

> **Note:** Do not stop or disable `sshd` during an active remote SSH session unless you have another way to access the server.

---

## 6. Troubleshooting Services

View system logs:

```bash
journalctl
```

View logs for a specific service:

```bash
journalctl -u <service>
```

These commands help identify service failures and understand why a service may not start correctly.

---


Successfully practiced **Linux process and service management on RHEL 9**, including monitoring processes, terminating test processes, managing systemd services, and analyzing service-related logs.
