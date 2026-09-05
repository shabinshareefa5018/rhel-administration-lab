# Lab 01 – RHEL 9 System Information

## Objective

Perform basic system identification and resource inspection on a
Red Hat Enterprise Linux 9 EC2 instance hosted on AWS.

## Environment

- Cloud Platform: AWS EC2
- Operating System: Red Hat Enterprise Linux 9.8
- User: ec2-user
- Architecture: x86_64

## Tasks Performed

### 1. Identify the Current User

```bash
whoami

Purpose: Verify the user currently logged into the RHEL system.

2. Check the Hostname
hostname

Purpose: Identify the hostname assigned to the server.

3. Identify the RHEL Version
cat /etc/redhat-release

Purpose: Verify the installed Red Hat Enterprise Linux version.

4. Check the Kernel Version
uname -r

Purpose: Display the Linux kernel version currently running.

5. Check System Uptime
uptime

Purpose: Check how long the system has been running and view
the current system load.

6. Inspect Network Interfaces
ip addr

Purpose: Display network interfaces, IP addresses, and interface
status.

7. Check Filesystem Usage
df -h

Purpose: Check disk space usage in human-readable format.

8. Check Memory Usage
free -h

Purpose: Display total, used, free, and available system memory.

Key Findings
The server is running Red Hat Enterprise Linux 9.8.
The system was accessed using the ec2-user account.
The primary network interface is eth0.
The system has a dynamically assigned private IP address.
Disk utilization was checked using df -h.
Memory utilization was checked using free -h.
The system is running on an x86_64 architecture.
Screenshots
System Information

Network, Storage and Memory

Skills Demonstrated
RHEL system identification
Linux command-line administration
Network interface inspection
Filesystem monitoring
Memory monitoring
AWS EC2 Linux administration
Outcome

Successfully connected to a RHEL 9.8 EC2 instance and performed
basic system information and resource inspection using standard
Linux administration commands.