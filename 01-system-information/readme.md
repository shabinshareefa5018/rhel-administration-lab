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

### 1. Identify the current user

```bash
whoami
2. Check the hostname
hostname
3. Identify the RHEL version
cat /etc/redhat-release
4. Check the kernel version
uname -r
5. Check system uptime
uptime
6. Inspect network interfaces
ip addr
7. Check filesystem usage
df -h
8. Check memory usage
free -h
