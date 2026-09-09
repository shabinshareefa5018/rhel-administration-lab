03 — Linux File Permissions
📌 Overview

This lab focuses on Linux file permissions, ownership, group-based access, directory permissions, and basic ACL concepts using RHEL 9.

The lab demonstrates how Linux controls access to files and directories and how permission issues can be identified and troubleshot from the command line.

🎯 Objectives

By completing this lab, I practiced:

Understanding Linux permission modes
Creating and modifying file permissions
Changing group ownership
Using groups for shared file access
Testing access as different users
Understanding directory traversal permissions
Troubleshooting Permission denied
Inspecting ACL information using getfacl
🖥️ Environment
Component	Details
OS	RHEL 9
Platform	AWS EC2
Primary User	ec2-user
Test User	devuser
Group	devops
Shell	Bash
1. Verify Current User

First, I verified the currently logged-in user and group membership.

whoami
id

Example output:

ec2-user

The id command displays:

UID
Primary GID
Supplementary groups
SELinux security context
2. Create a Group

I created a devops group for testing group-based access.

sudo groupadd devops

Verify the group:

getent group devops

Example:

devops:x:1001:
3. Create a User

A development user was created for permission testing.

sudo useradd -m -s /bin/bash devuser

Verify the user:

id devuser

Check the account information:

getent passwd devuser

The -m option creates the user's home directory and -s /bin/bash assigns Bash as the login shell.

4. Set User Password
sudo passwd devuser

The system may display a password-quality warning when a weak test password is entered.

For production systems, strong passwords should always be used according to the organization's password policy.

5. Add User to the DevOps Group
sudo usermod -aG devops devuser

Verify:

id devuser

Expected result:

groups=1002(devuser),1001(devops)
Why use -aG?
-G specifies supplementary groups.
-a appends the new group instead of replacing existing supplementary group memberships.
6. Test User Login

Switch to the newly created user:

su - devuser

Verify:

whoami
pwd

Expected:

devuser
/home/devuser

Exit:

exit
7. Create Permission Lab Directory

As ec2-user:

mkdir ~/permission-lab
cd ~/permission-lab

Create a test file:

echo "Linux permissions practice" > testfile.txt

Check the initial permissions:

ls -l testfile.txt

Example:

-rw-r--r--. 1 ec2-user ec2-user 27 testfile.txt

The initial permission mode is:

644
8. Modify File Permissions
Set permission to 600
chmod 600 testfile.txt

Verify:

ls -l testfile.txt

Result:

-rw-------. 1 ec2-user ec2-user 27 testfile.txt

This means:

User	Permission
Owner	Read + Write
Group	None
Others	None
Change permission to 640
chmod 640 testfile.txt

Verify:

ls -l testfile.txt

Result:

-rw-r-----. 1 ec2-user ec2-user 27 testfile.txt

Permission breakdown:

Owner  → rw-
Group  → r--
Others → ---

Therefore:

640 = rw-r-----
9. Change Group Ownership

The file group was changed from ec2-user to devops.

sudo chgrp devops testfile.txt

Verify:

ls -l testfile.txt

Result:

-rw-r-----. 1 ec2-user devops 27 testfile.txt

The ownership is now:

Owner → ec2-user
Group → devops

This allows group-based permission testing.

10. Test Access as devuser

Switch to devuser:

su - devuser

Verify group membership:

id

The user should be a member of:

devops

Attempt to read the file:

cat /home/ec2-user/permission-lab/testfile.txt

Initially, access failed:

Permission denied
11. Troubleshoot the Permission Denied Error

The file had group-read permission:

-rw-r-----. 1 ec2-user devops testfile.txt

Since devuser belonged to the devops group, the file permissions appeared correct.

However, Linux also checks permissions on the directories leading to the file.

I checked the home directory:

ls -ld /home/ec2-user

The result showed:

drwx------. 4 ec2-user ec2-user ...

This corresponds to:

700

Only ec2-user had access to the directory.

Therefore, devuser could not traverse /home/ec2-user to reach the file.

12. Understand Directory Traversal

This lab demonstrated an important Linux administration concept:

File permissions are not the only factor controlling access.

To access:

/home/ec2-user/permission-lab/testfile.txt

the user must be able to traverse the directories in the path.

The x permission on a directory allows a user to traverse/search that directory.

Therefore, even though the file granted read access to the devops group, the restrictive /home/ec2-user directory prevented access.

13. Inspect ACL Information

I used getfacl to inspect the directory permissions:

getfacl /home/ec2-user

Example output:

# file: home/ec2-user
# owner: ec2-user
# group: ec2-user
user::rwx
group::rwx
other::rwx

getfacl can be used to inspect both traditional permissions and extended ACL entries.

14. Lab Troubleshooting Test

For demonstration purposes, the directory permission was temporarily changed:

chmod 777 /home/ec2-user

After this change, devuser was able to access the test file:

cat /home/ec2-user/permission-lab/testfile.txt

Output:

Linux permissions practice

This confirmed that the original problem was related to directory traversal permissions.

⚠️ Security Warning

777 grants read, write, and execute permissions to everyone.

It was used here only as a controlled troubleshooting demonstration.

It should generally not be used on a user's home directory in a production environment.

A production system should use the minimum permissions required, potentially with more precise ACLs.

🔐 Permission Reference
Mode	Meaning
600	Owner read/write
640	Owner read/write, group read
644	Owner read/write, group read, others read
700	Owner full access
755	Owner full access, group/others read + execute
777	Everyone full access — generally unsafe
🧠 Key Commands Practiced
User Management
useradd
passwd
usermod
id
getent passwd
Group Management
groupadd
getent group
usermod -aG
File Permissions
chmod
ls -l
Ownership
chgrp
ACL
getfacl
setfacl
User Switching
su -
🔍 Key Learnings
1. Linux uses three main permission categories
Owner
Group
Others
2. File permissions and directory permissions work together

A user may have permission to read a file but still receive:

Permission denied

if they cannot traverse the parent directory.

3. Groups provide scalable access control

Instead of granting permissions to individual users, users can be placed into groups such as:

devops
developers
sysadmins

and access can be managed at the group level.

4. chmod 777 should be avoided in production

Broad permissions may expose sensitive files and directories to unauthorized users.

5. ACLs provide more granular access control

getfacl and setfacl can be used when traditional owner/group/other permissions are not sufficient.
