# Lab 13: SELinux Troubleshooting on RHEL 9

## Overview

This lab demonstrates how to identify and troubleshoot an SELinux file-labeling issue affecting the Apache HTTP Server on Red Hat Enterprise Linux 9.

The objective is to understand how SELinux security contexts affect service access and how to restore the correct context without disabling SELinux.

## Environment

- OS: Red Hat Enterprise Linux 9
- Platform: AWS EC2
- Web server: Apache HTTP Server (httpd)
- Security framework: SELinux
- Tools: `getenforce`, `sestatus`, `ls -Z`, `chcon`, `matchpathcon`, `restorecon`, `ausearch`, and `curl`

## Objectives

- Verify SELinux enforcement mode.
- Inspect file security contexts.
- Test HTTP access to a webpage.
- Simulate an incorrect SELinux file label.
- Diagnose the access issue.
- Restore the correct security context.
- Verify service recovery and review audit logs.

## Procedure

### 1. Verify SELinux

```bash
getenforce
sestatus
```

Confirm that SELinux is enforcing its security policy before beginning the test.

### 2. Create a test webpage

```bash
echo "RHEL Lab 13 - SELinux Troubleshooting" \
| sudo tee /var/www/html/lab13.html

sudo chmod 644 /var/www/html/lab13.html
ls -lZ /var/www/html/lab13.html
```

### 3. Verify normal access

```bash
curl -i http://localhost/lab13.html
```

Expected result: HTTP 200 OK.

### 4. Simulate an incorrect file context

```bash
sudo chcon -t user_home_t /var/www/html/lab13.html
ls -lZ /var/www/html/lab13.html
```

Test the webpage again:

```bash
curl -i http://localhost/lab13.html
```

Expected result under the standard enforcing policy: HTTP 403 Forbidden.

### 5. Diagnose the context

```bash
matchpathcon /var/www/html/lab13.html
matchpathcon -V /var/www/html/lab13.html
```

The commands help identify whether the current file context differs from the expected context.

### 6. Restore the correct context

```bash
sudo restorecon -v /var/www/html/lab13.html
```

Verify the context and web access:

```bash
ls -lZ /var/www/html/lab13.html
matchpathcon -V /var/www/html/lab13.html
curl -i http://localhost/lab13.html
```

Expected result: HTTP 200 OK, with the appropriate Apache-readable SELinux context.

### 7. Review audit logs

```bash
sudo ausearch -m AVC,USER_AVC,SELINUX_ERR,USER_SELINUX_ERR -ts recent
```

Review any relevant SELinux denial records, if available.

## Expected Learning Outcomes

- SELinux can restrict access even when traditional file permissions appear correct.
- Incorrect file labels can cause Apache access failures.
- `matchpathcon` helps compare actual and expected file contexts.
- `restorecon` restores labels according to the configured SELinux file-context policy.
- Troubleshooting should preserve SELinux enforcement whenever possible.

## Troubleshooting Notes

If the HTTP request does not return the expected response, verify that Apache is running, that the file is readable, and that SELinux is enforcing. Investigate logs before attributing the problem to SELinux.

Audit events may not be available in every environment.

## Conclusion

This lab demonstrates a practical Linux administration troubleshooting workflow: reproduce a controlled failure, inspect the evidence, correct the underlying configuration, and verify service recovery without disabling SELinux.


