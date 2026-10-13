# Lab 08: Nginx Web Server Troubleshooting

## 1. Objective

The objective of this lab is to practise checking Nginx service status, validating web-server configuration, inspecting listening ports, testing HTTP responses, analysing logs, and troubleshooting common website availability problems.

## 2. Lab Environment

* **Operating system:** Ubuntu 24.04 LTS
* **Platform:** Windows Subsystem for Linux 2 (WSL2)
* **Web server:** Nginx
* **Tools:** `systemctl`, `nginx`, `ss`, `curl`, `journalctl`, `grep`

Run the checks in Ubuntu and record the actual results. The commands below are exercises; do not mark an item as completed until you have verified it.

## 3. What Is Nginx?

Nginx is web-server software that can serve web pages and act as a reverse proxy for backend applications.

A simple website request follows this path:

```text
Client
   |
   | HTTP request
   v
Nginx web server
   |
   v
HTTP response
```

For a reverse-proxy application, the flow may be:

```text
Client
   |
   v
Nginx
   |
   v
Backend application
```

If a website is unavailable, the problem could be with Nginx, its configuration, its listening port, a network control, or a backend application.

## 4. Check Whether Nginx Is Installed

Check whether the Nginx executable is available:

```bash
which nginx
```

Check the installed package:

```bash
dpkg -l nginx
```

If Nginx is not installed and you want to perform this lab, install it only after confirming that installation is appropriate for your environment.

```bash
sudo apt update
sudo apt install nginx
```

Do not reinstall Nginx if it is already installed and working.

## 5. Check Nginx Service Status

Run:

```bash
sudo systemctl status nginx --no-pager
```

For a quick status check:

```bash
sudo systemctl is-active nginx
```

Possible results include:

* `active`: The service is active.
* `inactive`: The service is not currently active.
* `failed`: The service encountered a failure.

An active service does not guarantee that the website is reachable or that the correct content is being served. Continue with port and HTTP checks.

## 6. Start Nginx If Required

If Nginx is installed but stopped, and you intend to use it, start the service:

```bash
sudo systemctl start nginx
```

Check its status again:

```bash
sudo systemctl is-active nginx
```

To configure Nginx to start automatically when the Linux system boots:

```bash
sudo systemctl enable nginx
```

Verify:

```bash
sudo systemctl is-enabled nginx
```

**Important distinction:**

* `start` attempts to run the service now.
* `enable` configures automatic startup during boot.

Under WSL2, automatic startup also depends on how WSL and systemd are configured.

## 7. Validate Nginx Configuration

Before reloading or restarting Nginx after a configuration change, validate the configuration:

```bash
sudo nginx -t
```

A successful test normally reports that the syntax is OK and the configuration test is successful.

If validation fails, inspect the reported file and line number. Correct the actual configuration problem and run `nginx -t` again.

Do not reload or restart Nginx with a known invalid configuration.

## 8. Check Whether Nginx Is Listening on Port 80

Run:

```bash
sudo ss -lntp | grep ':80'
```

This checks for matching listening TCP sockets on port 80.

The `ss` command displays socket information, while these options mean:

* `-l`: Show listening sockets.
* `-n`: Display numeric addresses and ports.
* `-t`: Show TCP sockets.
* `-p`: Show process information when available and permitted.

HTTP commonly uses TCP port 80. HTTPS commonly uses TCP port 443.

If the command returns no output, investigate whether Nginx is running, which address and port it is configured to use, and whether another issue is preventing it from listening.

## 9. Test the Local Website Using curl

Run:

```bash
curl -I http://localhost
```

This sends an HTTP request for response headers to the local web server.

To see more request and response details, run:

```bash
curl -v http://localhost
```

Interpret the result:

* An HTTP response indicates that an HTTP server responded.
* `Connection refused` commonly means no service is accepting the connection at the specified address and port, or the connection was actively rejected.
* A timeout means no timely response was received.
* An HTTP error status means an HTTP server responded but reported an error condition.

An HTTP response alone does not guarantee that the application is working correctly. Check the response status and content.

## 10. Inspect Nginx Logs

Inspect recent Nginx service logs:

```bash
sudo journalctl -u nginx -n 50 --no-pager
```

Nginx commonly also writes to these log files:

```text
/var/log/nginx/access.log
/var/log/nginx/error.log
```

Inspect recent error-log entries:

```bash
sudo tail -n 50 /var/log/nginx/error.log
```

Search for messages containing the word `error`:

```bash
sudo grep -i error /var/log/nginx/error.log
```

The access log records HTTP requests, while the error log records Nginx operational issues and other diagnostic messages.

Log contents vary with the configuration and the events that have occurred. Record the actual error messages rather than assuming a particular cause.

## 11. Troubleshooting Scenario: Website Is Not Opening

### Scenario

A user reports that a website hosted on the Linux server is unavailable.

### Step 1: Check the service

```bash
sudo systemctl status nginx --no-pager
```

### Step 2: Check the listening port

```bash
sudo ss -lntp | grep ':80'
```

### Step 3: Validate the configuration

```bash
sudo nginx -t
```

### Step 4: Test HTTP locally

```bash
curl -I http://localhost
```

### Step 5: Review logs

```bash
sudo journalctl -u nginx -n 50 --no-pager
```

If required, inspect the error log:

```bash
sudo tail -n 50 /var/log/nginx/error.log
```

### Troubleshooting workflow

1. Identify the reported symptom and whether it affects one user or multiple users.
2. Check Nginx service status.
3. Check the listening port and configured address.
4. Test the local HTTP endpoint.
5. Inspect the configuration and logs.
6. Investigate network connectivity, DNS, firewall rules, or other dependencies if the evidence points to them.
7. Correct the identified problem.
8. Reload or restart the service only when appropriate.
9. Repeat the HTTP test and confirm whether the issue is resolved.

Do not assume that Nginx is the cause simply because a website is unavailable.

## 12. Troubleshooting Scenario: Connection Refused

### Scenario

A request to the local website returns a connection-refused error.

### Investigation

Check:

```bash
sudo systemctl status nginx --no-pager
```

Then:

```bash
sudo ss -lntp | grep ':80'
```

Validate the configuration:

```bash
sudo nginx -t
```

Investigate whether Nginx is active, whether it is listening on the requested address and port, and whether another relevant configuration or network issue is involved.

A connection-refused error alone does not establish the exact root cause.

## 13. Troubleshooting Scenario: HTTP 502 Bad Gateway

### Scenario

The browser displays:

```text
502 Bad Gateway
```

This can occur when Nginx acts as a reverse proxy and cannot obtain a valid response from its backend.

The request path may look like:

```text
Client
   |
   v
Nginx
   |
   | Backend request fails
   v
Backend application
```

### Investigation

Inspect the Nginx error log:

```bash
sudo tail -n 50 /var/log/nginx/error.log
```

Inspect the configuration:

```bash
sudo nginx -T
```

**Note:** `nginx -T` prints the effective configuration, which may contain internal details. Review the output locally and do not upload sensitive configuration values to a public repository.

If a backend is configured, inspect whether the application is running and listening on its expected port.

For example, if the configured backend is `127.0.0.1:8080`, check:

```bash
sudo ss -lntp | grep ':8080'
```

Compare the backend address and port configured in `proxy_pass` with the actual listener. Review the backend service status and logs when available.

Do not assume every 502 error is a port mismatch; use the logs to establish the likely cause.

## 14. Troubleshooting Scenario: Upstream Timeout

An Nginx error log may contain a message such as:

```text
upstream timed out
```

This indicates that Nginx did not receive the expected response from an upstream operation within the configured time.

Possible causes include:

* A slow or overloaded backend application.
* An unavailable backend.
* Network connectivity problems between Nginx and the backend.
* An incorrect upstream address or port.
* Backend dependencies that are slow or unavailable.

### Investigation

Check the Nginx logs:

```bash
sudo tail -n 50 /var/log/nginx/error.log
```

Check the configured backend address and port:

```bash
sudo nginx -T
```

Check the relevant listening port, using the actual port from your configuration:

```bash
sudo ss -lntp
```

If the backend service is known, inspect its status and logs as well.

Identify the actual cause before changing timeout settings. Increasing a timeout may conceal the symptom without addressing the underlying problem.

## 15. Troubleshooting Scenario: Port 80 Is Already in Use

If Nginx cannot bind to port 80, another process may be using that port or Nginx may have a conflicting listener configuration.

Inspect listening TCP sockets:

```bash
sudo ss -lntp | grep ':80'
```

Check Nginx configuration syntax:

```bash
sudo nginx -t
```

Investigate the process and its purpose before making changes. Do not stop another service until you understand whether it is required.

## 16. Reload or Restart Nginx After a Fix

After correcting a configuration change, validate it:

```bash
sudo nginx -t
```

If validation succeeds, request a reload:

```bash
sudo systemctl reload nginx
```

Check the status:

```bash
sudo systemctl status nginx --no-pager
```

Test HTTP again:

```bash
curl -I http://localhost
```

A reload is generally intended to apply supported configuration changes without fully stopping the service. A restart stops and starts the service and may interrupt existing connections. Choose the appropriate action for the situation.

If the reload fails, inspect the command output, service state, and logs instead of repeatedly retrying it.

## 17. Key Commands Reference

| Command                                  | Purpose                                            |
| ---------------------------------------- | -------------------------------------------------- |
| `which nginx`                            | Locate the Nginx executable                        |
| `systemctl status nginx`                 | Inspect service status                             |
| `systemctl is-active nginx`              | Check whether Nginx is active                      |
| `systemctl start nginx`                  | Start Nginx                                        |
| `systemctl enable nginx`                 | Enable automatic startup                           |
| `nginx -t`                               | Validate Nginx configuration                       |
| `ss -lntp`                               | Inspect listening TCP sockets                      |
| `curl -I http://localhost`               | Test an HTTP response                              |
| `curl -v http://localhost`               | Inspect detailed HTTP request/response information |
| `journalctl -u nginx`                    | Inspect Nginx service logs                         |
| `tail -n 50 /var/log/nginx/error.log`    | View recent error-log entries                      |
| `grep -i error /var/log/nginx/error.log` | Search the error log                               |
| `nginx -T`                               | Print the effective configuration                  |

## 18. Learning Outcomes

This lab provides practice in:

* Nginx service administration.
* Configuration validation.
* Listening-port investigation.
* HTTP testing with `curl`.
* Log analysis.
* Troubleshooting website connection failures.
* Investigating reverse-proxy and upstream problems.
* Verifying service recovery after a correction.

