# Linux Command Cheat Sheet

## 1. Purpose

This document is a quick reference for Linux commands used in the Linux Administration and Troubleshooting Lab.

The commands are grouped by task to make it easier to select the appropriate tool during troubleshooting.

## 2. Navigation and File Operations

| Command                    | Purpose                            |
| -------------------------- | ---------------------------------- |
| `pwd`                      | Display the current directory      |
| `ls -la`                   | List files, including hidden files |
| `cd /path`                 | Change directory                   |
| `cd ..`                    | Move to the parent directory       |
| `mkdir -p folder`          | Create a directory                 |
| `touch file.txt`           | Create an empty file               |
| `cat file.txt`             | Display file contents              |
| `cp source.txt backup.txt` | Copy a file                        |
| `mv old.txt new.txt`       | Move or rename a file              |
| `rm file.txt`              | Remove a file                      |

**Safety:** Check the path before using `rm`. Deleted files may not be recoverable.

## 3. User and Group Management

| Command                               | Purpose                             |
| ------------------------------------- | ----------------------------------- |
| `whoami`                              | Display the current username        |
| `id username`                         | Display user and group IDs          |
| `groups username`                     | Display the user's groups           |
| `getent passwd username`              | Check whether a user account exists |
| `getent group groupname`              | Check group information             |
| `sudo groupadd groupname`             | Create a group                      |
| `sudo useradd -m username`            | Create a user with a home directory |
| `sudo passwd username`                | Set or change a user's password     |
| `sudo usermod -aG groupname username` | Add a supplementary group           |
| `sudo -u username command`            | Run a command as another user       |

**Important:** The `-a` in `usermod -aG` appends a supplementary group without replacing the user's existing supplementary group memberships.

## 4. File Ownership and Permissions

| Command                             | Purpose                                                      |
| ----------------------------------- | ------------------------------------------------------------ |
| `ls -l file.txt`                    | Inspect file ownership and permissions                       |
| `ls -ld directory`                  | Inspect directory permissions                                |
| `chmod 640 file.txt`                | Set permissions to owner read/write, group read, others none |
| `chmod 660 file.txt`                | Give owner and group read/write access                       |
| `chmod 750 directory`               | Give owner full access, group read/execute, others none      |
| `chown username file.txt`           | Change file owner                                            |
| `chgrp groupname file.txt`          | Change group ownership                                       |
| `chown username:groupname file.txt` | Change owner and group together                              |
| `namei -l /path/to/file`            | Inspect permissions along a path                             |

### Permission values

* `r` = read
* `w` = write
* `x` = execute; for directories, it allows traversal
* `4` = read
* `2` = write
* `1` = execute

For example, permission `750` means:

* Owner: `rwx`
* Group: `r-x`
* Others: `---`

## 5. Processes and Resource Usage

| Command                       | Purpose                                          |
| ----------------------------- | ------------------------------------------------ |
| `ps aux`                      | List running processes                           |
| `ps aux --sort=-%cpu \| head` | Show processes sorted by CPU usage               |
| `ps aux --sort=-%mem \| head` | Show processes sorted by memory usage            |
| `top`                         | Monitor processes and resource usage live        |
| `free -h`                     | Display memory statistics                        |
| `kill PID`                    | Send the default termination signal to a process |
| `sleep 300 &`                 | Start a background test process                  |

**Safety:** Identify a process and understand its purpose before terminating it. Avoid killing unknown system or application processes.

## 6. Disk Usage

| Command            | Purpose                                     |
| ------------------ | ------------------------------------------- |
| `df -h`            | Display filesystem space usage              |
| `du -sh directory` | Display the total space used by a directory |
| `du -sh /var/*`    | Compare space used by entries under `/var`  |

Remember:

* `df` shows filesystem usage.
* `du` estimates the space used by files and directories.

## 7. Service Management

| Command                                  | Purpose                             |
| ---------------------------------------- | ----------------------------------- |
| `sudo systemctl status nginx --no-pager` | Inspect service status              |
| `sudo systemctl is-active nginx`         | Check whether the service is active |
| `sudo systemctl start nginx`             | Start a service                     |
| `sudo systemctl stop nginx`              | Stop a service                      |
| `sudo systemctl restart nginx`           | Restart a service                   |
| `sudo systemctl reload nginx`            | Request a configuration reload      |
| `sudo systemctl enable nginx`            | Enable automatic startup            |
| `sudo systemctl is-enabled nginx`        | Check boot enablement               |

Replace `nginx` with the relevant service name.

* **Start** means start now.
* **Enable** means configure automatic startup during boot.
* **Restart** stops and starts the service.
* **Reload** asks the service to apply supported configuration changes.

A reload may fail even when a service is running. Check its output and logs rather than repeatedly retrying.

## 8. Network Troubleshooting

| Command                    | Purpose                               |
| -------------------------- | ------------------------------------- |
| `ip addr`                  | Inspect interfaces and IP addresses   |
| `ip route`                 | Inspect the routing table             |
| `ping -4 -c 4 8.8.8.8`     | Test IPv4 reachability using ICMP     |
| `ss -lntp`                 | Inspect listening TCP sockets         |
| `ss -lntup`                | Inspect listening TCP and UDP sockets |
| `curl -I http://localhost` | Test the local HTTP response          |

A failed ping does not always mean a host is unavailable. ICMP can be blocked independently of application traffic.

## 9. DNS Troubleshooting

| Command                              | Purpose                                                |
| ------------------------------------ | ------------------------------------------------------ |
| `cat /etc/resolv.conf`               | Inspect resolver configuration                         |
| `getent ahostsv4 google.com`         | Test system IPv4 name resolution                       |
| `nslookup google.com`                | Perform a DNS lookup                                   |
| `nslookup google.com DNS-IP`         | Query a specific DNS resolver                          |
| `dig google.com A`                   | Query an IPv4 A record                                 |
| `dig @DNS-IP google.com A`           | Query a specific DNS server                            |
| `dig @DNS-IP server.lab.test +short` | Perform a concise DNS query against a specified server |

Replace `DNS-IP` with the correct resolver address for your environment.

## 10. BIND9 Configuration

| Command                                               | Purpose                                          |
| ----------------------------------------------------- | ------------------------------------------------ |
| `named -v`                                            | Display the BIND version                         |
| `sudo named-checkconf`                                | Validate BIND configuration syntax               |
| `sudo named-checkconf -z`                             | Validate configuration and attempt to load zones |
| `sudo named-checkzone lab.test /etc/bind/db.lab.test` | Validate the local zone file                     |
| `sudo systemctl status named --no-pager`              | Inspect BIND service status                      |
| `sudo journalctl -u named -n 50 --no-pager`           | Inspect recent BIND service logs                 |
| `sudo rndc status`                                    | Query the BIND control interface                 |

Configuration validation does not guarantee that every network query succeeds. Verify both service operation and DNS responses.

## 11. Nginx Troubleshooting

| Command                                     | Purpose                                |
| ------------------------------------------- | -------------------------------------- |
| `which nginx`                               | Locate the Nginx executable            |
| `sudo nginx -t`                             | Validate Nginx configuration           |
| `sudo ss -lntp \| grep ':80'`               | Check for a TCP listener on port 80    |
| `curl -I http://localhost`                  | Test a local HTTP response             |
| `sudo journalctl -u nginx -n 50 --no-pager` | Inspect recent service logs            |
| `sudo tail -n 50 /var/log/nginx/error.log`  | Inspect recent Nginx error-log entries |

## 12. SSH Troubleshooting

| Command                                   | Purpose                                  |
| ----------------------------------------- | ---------------------------------------- |
| `which ssh`                               | Locate the SSH client                    |
| `which sshd`                              | Locate the SSH server executable         |
| `sudo systemctl status ssh --no-pager`    | Inspect SSH service status               |
| `sudo ss -lntp \| grep ':22'`             | Check for a TCP listener on port 22      |
| `sudo sshd -t`                            | Validate SSH server configuration syntax |
| `ssh localhost`                           | Attempt a local SSH connection           |
| `ssh -v username@server`                  | Display detailed client diagnostics      |
| `sudo journalctl -u ssh -n 50 --no-pager` | Inspect recent SSH service logs          |

Do not publish passwords or private keys. Review verbose diagnostic output before sharing it publicly.

## 13. Log Analysis

| Command                                           | Purpose                            |
| ------------------------------------------------- | ---------------------------------- |
| `sudo journalctl -n 50 --no-pager`                | View recent journal entries        |
| `sudo journalctl -p err -n 30 --no-pager`         | View recent error-priority entries |
| `sudo journalctl -u nginx -n 50 --no-pager`       | View Nginx service logs            |
| `sudo journalctl -u ssh -n 50 --no-pager`         | View SSH service logs              |
| `sudo journalctl -u named -n 50 --no-pager`       | View BIND service logs             |
| `sudo journalctl -b --no-pager`                   | View logs from the current boot    |
| `sudo journalctl --since "1 hour ago" --no-pager` | View logs from a recent time range |
| `grep -i error file.log`                          | Search for error text              |
| `tail -f file.log`                                | Follow new log entries             |

## 14. General Troubleshooting Sequence

Use this as a guide rather than running every command for every issue:

1. Identify the symptom.
2. Check the relevant service or process.
3. Check connectivity, routing, or listening ports when relevant.
4. Review logs around the incident time.
5. Validate configuration and dependencies.
6. Apply the correction supported by the evidence.
7. Repeat the original test.
8. Record the result and any unresolved issue.

## 15. Important Notes

* Use the actual IP addresses, usernames, paths, and service names from your environment.
* WSL network addresses may change.
* A running service is not necessarily reachable on every interface.
* A configuration test does not prove that the service is fully functional.
* Record actual command outputs and do not claim an exercise succeeded until you verify it.
* Avoid uploading passwords, private keys, authentication tokens, or other sensitive data to a public repository.

