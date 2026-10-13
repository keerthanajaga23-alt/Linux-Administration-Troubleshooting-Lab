# Lab 09: SSH Administration and Troubleshooting

## 1. Objective

The objective of this lab is to understand Secure Shell (SSH), check the SSH server service, inspect port 22, validate SSH configuration, test SSH connectivity, and investigate common connection and authentication problems.

## 2. Lab Environment

* **Operating system:** Ubuntu 24.04 LTS
* **Platform:** Windows Subsystem for Linux 2 (WSL2)
* **Protocol:** Secure Shell (SSH)
* **Tools:** `ssh`, `sshd`, `systemctl`, `ss`, `journalctl`

Run the checks in Ubuntu and record your actual results. Do not mark an exercise as completed unless you have verified it.

## 3. What Is SSH?

SSH stands for **Secure Shell**. It is a network protocol used to establish encrypted connections to remote systems.

It is commonly used for:

* Remote Linux administration.
* Secure command-line access.
* Running commands on remote systems.
* Transferring files using related tools such as SCP and SFTP.

A basic SSH connection follows this pattern:

```text
SSH client
    |
    | Encrypted SSH connection
    v
SSH server (sshd)
    |
    v
User authentication
    |
    v
Remote shell or command
```

## 4. Important SSH Terms

* **SSH client:** The program used to connect to an SSH server. The command is normally `ssh`.
* **SSH server:** The component that accepts SSH connections. OpenSSH uses the `sshd` daemon.
* **Daemon:** A program that runs in the background.
* **Port 22:** The default TCP port used by SSH.
* **Authentication:** The process of verifying a user's identity.
* **Password authentication:** A method that verifies a user's password.
* **Public-key authentication:** A method that uses a private key on the client and a corresponding public key authorised by the server.
* **Private key:** A secret key that must be protected.
* **Public key:** The matching public part of a key pair.
* **`authorized_keys`:** A file that can contain public keys allowed to authenticate as a particular user.

## 5. Check Whether SSH Is Installed

Check for the SSH client:

```bash
which ssh
```

Check for the SSH server executable:

```bash
which sshd
```

The SSH client and server are separate components. Having the `ssh` command does not necessarily mean that the SSH server is installed.

On Ubuntu, the OpenSSH server package is normally called `openssh-server`.

Check whether it is installed:

```bash
dpkg -l openssh-server
```

If the server is not installed and you need it for this lab, install it:

```bash
sudo apt update
sudo apt install openssh-server
```

Do not reinstall the package if it is already installed.

## 6. Check the SSH Service

On Ubuntu, the OpenSSH server service is commonly named `ssh`.

Check its status:

```bash
sudo systemctl status ssh --no-pager
```

Check whether it is active:

```bash
sudo systemctl is-active ssh
```

Possible results include:

* `active`: The service is active.
* `inactive`: The service is not currently active.
* `failed`: The service encountered a failure.

If the service unit cannot be found, first verify that the server package is installed and identify the correct service name.

## 7. Start and Enable SSH

If the SSH server is installed but stopped, and you intend to use it, start the service:

```bash
sudo systemctl start ssh
```

Check the status:

```bash
sudo systemctl status ssh --no-pager
```

To configure SSH to start automatically during Linux system startup:

```bash
sudo systemctl enable ssh
```

Verify the setting:

```bash
sudo systemctl is-enabled ssh
```

### Start versus enable

* `systemctl start ssh` attempts to start the service now.
* `systemctl enable ssh` configures automatic startup during boot.
* `systemctl enable --now ssh` enables the service and attempts to start it immediately.

Under WSL2, automatic startup also depends on the WSL and systemd configuration.

## 8. Check Whether SSH Is Listening on Port 22

Run:

```bash
sudo ss -lntp | grep ':22'
```

This checks for listening TCP sockets matching port 22.

The options mean:

* `-l`: Show listening sockets.
* `-n`: Show numeric addresses and ports.
* `-t`: Show TCP sockets.
* `-p`: Display associated process information when available and permitted.

A listener may appear on `0.0.0.0:22`, `[::]:22`, or a specific local IP address, depending on the configuration.

**Important:** A server listening only on `127.0.0.1:22` may accept local connections but not connections through its external interface. Always consider the address as well as the port.

## 9. Validate the SSH Server Configuration

OpenSSH server configuration is commonly stored in:

```text
/etc/ssh/sshd_config
```

Validate the configuration syntax:

```bash
sudo sshd -t
```

No output normally indicates that the configuration syntax is valid.

If the command reports an error, inspect the reported location and correct the configuration before reloading or restarting the service.

To inspect the configuration locally:

```bash
sudo less /etc/ssh/sshd_config
```

Press `q` to exit `less`.

Do not change authentication settings simply to make a test pass. First understand the current configuration and the security implications of any change.

## 10. Test a Local SSH Connection

Attempt to connect to the local system:

```bash
ssh localhost
```

On the first connection, SSH may ask you to confirm the server's host key. For a local lab, verify that you are connecting to the intended local system before accepting it.

If password authentication is enabled, you may be prompted for the account password. If key-based authentication is required, use an appropriate configured key.

Once connected, run:

```bash
whoami
hostname
```

These commands identify the user and hostname of the system where the session is running.

Exit the SSH session:

```bash
exit
```

A successful local connection tests SSH on the local system. It does not by itself prove that remote clients can reach the server over the network.

## 11. Understand SSH Authentication

SSH may use different authentication methods, depending on server and client configuration.

### Password authentication

The server verifies the supplied password for the user account, subject to the account status and SSH policy.

### Public-key authentication

A key pair contains:

* A private key held securely by the client.
* A public key installed or otherwise authorised on the server.

A user's authorised public keys are commonly stored in:

```text
~/.ssh/authorized_keys
```

The private key should not be copied into the server's `authorized_keys` file and should never be uploaded to a public GitHub repository.

### File permissions

Incorrect permissions on the user's SSH directory or key files can prevent public-key authentication.

Inspect the relevant paths and permissions carefully before changing them. Use the account's actual home directory and the authentication method configured on the server.

## 12. Troubleshooting Scenario: Connection Refused

### Scenario

An SSH client reports that the connection was refused.

Example command:

```bash
ssh localhost
```

### Investigation

Check the service:

```bash
sudo systemctl status ssh --no-pager
```

Check for a listener on port 22:

```bash
sudo ss -lntp | grep ':22'
```

Validate the configuration:

```bash
sudo sshd -t
```

Review recent SSH service logs:

```bash
sudo journalctl -u ssh -n 50 --no-pager
```

Investigate whether:

* The SSH server is installed.
* The service is running.
* The expected address and port are listening.
* The configuration is valid.
* A local firewall or other network control is involved.

A connection-refused error does not by itself establish the exact root cause.

## 13. Troubleshooting Scenario: Connection Timeout

### Scenario

An SSH connection to a remote host does not complete before the timeout.

Investigate the network path before assuming that SSH authentication is the problem.

Useful commands include:

```bash
ip addr
ip route
```

Test the destination if ICMP is permitted:

```bash
ping -4 -c 4 <SERVER-IP>
```

Inspect the expected SSH listener on a server you administer:

```bash
sudo ss -lntp | grep ':22'
```

Review the server's SSH service status and logs when authorised.

Possible causes include:

* Network routing problems.
* An unavailable server.
* Firewall or security rules.
* SSH listening on a different address or port.
* A network path that drops the connection attempt.

A failed ping does not prove that SSH is unavailable, because ICMP may be blocked separately.

## 14. Troubleshooting Scenario: Permission Denied (Authentication Failed)

### Scenario

The SSH server is reachable, but the connection fails with an authentication error.

Investigate:

1. Whether the username is correct.
2. Whether the account exists and is allowed to log in.
3. Which authentication method the client is using.
4. Whether the correct password or private key is being supplied.
5. Whether the matching public key is authorised on the server.
6. Whether SSH server policy permits the attempted authentication.
7. What the server logs report at the time of the attempt.

Check whether the account exists:

```bash
getent passwd <USERNAME>
```

On the client, enable verbose connection output:

```bash
ssh -v <USERNAME>@<SERVER>
```

Replace the placeholders with the actual username and hostname or IP address.

Verbose output helps identify which authentication methods are attempted and where the connection fails. Review it before sharing, because diagnostic output may contain internal usernames, hostnames, paths, and other information.

Check server logs, when available:

```bash
sudo journalctl -u ssh -n 50 --no-pager
```

Do not repeatedly guess passwords or disable security controls without understanding the cause.

## 15. Review SSH Configuration Safely

If a configuration change is necessary, make a backup before editing:

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

Validate the configuration after editing:

```bash
sudo sshd -t
```

Only if validation succeeds, apply changes using an appropriate service reload or restart:

```bash
sudo systemctl reload ssh
```

Then verify:

```bash
sudo systemctl status ssh --no-pager
```

If reload is unsupported or fails, investigate the service output and logs. On remote systems, maintain an existing working administrative session and an alternative access method before changing SSH configuration.

Do not enable root login or weaken authentication policies as a shortcut.

## 16. Key Commands Reference

| Command                   | Purpose                                  |
| ------------------------- | ---------------------------------------- |
| `which ssh`               | Locate the SSH client                    |
| `which sshd`              | Locate the SSH server executable         |
| `dpkg -l openssh-server`  | Check the server package status          |
| `systemctl status ssh`    | Inspect the SSH service                  |
| `systemctl is-active ssh` | Check whether the service is active      |
| `systemctl start ssh`     | Start the service                        |
| `systemctl enable ssh`    | Enable automatic startup                 |
| `ss -lntp`                | Inspect listening TCP sockets            |
| `sshd -t`                 | Validate SSH server configuration syntax |
| `ssh localhost`           | Attempt a local SSH connection           |
| `ssh -v USER@SERVER`      | Display verbose client diagnostics       |
| `getent passwd USER`      | Check account information                |
| `journalctl -u ssh`       | Inspect SSH service logs                 |

Replace placeholders such as `USER` and `SERVER` with the appropriate values.

## 17. Learning Outcomes

This lab provides practice in:

* Understanding SSH clients and servers.
* Managing the OpenSSH service.
* Inspecting listening ports.
* Validating SSH server configuration.
* Testing a local SSH connection.
* Understanding password and public-key authentication.
* Investigating network, connection, and authentication failures.
* Verifying changes while preserving security.


