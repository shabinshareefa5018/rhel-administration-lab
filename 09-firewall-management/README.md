# Lab 09 — Firewall Management with firewalld

## 📌 Overview

This lab focuses on firewall administration in **Red Hat Enterprise Linux 9**, using `firewalld`.

The exercises were performed on a **RHEL 9.8 AWS EC2 instance** and cover firewall zones, services, ports, runtime and permanent configurations, rich rules, and basic firewall troubleshooting.

The lab also demonstrates how firewall configuration interacts with **AWS Security Groups**, providing practical experience with layered network security.

---

## 🎯 Objectives

By completing this lab, I practiced:

* Managing the `firewalld` service
* Understanding firewall zones
* Identifying the active firewall zone
* Managing firewall services
* Managing custom TCP ports
* Understanding runtime vs permanent configuration
* Reloading firewall configuration
* Querying firewall rules
* Working with rich rules
* Validating firewall configuration
* Understanding AWS Security Groups and host-based firewalls
* Troubleshooting network access systematically

---

## 🖥️ Environment

| Component         | Details                      |
| ----------------- | ---------------------------- |
| Operating System  | Red Hat Enterprise Linux 9.8 |
| Platform          | AWS EC2                      |
| Architecture      | x86_64                       |
| Firewall          | firewalld                    |
| Default Zone      | public                       |
| Network Interface | eth0                         |
| Firewall Backend  | nftables                     |

---

# 01 — Installing firewalld

Initially, `firewalld` was not installed.

The package was installed using:

```bash
sudo dnf install firewalld -y
```

The instance had limited memory, so a 1 GiB swap file was also configured to provide additional memory capacity for package-management operations.

Swap was created with:

```bash
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

The configuration was made persistent through `/etc/fstab`.

---

# 02 — Enable and Start firewalld

The firewall service was enabled and started:

```bash
sudo systemctl enable --now firewalld
```

Status was verified with:

```bash
sudo systemctl status firewalld
```

Firewall state was checked with:

```bash
sudo firewall-cmd --state
```

Expected result:

```text
running
```

---

# 03 — Firewall Zones

Available zones were listed using:

```bash
sudo firewall-cmd --get-zones
```

The default zone was checked:

```bash
sudo firewall-cmd --get-default-zone
```

Result:

```text
public
```

Active zones were identified with:

```bash
sudo firewall-cmd --get-active-zones
```

Result:

```text
public
  interfaces: eth0
```

This showed that the `eth0` network interface was assigned to the `public` zone.

---

# 04 — Inspecting the Public Zone

The complete configuration of the active zone was examined using:

```bash
sudo firewall-cmd --zone=public --list-all
```

The configuration included services such as:

```text
cockpit dhcpv6-client http ssh
```

and a custom port:

```text
8080/tcp
```

This provided a practical view of how services, ports, interfaces, and advanced rules are represented by `firewalld`.

---

# 05 — Managing Firewall Services

HTTP was added as an allowed service:

```bash
sudo firewall-cmd --permanent --add-service=http
```

The firewall configuration was reloaded:

```bash
sudo firewall-cmd --reload
```

The service was verified:

```bash
sudo firewall-cmd --query-service=http
```

Result:

```text
yes
```

The permanent configuration can also be queried with:

```bash
sudo firewall-cmd --permanent --query-service=http
```

---

# 06 — Managing Custom Ports

Port `8080/tcp` was used as an example of a custom application port.

The port was added with:

```bash
sudo firewall-cmd --permanent --add-port=8080/tcp
```

After reloading:

```bash
sudo firewall-cmd --reload
```

The port was verified:

```bash
sudo firewall-cmd --query-port=8080/tcp
```

This demonstrated how applications that do not have a predefined firewalld service can be exposed through an explicit port rule.

---

# 07 — Runtime vs Permanent Configuration

One of the key concepts practiced in this lab was the difference between **runtime** and **permanent** firewall configuration.

### Runtime configuration

```bash
sudo firewall-cmd --add-port=8080/tcp
```

A runtime rule is immediately active but is not automatically preserved after a restart/reload unless it is made permanent.

### Permanent configuration

```bash
sudo firewall-cmd --permanent --add-port=8080/tcp
```

The firewall must then be reloaded:

```bash
sudo firewall-cmd --reload
```

Runtime configuration can also be transferred to permanent configuration with:

```bash
sudo firewall-cmd --runtime-to-permanent
```

---

# 08 — Rich Rules

Rich rules provide more granular firewall control.

The existing rich rules were inspected using:

```bash
sudo firewall-cmd --list-rich-rules
```

A practice rule was created to allow SSH from a specific IPv4 address:

```bash
sudo firewall-cmd --permanent \
--add-rich-rule='rule family="ipv4" source address="13.232.202.35/32" service name="ssh" accept'
```

The configuration was reloaded:

```bash
sudo firewall-cmd --reload
```

The rule was verified:

```bash
sudo firewall-cmd --list-rich-rules
```

Result:

```text
rule family="ipv4" source address="13.232.202.35/32" service name="ssh" accept
```

### Note

This was performed as a **rich-rule learning exercise**. The address returned by `curl ifconfig.me` was the EC2 instance's public IP, so this should not be interpreted as a production-grade restriction of SSH to the administrator's client machine.

---

# 09 — AWS Security Group and firewalld

This lab also reinforced the concept that an AWS EC2 instance can have multiple network security layers.

A simplified traffic path is:

```text
Internet
   │
   ▼
AWS Security Group
   │
   ▼
EC2 Instance
   │
   ▼
RHEL firewalld
   │
   ▼
Application / Service
```

For example, allowing HTTP access requires consideration of both:

* AWS Security Group rules
* RHEL `firewalld` rules

A connection can fail if either layer blocks the traffic.

---

# 10 — Firewall Troubleshooting

The following commands were practiced for troubleshooting:

### Check firewall state

```bash
sudo firewall-cmd --state
```

### Check active zones

```bash
sudo firewall-cmd --get-active-zones
```

### Inspect zone configuration

```bash
sudo firewall-cmd --zone=public --list-all
```

### Check allowed services

```bash
sudo firewall-cmd --list-services
```

### Check allowed ports

```bash
sudo firewall-cmd --list-ports
```

### Check a specific service

```bash
sudo firewall-cmd --query-service=http
```

### Check a specific port

```bash
sudo firewall-cmd --query-port=8080/tcp
```

### Check listening services

```bash
sudo ss -tulnp
```

These commands provide a systematic approach to determining whether a service is running, listening, and permitted through the host firewall.

---

# 11 — Configuration Validation

The firewall configuration was validated with:

```bash
sudo firewall-cmd --check-config
```

Permanent configuration can be inspected with:

```bash
sudo firewall-cmd --list-all --permanent
```

---

# 🧠 Key Takeaways

This lab demonstrated that firewall administration is more than simply opening and closing ports.

The main concepts practiced were:

* Firewall zones organize network access policies.
* Services provide predefined firewall rules.
* Custom ports can be explicitly allowed.
* Runtime and permanent configurations behave differently.
* Rich rules provide more granular control.
* AWS Security Groups and `firewalld` operate as separate security layers.
* Troubleshooting should verify the service, listening port, firewall, and external network controls systematically.

---

# 📸 Evidence

Screenshots captured for this lab include:

* firewalld service status
* Firewall version and backend
* Available zones
* Active zone and interface
* Public zone configuration
* Allowed services
* HTTP service verification
* Custom port configuration
* Rich rule configuration
* Firewall configuration validation

