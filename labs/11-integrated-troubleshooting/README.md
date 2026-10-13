# Lab 11: Integrated Linux Troubleshooting

## 1. Objective

The objective of this lab is to apply Linux administration and troubleshooting skills to realistic technical support scenarios.

This lab combines service management, network troubleshooting, DNS, file permissions, process monitoring, resource investigation, and log analysis.

For each scenario, the goal is to identify the symptom, collect evidence, determine the likely cause, apply an appropriate correction where safe, and verify the result.

## 2. Lab Environment

* **Operating system:** Ubuntu 24.04 LTS
* **Platform:** Windows Subsystem for Linux 2 (WSL2)
* **Services covered:** Nginx, SSH, and BIND9 where available
* **Tools:** `systemctl`, `journalctl`, `ip`, `ping`, `ss`, `dig`, `nslookup`, `curl`, `ps`, `top`, `free`, `df`, `du`, `ls`, `id`, `chmod`, and `chown`

The results depend on the actual state of the WSL environment. Record the commands you ran and the output you observed.

## 3. Troubleshooting Method

Use this workflow for each scenario:

1. **Identify the symptom.** Confirm what is failing and what the expected behaviour is.
2. **Collect evidence.** Run appropriate diagnostic commands before making changes.
3. **Narrow down the cause.** Compare the results against the expected configuration and behaviour.
4. **Apply a suitable correction.** Make only the changes justified by the evidence.
5. **Validate the configuration.** Use the relevant validation tool when available.
6. **Test again.** Repeat the original test to establish whether the problem is resolved.
7. **Document the findings.** Record the symptom, evidence, likely cause, correction, and final result.

Do not restart services, terminate processes, or change permissions blindly. Understand the possible impact before making changes.

## 4. Scenario 1: Website Is Not Opening

### Problem statement

A user reports that the website hosted on the Linux machine is unavailable.

Nginx is expected to serve the website over HTTP.

### Investigation

Check the Nginx service:

```bash
sudo systemctl status nginx --no-pager
```

Check whether a process is listening on HTTP port 80:

```bash
sudo ss -lntp | grep ':80'
```

Validate the Nginx configuration:

```bash
sudo nginx -t
```

Test the local HTTP endpoint:

```bash
curl -I http://localhost
```

Inspect recent service logs:

```bash
sudo journalctl -u nginx -n 50 --no-pager
```

### Record your findings

* **Symptom:**
* **Commands executed:**
* **Evidence collected:**
* **Likely cause:**
* **Correction, if required:**
* **Verification command and result:**

**Status:** Pending practical investigation.

## 5. Scenario 2: IP Connectivity Works but DNS Fails

### Problem statement

A user can reach an IP address, but a hostname cannot be resolved.

### Investigation

Test IPv4 reachability:

```bash
ping -4 -c 4 8.8.8.8
```

Inspect the resolver configuration:

```bash
cat /etc/resolv.conf
```

Test system name resolution:

```bash
getent ahostsv4 google.com
```

Test a DNS query:

```bash
nslookup google.com
```

Inspect the DNS response directly:

```bash
dig google.com A
```

### Record your findings

* **Did the IP connectivity test succeed?**
* **Which DNS resolver is configured?**
* **Did the hostname lookup succeed?**
* **What did `dig` report?**
* **What is the likely cause?**
* **What additional evidence is required?**
* **How would you verify a correction?**

Do not conclude that DNS is the cause from a single failed ping. Compare the results of the different tests.

**Status:** Pending practical investigation.

## 6. Scenario 3: User Receives Permission Denied

### Problem statement

A Linux user cannot access a file in the shared directory `/opt/itlab`.

### Investigation

Check the user's identity and group memberships:

```bash
id alice
```

Check the directory permissions:

```bash
ls -ld /opt /opt/itlab
```

Check the file ownership and permissions:

```bash
ls -l /opt/itlab/
```

Inspect permissions along the path when required:

```bash
namei -l /opt/itlab/private.txt
```

### Record your findings

* **Which user is affected?**
* **Is the user in the required group?**
* **Who owns the file?**
* **What permissions are configured?**
* **Can the user traverse the parent directories?**
* **What is the likely cause?**
* **What correction would you apply?**
* **Can the user access the file after the correction?**

Do not use unrestricted permissions such as `777` as a shortcut. Apply only the access required by the task.

**Status:** Pending practical investigation.

## 7. Scenario 4: SSH Connection Failure

### Problem statement

A user cannot connect to a Linux server using SSH.

### Investigation

Check whether the SSH server is active:

```bash
sudo systemctl status ssh --no-pager
```

Check whether TCP port 22 is listening:

```bash
sudo ss -lntp | grep ':22'
```

Validate the SSH server configuration:

```bash
sudo sshd -t
```

Inspect recent service logs:

```bash
sudo journalctl -u ssh -n 50 --no-pager
```

If appropriate, test a local connection:

```bash
ssh localhost
```

### Record your findings

* **Is the SSH server installed?**
* **Is the service active?**
* **Is a listener present on the expected address and port?**
* **Does the configuration validation succeed?**
* **Does the log show a relevant error?**
* **Is the symptom a connection refusal, timeout, or authentication failure?**
* **What additional checks are needed?**
* **What is the final verification result?**

A local SSH test does not prove remote network access is working. Consider the actual source, destination, and network path involved.

**Status:** Pending practical investigation.

## 8. Scenario 5: High CPU or Memory Usage

### Problem statement

A user reports that the Linux system or an application is running slowly.

### Investigation

Display processes sorted by CPU usage:

```bash
ps aux --sort=-%cpu | head
```

Display processes sorted by memory usage:

```bash
ps aux --sort=-%mem | head
```

Monitor live system activity:

```bash
top
```

Check memory information:

```bash
free -h
```

Press `q` to exit `top`.

### Record your findings

* **Is CPU usage unusually high?**
* **Which processes are using the most CPU?**
* **Is memory availability low?**
* **Which processes are using the most memory?**
* **Are the processes expected to be running?**
* **What further evidence would help identify the cause?**
* **What action would be appropriate and safe?**
* **Did system behaviour improve after any correction?**

Do not terminate a process simply because it appears near the top of the list. Identify the process and investigate before taking action.

**Status:** Pending practical investigation.

## 9. Scenario 6: Disk Space Is Running Low

### Problem statement

An application reports that it cannot save files, or the Linux system reports low disk space.

### Investigation

Check filesystem usage:

```bash
df -h
```

Inspect space used by top-level directories:

```bash
sudo du -sh /var/*
```

Check the relevant directory in more detail if the output identifies an area using substantial space.

### Record your findings

* **Which filesystem is affected?**
* **What percentage of its space is used?**
* **Which directory is using substantial space?**
* **Which files or data need further investigation?**
* **What safe cleanup or corrective action is appropriate?**
* **Did available space improve after the correction?**

Do not delete system files, logs, or application data blindly. Identify the relevant files and consider their purpose before removing anything.

**Status:** Pending practical investigation.

## 10. Scenario 7: Service Fails to Start or Reload

### Problem statement

A Linux service reports a failure when it starts or reloads.

### Investigation

Check the service status:

```bash
sudo systemctl status SERVICE --no-pager
```

Replace `SERVICE` with the actual service name.

Check recent logs:

```bash
sudo journalctl -u SERVICE -n 50 --no-pager
```

Validate the configuration using the relevant service-specific command. For example:

```bash
sudo nginx -t
```

For BIND9:

```bash
sudo named-checkconf -z
```

For SSH:

```bash
sudo sshd -t
```

### Record your findings

* **Which service failed?**
* **Did the start or reload operation fail?**
* **What does the service status show?**
* **What does the log report at the time of failure?**
* **Does configuration validation succeed?**
* **What is the likely cause?**
* **What correction is justified by the evidence?**
* **Does the service work after the correction?**

Not every service supports the same reload mechanism. A successful configuration check does not by itself prove that the reload succeeded.

**Status:** Pending practical investigation.

## 11. Evidence and Results Table

Use this table to summarise each investigation.

| Scenario            | Initial symptom | Likely cause | Correction applied | Verification |
| ------------------- | --------------- | ------------ | ------------------ | ------------ |
| Website unavailable | Pending         | Pending      | Pending            | Pending      |
| DNS failure         | Pending         | Pending      | Pending            | Pending      |
| Permission denied   | Pending         | Pending      | Pending            | Pending      |
| SSH failure         | Pending         | Pending      | Pending            | Pending      |
| High CPU or memory  | Pending         | Pending      | Pending            | Pending      |
| Low disk space      | Pending         | Pending      | Pending            | Pending      |
| Service failure     | Pending         | Pending      | Pending            | Pending      |

Replace `Pending` with your actual findings as you work through the scenarios.

## 12. Final Practical Assessment

After working through the individual scenarios, complete an integrated assessment without following a supplied command sequence.

For each new problem:

1. State the symptom.
2. Explain your initial hypothesis.
3. Choose the diagnostic commands yourself.
4. Interpret the command output.
5. Identify the likely cause and supporting evidence.
6. Propose an appropriate correction.
7. Verify the result.
8. Explain when you would escalate the problem to a senior engineer.

The purpose is to demonstrate an evidence-based troubleshooting method rather than memorising commands.

## 13. Learning Outcomes

This lab provides practice in:

* Combining Linux administration and troubleshooting skills.
* Choosing commands based on the reported symptom.
* Interpreting command outputs and service logs.
* Separating symptoms from likely root causes.
* Applying safe and appropriate corrections.
* Confirming service recovery through a repeatable test.
* Documenting findings and recognising when to escalate a problem.

