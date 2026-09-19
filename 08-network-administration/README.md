# Lab 08 — Network Administration

## Objective

This lab focuses on practical network administration and troubleshooting on a **Red Hat Enterprise Linux 9** server running on **AWS EC2**.

The objective was to understand network interfaces, IP addressing, routing, connectivity, DNS resolution, NetworkManager, listening ports, service-to-port mapping, hostname management, network statistics, and basic network troubleshooting.

---

## Environment

* **Operating System:** Red Hat Enterprise Linux 9.8
* **Platform:** AWS EC2
* **Instance Type:** t3.micro
* **Network Interface:** eth0
* **Private IP:** 172.31.9.198/20
* **Default Gateway:** 172.31.0.1
* **DNS Server:** 172.31.0.2
* **NetworkManager:** Active and enabled

---

# 08.1 — Network Interface Identification

### Commands

```bash
ip addr
ip link
```

### Key Findings

The server contains:

* `lo` — loopback interface
* `eth0` — primary Ethernet interface

The `eth0` interface was in an **UP** state with an MTU of **9001**.

The interface had:

```text
IPv4: 172.31.9.198/20
Broadcast: 172.31.15.255
```

---

# 08.2 — IP Address and Subnet Analysis

### Command

```bash
ip addr show eth0
```

The interface was configured with:

```text
IP Address: 172.31.9.198
Prefix: /20
Broadcast: 172.31.15.255
```

The `/20` prefix identifies the local network as:

```text
172.31.0.0/20
```

---

# 08.3 — Routing Table

### Command

```bash
ip route
```

Output showed:

```text
default via 172.31.0.1 dev eth0 proto dhcp src 172.31.9.198 metric 100
172.31.0.0/20 dev eth0 proto kernel scope link src 172.31.9.198 metric 100
```

### Interpretation

The system has:

* A local route for `172.31.0.0/20`
* A default route through `172.31.0.1`

This allows traffic destined outside the local subnet to be forwarded through the default gateway.

---

# 08.4 — Connectivity Testing

Connectivity was tested at three levels.

### Local Host

```bash
ping -c 4 127.0.0.1
```

Result:

```text
4 packets transmitted, 4 received, 0% packet loss
```

### Default Gateway

```bash
ping -c 4 172.31.0.1
```

Result:

```text
4 packets transmitted, 4 received, 0% packet loss
```

### External Network

```bash
ping -c 4 8.8.8.8
```

Result:

```text
4 packets transmitted, 4 received, 0% packet loss
```

Average response time was approximately:

```text
1.469 ms
```

### Conclusion

The tests confirmed successful:

```text
Local connectivity
        ↓
Gateway connectivity
        ↓
External network connectivity
```

---

# 08.5 — DNS Resolution

### Commands

```bash
getent hosts google.com
getent ahostsv4 google.com
getent ahostsv6 google.com
```

DNS resolution successfully returned both IPv4 and IPv6 addresses for `google.com`.

The configured DNS server was:

```text
172.31.0.2
```

The resolver configuration was managed by NetworkManager.

---

# 08.6 — NetworkManager Administration

### Commands

```bash
nmcli general status
nmcli device status
nmcli device show eth0
nmcli connection show
nmcli connection show "System eth0"
```

### Findings

NetworkManager reported:

```text
STATE: connected
CONNECTIVITY: full
```

The primary connection was:

```text
System eth0
```

The device status showed:

```text
eth0    ethernet    connected    System eth0
lo      loopback    connected (externally)    lo
```

The connection was configured for automatic IPv4 configuration:

```text
ipv4.method: auto
connection.autoconnect: yes
```

NetworkManager was also confirmed to be active and enabled.

---

# 08.7 — Listening Ports

### Commands

```bash
ss -tuln
sudo ss -tulpn
```

The server was listening on:

```text
TCP 22
UDP 323
```

The detailed output identified the associated processes.

```text
TCP 22  → sshd
UDP 323 → chronyd
```

### Interpretation

Port **22** is used by the OpenSSH server for remote administration.

Port **323** is used locally by `chronyd` for time synchronization.

---

# 08.8 — Service-to-Port Mapping

The listening socket information was correlated with the systemd service information from Lab 07.

### SSH

```text
systemd service
      ↓
sshd.service
      ↓
sshd process
      ↓
TCP port 22
```

### Chrony

```text
systemd service
      ↓
chronyd
      ↓
chronyd process
      ↓
UDP port 323
```

This demonstrated how Linux administrators can trace a network port back to the process and service responsible for it.

---

# 08.9 — DNS Troubleshooting

DNS resolution was tested separately for IPv4 and IPv6.

### IPv4

```bash
getent ahostsv4 google.com
```

Successfully returned an IPv4 address.

### IPv6

```bash
getent ahostsv6 google.com
```

Successfully returned IPv6 addresses.

This demonstrated the difference between:

```text
Network connectivity
        ↓
DNS resolution
        ↓
Application connectivity
```

A system can have network connectivity while DNS resolution is failing, so these should be tested separately during troubleshooting.

---

# 08.10 — NetworkManager Connection Inspection

NetworkManager logs were inspected using:

```bash
sudo journalctl -u NetworkManager -n 30 --no-pager
```

The logs showed regular DHCP lease renewals for `eth0`.

Example:

```text
dhcp4 (eth0): state changed new lease, address=172.31.9.198
```

This confirmed that the interface was receiving and renewing its DHCP lease normally.

---

# 08.11 — Hostname Management

### Commands

```bash
hostname
hostnamectl
hostnamectl status
```

The system hostname was:

```text
ip-172-31-9-198.ap-south-1.compute.internal
```

System information confirmed:

```text
Operating System: Red Hat Enterprise Linux 9.8
Kernel: Linux 5.14.0-687.10.1.el9_8.x86_64
Architecture: x86-64
Virtualization: amazon
Hardware Model: t3.micro
```

No hostname configuration changes were required.

---

# 08.12 — Network Statistics

### Command

```bash
ip -s link
```

Network statistics were inspected for both `lo` and `eth0`.

For `eth0`, the observed counters showed:

```text
RX errors: 0
RX dropped: 0

TX errors: 0
TX dropped: 0
```

This provided a basic health check of the network interface.

---

# 08.13 — Connectivity and Path Troubleshooting

### Command

```bash
tracepath 8.8.8.8
```

The path showed the AWS gateway as the first hop and several intermediate hops.

Some hops returned:

```text
no reply
```

This was not treated as a network failure because direct connectivity testing with:

```bash
ping -c 4 8.8.8.8
```

was successful with **0% packet loss**.

### Troubleshooting Lesson

An incomplete `tracepath` result does not necessarily indicate a broken network.

Network troubleshooting should correlate multiple tests rather than relying on a single command.

---

# 08.14 — Network Troubleshooting Workflow

The following troubleshooting workflow was practiced:

```text
1. Check network interface
        ↓
2. Check IP address
        ↓
3. Check routing table
        ↓
4. Test local connectivity
        ↓
5. Test gateway connectivity
        ↓
6. Test external connectivity
        ↓
7. Test DNS resolution
        ↓
8. Check listening ports
        ↓
9. Identify the responsible process
        ↓
10. Check the related systemd service
        ↓
11. Inspect NetworkManager/system logs
        ↓
12. Verify the final network state
```

This provides a structured approach to diagnosing common Linux network problems.

---

# 08.15 — Final Network Health Check

The final checks confirmed that the system was operating normally.

### Interface

```text
eth0 → UP
```

### IP Address

```text
172.31.9.198/20
```

### Gateway

```text
172.31.0.1
```

### NetworkManager

```text
connected
full connectivity
```

### DNS

```text
google.com → successfully resolved
```

### Listening Services

```text
TCP 22 → sshd
UDP 323 → chronyd
```

### Connectivity

```text
127.0.0.1 → 0% packet loss
172.31.0.1 → 0% packet loss
8.8.8.8 → 0% packet loss
```

### Interface Errors

```text
RX errors → 0
RX drops  → 0
TX errors → 0
TX drops  → 0
```

---

# Key Skills Practiced

* Linux network interface administration
* `ip` command
* IP addressing and CIDR
* Routing tables
* Default gateways
* ICMP connectivity testing
* DNS troubleshooting
* `getent`
* NetworkManager
* `nmcli`
* `systemctl`
* `journalctl`
* TCP/UDP listening sockets
* `ss`
* Service-to-port mapping
* Hostname management
* Network statistics
* `tracepath`
* Structured network troubleshooting

---

# Troubleshooting Principles Learned

### 1. Check from the bottom up

Start with the interface and move toward external connectivity.

### 2. Separate network and DNS problems

Successful network connectivity does not automatically mean DNS is working.

### 3. Identify the process behind a port

`ss -tulpn` can help connect a listening port to the process responsible for it.

### 4. Correlate multiple tests

A `tracepath` hop showing `no reply` should not automatically be treated as a failure when direct connectivity is successful.

### 5. Avoid unnecessary configuration changes

On a remote AWS EC2 server, changing networking configuration without understanding the current state can cause loss of connectivity.

---

# Conclusion

Lab 08 provided practical experience with Linux network administration on a RHEL 9 AWS EC2 instance.

The lab covered the complete troubleshooting path from:

```text
Network Interface
       ↓
IP Address
       ↓
Routing
       ↓
Gateway
       ↓
External Connectivity
       ↓
DNS
       ↓
Listening Ports
       ↓
Processes
       ↓
Systemd Services
       ↓
Network Logs
```

This lab strengthened practical skills required for Linux System Administration, Cloud Support, Cloud Engineering, and DevOps environments.

---



Topics will include SSH configuration, authentication, key-based access, SSH security settings, connection troubleshooting, and log analysis.
