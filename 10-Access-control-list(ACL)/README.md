# Lab 10 — Access Control Lists (ACL)

## Objective

Practice configuring and verifying Access Control Lists (ACLs) in RHEL 9 to provide different permissions to individual users.

## Tasks Performed

### 1. Created Users

Created two users for ACL testing:

```bash
sudo useradd alice
sudo passwd alice

sudo useradd bob
sudo passwd bob
```

### 2. Created Shared Directory

```bash
sudo mkdir /shared
sudo chmod 770 /shared
```

### 3. Configured User ACLs

Granted Alice full access:

```bash
sudo setfacl -m u:alice:rwx /shared
```

Granted Bob read and execute access:

```bash
sudo setfacl -m u:bob:rx /shared
```

### 4. Verified ACL Configuration

```bash
getfacl /shared
```

Result:

```text
user::rwx
user:alice:rwx
user:bob:r-x
group::rwx
mask::rwx
other::---
```

### 5. Tested User Access

Logged in as Alice and created a file successfully:

```bash
su - alice
cd /shared
touch alice.txt
```

Logged in as Bob and attempted to create a file:

```bash
su - bob
cd /shared
touch bob.txt
```

Result:

```text
touch: cannot touch 'bob.txt': Permission denied
```

## Result

* Alice can read, write and execute within `/shared`.
* Bob can access and read the directory but cannot create files.
* ACL configuration was verified using `getfacl`.
* User-specific permissions were successfully tested.

## Commands Practiced

```bash
setfacl
getfacl
chmod
useradd
passwd
su
ls -ld
```

## Key Learning

ACLs allow administrators to provide **additional user-specific permissions** without changing the basic ownership and permission structure of a file or directory.

## Environment

* Red Hat Enterprise Linux 9
* AWS EC2
* Users: `alice`, `bob`
* Directory: `/shared`
