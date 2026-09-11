# Lab 05 — Package Management

## Scenario

As a Linux administrator, managing software packages is an essential task for maintaining applications, security updates, and system functionality.

## Objective

Practice package management on **RHEL 9** using **DNF and RPM**, including package searching, installation, removal, verification, updates, and troubleshooting.

## Environment

* OS: Red Hat Enterprise Linux 9
* Platform: AWS EC2
* User: `ec2-user`

---

## 1. DNF Basics

### Check DNF Version

```bash
dnf --version
```

### View Configured Repositories

```bash
dnf repolist
```

### Search for a Package

```bash
dnf search nginx
```

### Display Package Information

```bash
dnf info nginx
```

---

## 2. Install and Remove Packages

### Install Apache HTTP Server

```bash
sudo dnf install httpd -y
```

### Verify Installation

```bash
rpm -q httpd
```

```bash
httpd -v
```

### Display Package Information

```bash
rpm -qi httpd
```

### List Files Installed by the Package

```bash
rpm -ql httpd
```

### Remove the Package

```bash
sudo dnf remove httpd -y
```

### Verify Removal

```bash
rpm -q httpd
```

Expected result:

```text
package httpd is not installed
```

---

## 3. Package Updates

### Check for Available Updates

```bash
dnf check-update
```

> `dnf check-update` may return exit code `100` when updates are available. This is expected behavior.

### View DNF Transaction History

```bash
sudo dnf history
```

### View Transaction Details

```bash
sudo dnf history info
```

---

## 4. RPM Package Management

### Check Whether a Package Is Installed

```bash
rpm -q bash
```

### Find Which Package Provides a File

```bash
rpm -qf /usr/bin/bash
```

### Verify Package Files

```bash
rpm -V bash
```

---

## 5. Package Troubleshooting

### List Installed Packages

```bash
dnf list installed
```

### Search for a Package

```bash
dnf search <package-name>
```

### List Available Packages

```bash
dnf list available <package-name>
```

### Review Package Transactions

```bash
sudo dnf history
```

These commands help identify installed packages, available packages, package versions, and previous package-management transactions.

---

## 6. Practical Challenge

The following task was performed as a practical package-management exercise:

1. Search for the `nginx` package.
2. Display package information.
3. Install the package.
4. Verify the installation.
5. Check the installed version.
6. List the files installed by the package.
7. Remove the package.
8. Verify that it was removed.
9. Review the DNF transaction history.

---

## Key Takeaways

This lab provided practical experience with:

* DNF package management
* RPM package queries
* Package installation and removal
* Package information and file verification
* Repository management
* Checking package updates
* DNF transaction history
* Basic package troubleshooting

## Result

Successfully practiced **software package management on RHEL 9** using DNF and RPM, including installing, verifying, removing, and troubleshooting packages.

## Screenshots

Screenshots demonstrating the package-management tasks are available in the `screenshots/` directory.

