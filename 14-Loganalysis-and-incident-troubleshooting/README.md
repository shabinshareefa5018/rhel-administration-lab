# Lab 14: Linux Log Analysis and Incident Troubleshooting

## Overview

This lab demonstrates a practical incident investigation workflow on Red Hat Enterprise Linux 9. A controlled systemd service failure is generated, investigated through logs, corrected, and verified.

The lab focuses on evidence-based troubleshooting without interrupting production services or intentionally breaking SSH, Apache, or other existing system services.

## Environment

- Operating System: RHEL 9
- Platform: AWS EC2
- Service manager: systemd
- Logging tools: journalctl, systemctl
- Log locations: systemd journal and available files under /var/log

## Objectives

- Investigate system and service logs.
- Filter journal entries by service, boot, and priority.
- Reproduce a controlled service failure.
- Identify the failure reason from log evidence.
- Correct the fault and verify service recovery.
- Document findings as an incident report.

## Part 1: Baseline system investigation

### Step 1: Check system health

```bash
date
uptime
free -h
df -h /
systemctl --failed
```

Record any existing failed units before starting the experiment.

### Step 2: Inspect system logs

```bash
sudo journalctl -b -n 50 --no-pager
sudo journalctl -p err -b -n 30 --no-pager
sudo journalctl --list-boots
```

Understand the difference between all messages, error-level messages, and logs grouped by boot.

### Step 3: Inspect an existing service

Use SSH as an example:

```bash
sudo systemctl status sshd --no-pager
sudo journalctl -u sshd -n 30 --no-pager
```

Do not restart or modify SSH during this lab.

If your system uses a different SSH service name, identify the correct unit first.

## Part 2: Create a controlled service failure

### Step 4: Create a test script

Create a dedicated script that deliberately exits with a failure status.

```bash
sudo tee /usr/local/bin/lab14-demo.sh > /dev/null <<'EOF'
#!/bin/bash
echo "Lab 14: Simulated application startup failure"
exit 1
EOF

sudo chmod 755 /usr/local/bin/lab14-demo.sh
```

This script is intentionally designed to fail. It does not change system configuration or stop existing services.

### Step 5: Create a dedicated systemd unit

```bash
sudo tee /etc/systemd/system/lab14-demo.service > /dev/null <<'EOF'
[Unit]
Description=Lab 14 Controlled Troubleshooting Service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/lab14-demo.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOF
```

Load the unit and validate it:

```bash
sudo systemctl daemon-reload
sudo systemd-analyze verify /etc/systemd/system/lab14-demo.service
```

### Step 6: Trigger the failure

```bash
sudo systemctl start lab14-demo.service
```

The command is expected to report failure because the script exits with status 1.

Inspect the unit:

```bash
sudo systemctl status lab14-demo.service --no-pager -l
```

## Part 3: Investigate the incident

### Step 7: Find the service logs

```bash
sudo journalctl -u lab14-demo.service -n 30 --no-pager
```

Review the timestamp, process exit status, and error message.

### Step 8: Filter by priority and time

```bash
sudo journalctl -p err -b --no-pager -n 50
sudo journalctl -u lab14-demo.service --since "10 minutes ago" --no-pager
```

Compare filtered results with the complete service log.

### Step 9: Identify the root cause

Use the evidence to answer:

1. Which service failed?
2. What command did systemd execute?
3. What exit status was returned?
4. Which log entry confirms the failure?
5. Is the cause a systemd configuration error or an application-level failure?

Expected finding: the test script intentionally returns exit status 1. The failure is caused by the script's behavior, not by SELinux, SSH, or Apache.

## Part 4: Correct and verify recovery

### Step 10: Repair the script

Replace the test script with a successful version:

```bash
sudo tee /usr/local/bin/lab14-demo.sh > /dev/null <<'EOF'
#!/bin/bash
echo "Lab 14: Application started successfully"
exit 0
EOF

sudo chmod 755 /usr/local/bin/lab14-demo.sh
```

### Step 11: Reset the failed state and retry

```bash
sudo systemctl reset-failed lab14-demo.service
sudo systemctl start lab14-demo.service
sudo systemctl status lab14-demo.service --no-pager
```

Because the unit is a oneshot service with `RemainAfterExit=yes`, a successful run should report `active (exited)`.

### Step 12: Verify the recovery in logs

```bash
sudo journalctl -u lab14-demo.service --since "10 minutes ago" --no-pager
```

Confirm that the later execution completed successfully.

Check the state of the rest of the system:

```bash
systemctl --failed
sudo systemctl is-active sshd
sudo systemctl is-active httpd
```

Compare these results with your baseline. Existing unrelated failures, if any, should be recorded rather than assumed to be caused by this lab.

## Part 5: Incident report

Document your findings using this template.

| Field | Finding |
|---|---|
| Incident | Controlled application service failure |
| Affected unit | lab14-demo.service |
| Detection method | systemctl status and journalctl |
| Root cause | Test script returned exit status 1 |
| Remediation | Updated script to exit successfully |
| Verification | Service completed successfully; logs confirmed recovery |

Replace these example findings with your actual command outputs and observations.

## Troubleshooting notes

- If the unit cannot be loaded, inspect the unit file and run `systemd-analyze verify`.
- If no logs appear, confirm that the unit name is correct and that the service was started.
- If `journalctl -p err` shows unrelated errors, investigate their timestamps and unit names before attributing them to this exercise.
- Do not clear the system journal or modify SSH, firewall, or SELinux configuration to force the test to work.

## Cleanup

After collecting your evidence, remove the temporary lab service.

```bash
sudo systemctl stop lab14-demo.service
sudo systemctl disable lab14-demo.service
sudo rm -f /etc/systemd/system/lab14-demo.service
sudo rm -f /usr/local/bin/lab14-demo.sh
sudo systemctl daemon-reload
sudo systemctl reset-failed lab14-demo.service
```

Verify that the temporary unit is no longer loaded:

```bash
systemctl status lab14-demo.service --no-pager
```

A message indicating that the unit could not be found is expected after cleanup. Journal history may remain available according to the system's journal retention settings.

## Acceptance criteria

- [ ] Baseline system health recorded
- [ ] System and SSH logs inspected
- [ ] Controlled failure reproduced
- [ ] Root cause identified from journal evidence
- [ ] Script corrected
- [ ] Successful execution verified
- [ ] Incident report completed with actual findings
- [ ] Temporary service and script removed

## Conclusion

This lab demonstrates a repeatable incident response workflow: establish a baseline, reproduce a controlled failure, gather evidence, identify the root cause, apply a correction, verify recovery, and document the result.

These skills are relevant to Linux administration, cloud support, and DevOps operations.
