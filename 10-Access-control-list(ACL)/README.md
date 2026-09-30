# Lab 10 — ACL Access Control

## Objective

Practice using Access Control Lists (ACLs) in RHEL to provide different permissions to individual users.

## Environment

* RHEL 9
* AWS EC2
* Users: `alice`, `bob`
* Directory: `/shared`

## Tasks

### 1. Create Users

```bash
sudo useradd alice
sudo passwd alice

sudo useradd bob
sudo passwd bob
```

### 2. Create Shared Directory

```bash
sudo mkdir /shared
sudo chmod 770 /shared
```

### 3. Configure ACL

Give Alice full access:

```bash
sudo setfacl -m u:alice:rwx /shared
```

Give Bob read and execute access:

```bash
sudo setfacl -m u:bob:rx /shared
```

### 4. Verify ACL

```bash
getfacl /shared
```

Expected entries:

```text
user::rwx
user:alice:rwx
user:bob:r-x
group::rwx
mask::rwx
other::---
```

### 5. Test Access

As Alice:

```bash
su - alice
cd /shared
touch alice.txt
```

Alice successfully created the file.

As Bob:

```bash
su - bob
cd /shared
touch bob.txt
```

Bob received:

```text
Permission denied
```

## Result

The ACL configuration worked as expected:

* Alice can create files in `/shared`.
* Bob can access the directory but cannot create files.
* ACL entries were verified using `getfacl`.

## Commands Practiced

```text
useradd
passwd
chmod
setfacl
getfacl
su
```

## Key Learning

ACLs provide additional user-specific permissions while keeping the normal Linux ownership and permission structure intact.
