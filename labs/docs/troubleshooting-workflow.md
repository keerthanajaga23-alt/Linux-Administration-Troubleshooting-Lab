# Linux Troubleshooting Workflow

## 1. Purpose

This document describes a structured approach to diagnosing Linux administration and IT support problems.

The goal is to identify the likely cause using evidence, apply an appropriate correction, and verify the result.

## 2. Troubleshooting Principles

* Understand the symptom before changing anything.
* Choose commands that provide useful evidence.
* Check logs and configuration when the symptom requires it.
* Avoid changes that could cause unnecessary service interruption.
* Verify the result after making a correction.
* Document unresolved issues honestly.

## 3. General Troubleshooting Workflow

### Step 1: Identify the Symptom

Ask:

* What is not working?
* What should happen?
* When did the issue begin?
* Does it affect one user or multiple users?
* Has anything changed recently?

### Step 2: Collect Evidence

Select appropriate commands based on the issue.

Examples:

```bash
systemctl status SERVICE --no-pager
ip addr
ip route
ss -lntp
journalctl -u SERVICE -n 50 --no-pager
```

Replace `SERVICE` with the relevant service name. These are examples, not commands that must all be run for every issue.

### Step 3: Identify the Likely Cause

Compare the results with the expected behaviour.

For example:

* A stopped service suggests a service availability issue.
* A missing listener may indicate that the application is not listening on the expected address or port.
* A DNS error suggests further investigation of name resolution.
* A permission error requires investigation of users, groups, ownership and permissions.
* An error log can provide evidence about a configuration or application problem.

Do not treat a single clue as conclusive proof without supporting evidence.

### Step 4: Apply a Suitable Correction

Change only what is necessary to address the identified issue.

Examples include correcting a configuration value, adjusting the relevant permission, or starting a service when it is supposed to be running.

Before making a change:

* Check its likely impact.
* Back up important configuration files when appropriate.
* Validate the configuration.
* Avoid weakening security just to make the error disappear.

### Step 5: Verify the Outcome

Repeat the original test.

Examples:

```bash
systemctl is-active SERVICE
curl -I http://localhost
dig @DNS-IP HOSTNAME A
```

Replace placeholders with values appropriate to the system.

A successful command should be interpreted in context. For example, an active service does not prove that its website or DNS response is correct.

### Step 6: Document the Result

Record:

* Reported symptom.
* Diagnostic commands and outputs.
* Likely root cause and supporting evidence.
* Correction applied.
* Verification command and result.
* Remaining issues or escalation needs.

## 4. Workflow: Website Is Not Opening

Investigate Nginx only if it is the expected web server.

Useful checks:

```bash
sudo systemctl status nginx --no-pager
sudo nginx -t
sudo ss -lntp | grep ':80'
curl -I http://localhost
sudo journalctl -u nginx -n 50 --no-pager
```

Consider:

* Is the service active?
* Is it listening on the intended address and port?
* Does the configuration validate?
* Does the local HTTP request return a response?
* Do the logs show an associated error?
* Does the problem occur only for remote clients?

If the local test succeeds but remote users cannot connect, investigate the network path, name resolution, and relevant security controls.

## 5. Workflow: DNS Resolution Failure

Useful checks:

```bash
cat /etc/resolv.conf
ip route
getent ahostsv4 google.com
nslookup google.com
dig google.com A
```

Consider:

* Which resolver is configured?
* Does system name resolution return an address?
* Does a direct query to the expected DNS server work?
* Is the server reachable?
* Is the query reporting a timeout, NXDOMAIN, or SERVFAIL?

A failed ping alone does not prove a DNS failure. Compare IP connectivity with actual DNS query results.

## 6. Workflow: Permission Denied

Useful checks:

```bash
id username
ls -l /path/to/file
ls -ld /path/to/directory
namei -l /path/to/file
```

Consider:

* Does the user exist?
* Is the user in the required group?
* Who owns the file?
* What are the file permissions?
* Can the user traverse the parent directories?

Correct the specific cause and test access as the affected user. Avoid `chmod 777` as a general fix.

## 7. Workflow: SSH Connection Failure

Useful checks on the server:

```bash
sudo systemctl status ssh --no-pager
sudo ss -lntp | grep ':22'
sudo sshd -t
sudo journalctl -u ssh -n 50 --no-pager
```

Consider whether the symptom is:

* Connection refused.
* Connection timeout.
* Authentication failure.

These symptoms suggest different investigation paths. A reachable SSH server can still reject a login due to account or authentication configuration.

## 8. Workflow: High CPU or Memory Usage

Useful checks:

```bash
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
top
free -h
```

Consider:

* Which processes are using resources?
* Is the usage sustained?
* Is the process expected to run?
* Are application logs or recent changes relevant?

Do not terminate an unknown process without understanding its purpose.

## 9. Workflow: Low Disk Space

Useful checks:

```bash
df -h
du -sh /var/*
```

Consider:

* Which filesystem is full?
* Which directories use substantial space?
* Are large files, logs, or application data responsible?
* Is the data safe to remove or does it need to be retained?

Do not delete files blindly. Identify the files and understand their purpose before taking action.

## 10. Workflow: Service Fails to Start or Reload

Useful checks:

```bash
sudo systemctl status SERVICE --no-pager
sudo journalctl -u SERVICE -n 50 --no-pager
```

Then run the appropriate configuration-validation command, where available.

Examples:

```bash
sudo nginx -t
sudo sshd -t
sudo named-checkconf -z
```

A running service can have a failing reload mechanism. A successful configuration validation does not necessarily prove that the reload succeeded.

## 11. Result Documentation Template

Use this format when documenting a troubleshooting exercise:

### Problem

Describe the symptom and expected behaviour.

### Environment

Record the relevant operating system, service, and configuration.

### Investigation

List the commands used and summarise the important results.

### Findings

Explain the likely cause and identify the evidence supporting the conclusion.

### Correction

Describe the action taken, or record that no change was made.

### Verification

Describe the test repeated after the investigation or correction and its actual result.

### Final Status

Choose one:

* Resolved and verified.
* Partially resolved.
* Unresolved; further investigation required.

## 12. Escalation

Escalate when:

* The cause cannot be identified using available evidence.
* A change could cause significant service interruption.
* Access to a dependency or administrative system is unavailable.
* The issue affects production systems or multiple users.
* The proposed fix requires permissions or knowledge outside your responsibility.

When escalating, provide the symptom, impact, commands run, relevant logs, findings, and actions already attempted.

## 13. Conclusion

A structured troubleshooting process helps reduce guesswork and unnecessary changes. The key is to understand the symptom, gather evidence, investigate the likely cause, apply an appropriate correction, and verify the outcome before declaring the issue resolved.

