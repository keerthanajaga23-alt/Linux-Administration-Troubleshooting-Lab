# Lab 04: Linux Service Management

## 1. Objective

The objective of this lab is to understand Linux services and practice checking service status, starting and stopping services, enabling services at boot, validating configurations, checking listening ports, and investigating service failures using logs.

## 2. Lab Environment

* **Operating system:** Ubuntu 24.04 LTS
* **Platform:** Windows Subsystem for Linux 2 (WSL2)
* **Service manager:** systemd
* **Services covered:** Nginx, SSH, and BIND9
* **Tools:** `systemctl`, `journalctl`, `ss`, `nginx`, `sshd`, `named-checkconf`

> Service availability can vary depending on the WSL configuration and installed packages. Run the checks below to confirm the actual state of each service.

## 3. Understanding Linux Services

A service is a background component that provides a function to the system or other applications.

Examples:

* **Nginx:** Provides web-server functionality.
* **SSH:** Allows secure remote access.
* **BIND9:** Provides DNS-server functionality.

### Important terms

* **systemd:** A system and service manager used by many Linux distributions.
* **Service unit:** A configuration and management definition for a service.
* **Daemon:** A program that runs in the background.
* **Service status:** Information about whether a service is running, stopped, or has failed.
* **Listening port:** A network port on which a program accepts connections.

## 4. Check Service Status

Check Nginx:

```bash
sudo systemctl status nginx --no-pager
```

Check SSH:

```bash
sudo systemctl status ssh --no-pager
```

Check BIND9:

```bash
sudo systemctl status named --no-pager
```

**Explanation:**

* `systemctl` communicates with the service manager.
* `status` displays the service state and recent status information.
* `--no-pager` displays the output directly in the terminal.

Possible states include:

* `active (running)`: The service is running.
* `inactive (dead)`: The service is not running.
* `failed`: The service encountered a failure.

A service can have other valid states, so read the complete output when troubleshooting.

## 5. Check Whether a Service Is Active

Use:

```bash
sudo systemctl is-active nginx
```

For SSH:

```bash
sudo systemctl is-active ssh
```

For BIND9:

```bash
sudo systemctl is-active named
```

This provides a quick check of whether a service is active.

## 6. Start a Service

Start Nginx:

```bash
sudo systemctl start nginx
```

Check the result:

```bash
sudo systemctl is-active nginx
```

Starting a service attempts to run it immediately. It does not automatically mean that the service will start after a reboot.

Only start a service if it is installed and you intend to use it.

## 7. Stop a Service

Stop Nginx:

```bash
sudo systemctl stop nginx
```

Check its state:

```bash
sudo systemctl status nginx --no-pager
```

**Important:** Stopping a service interrupts the functionality it provides. Do not stop SSH on a remote system without an alternative access method, and do not stop services that other applications depend on without understanding the impact.

## 8. Restart a Service

Restart Nginx:

```bash
sudo systemctl restart nginx
```

Then check the status:

```bash
sudo systemctl status nginx --no-pager
```

A restart stops and starts the service again. It may interrupt existing connections.

Use a restart when it is appropriate for the service and the change requires one.

## 9. Reload a Service

Some services support reloading their configuration without fully restarting.

For Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

First, validate the configuration. Reload only if the validation succeeds.

A reload is not supported by every service. If a reload fails, check the service documentation, unit definition, configuration, and logs rather than repeatedly retrying the command.


## 10. Enable a Service at Boot

Check whether Nginx is enabled:

```bash
sudo systemctl is-enabled nginx
```

Enable it to start automatically during system startup:

```bash
sudo systemctl enable nginx
```

Verify:

```bash
sudo systemctl is-enabled nginx
```

### Start versus enable

* `systemctl start`: Attempts to start the service now.
* `systemctl enable`: Configures the service to start automatically during boot.
* `systemctl enable --now`: Enables the service and attempts to start it immediately.

Whether a service starts automatically under WSL also depends on how WSL and systemd are configured.

## 11. Check Listening Ports

A service can be running without listening on the network port you expect.

Check Nginx's HTTP port:

```bash
sudo ss -lntp | grep ':80'
```

Check SSH's port:

```bash
sudo ss -lntp | grep ':22'
```

Check the DNS port:

```bash
sudo ss -lntup | grep ':53'
```

### Command explanation

* `ss`: Displays socket information.
* `-l`: Shows listening sockets.
* `-n`: Displays numeric addresses and ports.
* `-t`: Includes TCP sockets.
* `-u`: Includes UDP sockets.
* `-p`: Displays process information when available and permitted.

Common default ports:

| Service      | Typical port       |
| ------------ | ------------------ |
| HTTP / Nginx | TCP 80             |
| SSH          | TCP 22             |
| DNS / BIND9  | UDP and TCP 53     |
| RNDC control | TCP 953 by default |

A missing result does not automatically prove that a service is down. The service may use a different port, listen only on a particular address, or be configured differently.

## 12. Validate Service Configuration

Before restarting or reloading a service, validate its configuration when a validation tool is available.

### Nginx

```bash
sudo nginx -t
```

This checks the Nginx configuration syntax and related configuration-file errors.

### SSH

```bash
sudo sshd -t
```

No output normally indicates that the SSH daemon configuration syntax is valid.

### BIND9

```bash
sudo named-checkconf
```

This checks BIND configuration syntax.

To also attempt to load configured zones:

```bash
sudo named-checkconf -z
```

For the `lab.test` zone:

```bash
sudo named-checkzone lab.test /etc/bind/db.lab.test
```

A successful zone check should report `OK`.

Configuration validation does not prove that every aspect of the service is functioning. Follow it with service-status, port, log, and application-level tests as appropriate.

## 13. Investigate Service Logs

### Nginx logs

```bash
sudo journalctl -u nginx -n 50 --no-pager
```

### SSH logs

```bash
sudo journalctl -u ssh -n 50 --no-pager
```

### BIND9 logs

```bash
sudo journalctl -u named -n 50 --no-pager
```

### System error messages

```bash
sudo journalctl -p err -n 30 --no-pager
```

### Command explanation

* `journalctl` reads the system journal.
* `-u` filters messages by service unit.
* `-n 50` displays the last 50 entries.
* `-p err` filters for error-priority messages.
* `--no-pager` prints the output directly.

Investigate timestamps and messages related to the incident. A warning or old error may not be the reason the service is currently failing.

## 14 Troubleshooting: BIND9 Loopback DNS Queries

**Problem:** DNS queries to `127.0.0.1` failed with connection-refused or timeout errors, although queries to the WSL IPv4 address succeeded.

**Investigation:**
- Confirmed that the BIND9 service was active.
- Inspected DNS listeners with `ss`.
- Reviewed BIND9 logs with `journalctl`.
- Checked the route to `127.0.0.1` using `ip route get`.
- Found that loopback traffic was routed through `loopback0` instead of the expected local loopback route.

**Workaround:** Added temporary routing-policy rules to prioritise local routing for loopback destinations and flushed the route cache.

**Verification:** Record the actual results of the UDP and TCP DNS tests after applying the workaround.

**Limitation:** The routing-policy rules may not persist after WSL restarts. This workaround should not be described as a permanent fix without further investigation.

**Separate issue:** The RNDC control-channel timeout on port 953 remains unresolved.

## 15. Troubleshooting Scenario: Service Is Not Running

### Scenario

A user reports that a website is unavailable.

### Investigation

Check the Nginx service:

```bash
sudo systemctl status nginx --no-pager
```

If Nginx is inactive or failed, review the logs:

```bash
sudo journalctl -u nginx -n 50 --no-pager
```

Validate its configuration:

```bash
sudo nginx -t
```

Check whether the expected port is listening:

```bash
sudo ss -lntp | grep ':80'
```

Test the local HTTP response:

```bash
curl -I http://localhost
```

### Troubleshooting workflow

1. Understand the reported symptom.
2. Check the service status.
3. Review the service logs.
4. Validate the configuration.
5. Check the listening port and relevant dependencies.
6. Correct the identified problem.
7. Start, restart, or reload the service as appropriate.
8. Test the service again and record the verified result.

Do not automatically restart a service before collecting useful diagnostic information.

## 16. Troubleshooting Scenario: Port Is Not Listening

### Scenario

The service appears active, but the expected port is missing.

### Investigation

Check the service:

```bash
sudo systemctl status nginx --no-pager
```

Check its listening sockets:

```bash
sudo ss -lntp | grep ':80'
```

Validate the configuration:

```bash
sudo nginx -t
```

Review the service logs:

```bash
sudo journalctl -u nginx -n 50 --no-pager
```

Investigate whether the service is configured to listen on the expected address and port. Also check for port conflicts, which may prevent a service from binding to its configured port.

## 17. Troubleshooting Scenario: Service Fails to Reload

### Scenario

A service is running, but its reload command fails.

### Investigation

1. Record the exact reload error.
2. Validate the service configuration.
3. Inspect the service unit and its reload command.
4. Check service logs at the time of failure.
5. Confirm whether the service has a functioning control interface or reload mechanism.
6. Verify whether the service continued running and serving requests.

For BIND9, `systemctl reload named` failed in this WSL lab because the configured `rndc reload` command timed out when connecting to the control channel on `127.0.0.1:953`. The DNS zone passed validation, and a direct query to `172.16.36.24` successfully returned an A-record result for `server.lab.test`.

This control-channel problem remains a separate unresolved item. Do not describe the reload problem as fixed.

## 18 Key commands Reference

| Command                        | Purpose                             |
| ------------------------------ | ----------------------------------- |
| `systemctl status SERVICE`     | Inspect service status              |
| `systemctl is-active SERVICE`  | Check whether a service is active   |
| `systemctl is-enabled SERVICE` | Check whether it is enabled at boot |
| `systemctl start SERVICE`      | Start a service                     |
| `systemctl stop SERVICE`       | Stop a service                      |
| `systemctl restart SERVICE`    | Restart a service                   |
| `systemctl reload SERVICE`     | Request a configuration reload      |
| `systemctl enable SERVICE`     | Enable automatic startup            |
| `ss -lntp`                     | Inspect listening TCP ports         |
| `ss -lntup`                    | Inspect listening TCP and UDP ports |
| `journalctl -u SERVICE`        | Inspect service logs                |
| `journalctl -p err`            | Inspect error-priority messages     |
| `nginx -t`                     | Validate Nginx configuration        |
| `sshd -t`                      | Validate SSH daemon configuration   |
| `named-checkconf`              | Validate BIND configuration         |
| `named-checkzone`              | Validate a BIND zone file           |

Replace `SERVICE` with the actual service name, such as `nginx`, `ssh`, or `named`.

## 18. Learning Outcomes

This lab practices:

* Linux service management with systemd.
* Service status and startup configuration.
* Configuration validation before changes.
* Listening-port investigation.
* Service log analysis.
* Structured troubleshooting of failed services and reloads.

These skills are relevant to Linux administration, IT support, and infrastructure troubleshooting.

