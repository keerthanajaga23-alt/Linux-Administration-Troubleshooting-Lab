# Lab 02: Linux Users, Groups, Ownership and Permissions

## 1. Objective

The objective of this lab is to practice Linux user and group administration, manage file ownership and permissions, configure a shared directory, and troubleshoot access-denied problems.

## 2. Lab Environment

* **Operating system:** Ubuntu 24.04 LTS
* **Platform:** Windows Subsystem for Linux 2 (WSL2)
* **Test group:** `itlab`
* **Test users:** `alice`, `bob`, `charlie`
* **Shared directory:** `/opt/itlab`

## 3. Create and Verify a Group

Create the `itlab` group:

```bash
sudo groupadd itlab
```

Verify that the group exists:

```bash
getent group itlab
```

**Explanation:**

* `groupadd` creates a Linux group.
* `getent group` checks the group database for the specified group.

## 4. Create Linux Users

Create a user with a home directory:

```bash
sudo useradd -m alice
```

Set the user's password:

```bash
sudo passwd alice
```

Add Alice to the `itlab` supplementary group:

```bash
sudo usermod -aG itlab alice
```

Repeat for Bob and Charlie

```bash
sudo useradd -m bob
sudo passwd bob
sudo usermod -aG itlab bob
```

```bash
sudo useradd -m charlie
sudo passwd charlie
```

For this exercise, Charlie is initially kept outside the `itlab` group so that access restrictions can be tested.

## 5. Verify User and Group Membership

Check Alice:

```bash
id alice
```

Check Bob:

```bash
id bob
```

Check Charlie:

```bash
id charlie
```

Check the group:

```bash
getent group itlab
```

**Explanation:**

* `id` displays a user's UID, primary GID, and supplementary groups.
* `getent passwd username` checks whether a user account exists.
* `getent group groupname` checks group information.

The `-aG` options in `usermod -aG` are important because `-a` appends a supplementary group instead of replacing existing supplementary group memberships.

Group membership changes may require a new login session to take effect in an ordinary user session.

## 6. Create a Shared Directory

Create the shared directory:

```bash
sudo mkdir -p /opt/itlab
```

Assign the `itlab` group as its group owner:

```bash
sudo chgrp itlab /opt/itlab
```

Set directory permissions:

```bash
sudo chmod 2770 /opt/itlab
```

Check the directory:

```bash
ls -ld /opt/itlab
```

The expected permission format is similar to:

```text
drwxrws---
```

**Explanation:**

* `mkdir -p` creates the directory if it does not already exist.
* `chgrp` changes the directory's group ownership.
* `chmod` changes permissions.
* `ls -ld` displays the directory's details rather than listing its contents.

### Understanding permission 2770

The `770` portion gives the owner and group read, write, and execute permissions. Other users receive no permissions.

The leading `2` enables the setgid bit on the directory. New files and subdirectories normally inherit the directory's group, which is useful for shared team folders.

## 7. Test Shared Directory Access

Create a test file as Alice:

```bash
sudo -u alice touch /opt/itlab/alice.txt
```

Check the result:

```bash
ls -l /opt/itlab
```

test bob directory access:

```bash
sudo -u bob ls -la /opt/itlab
```

**Explanation:**

* `sudo -u alice` runs a command as Alice.
* `touch` creates an empty file.
* `ls -la` lists files, including hidden files.

The commands test whether a user can access the shared directory under the configured permissions.

## 8. Practice File Permissions

Create a test file:

```bash
sudo touch /opt/itlab/private.txt
```

Set restrictive permissions:

```bash
sudo chmod 600 /opt/itlab/private.txt
```

Check the permissions:

```bash
ls -l /opt/itlab/private.txt
```

Permission `600` means:

| Category | Permission     |
| -------- | -------------- |
| Owner    | Read and write |
| Group    | No access      |
| Others   | No access      |

Now allow the owner and group to read and write:

```bash
sudo chmod 660 /opt/itlab/private.txt
```

Verify again:

```bash
ls -l /opt/itlab/private.txt
```

Permission `660` means:

| Category | Permission     |
| -------- | -------------- |
| Owner    | Read and write |
| Group    | Read and write |
| Others   | No access      |

**Important:** A user belonging to a group does not automatically have access to every file owned by that group. The file permissions must also grant the required access.

## 9. Practice File Ownership

Change the file owner to Alice:

```bash
sudo chown alice /opt/itlab/private.txt
```

Change the file's group:

```bash
sudo chgrp itlab /opt/itlab/private.txt
```

Change both owner and group in one command:

```bash
sudo chown alice:itlab /opt/itlab/private.txt
```

Verify the result:

```bash
ls -l /opt/itlab/private.txt
```

**Explanation:**

* `chown` changes the owner.
* `chgrp` changes the group owner.
* `chown alice:itlab` changes both.

## 10. Troubleshooting Scenario: Permission Denied

### Scenario

A user cannot access a file in `/opt/itlab`.

### Investigation

Check the user's identity and group memberships:

```bash
id username
```

Check file ownership and permissions:

```bash
ls -l /opt/itlab/filename
```

Check the directory permissions:

```bash
ls -ld /opt /opt/itlab
```

To inspect the permissions of each directory component along a path, use:

```bash
namei -l /opt/itlab/filename
```

### Troubleshooting workflow

1. Confirm that the user exists.
2. Check whether the user belongs to the required group.
3. Check the file's owner and group.
4. Check the file's read, write, and execute permissions.
5. Check the parent directory's traversal permissions.
6. Correct the specific issue.
7. Test access again as the affected user.

Do not change permissions broadly without understanding why access is denied.

## 11. Understanding Linux Permissions

Linux uses three basic permission types:

| Symbol | Meaning for a file   | Meaning for a directory                                       |
| ------ | -------------------- | ------------------------------------------------------------- |
| `r`    | Read file contents   | List directory names                                          |
| `w`    | Modify file contents | Create or remove directory entries, subject to other controls |
| `x`    | Execute a file       | Traverse or enter the directory                               |

Common numeric values:

| Value | Permissions |
| ----- | ----------- |
| `7`   | `rwx`       |
| `6`   | `rw-`       |
| `5`   | `r-x`       |
| `4`   | `r--`       |
| `0`   | `---`       |

## 12. Key Commands Reference

| Command         | Purpose                             |
| --------------- | ----------------------------------- |
| `groupadd`      | Create a group                      |
| `useradd -m`    | Create a user with a home directory |
| `passwd`        | Set or change a password            |
| `usermod -aG`   | Add a supplementary group           |
| `id`            | Display user and group IDs          |
| `getent passwd` | Check user account information      |
| `getent group`  | Check group information             |
| `mkdir -p`      | Create a directory                  |
| `chown`         | Change owner                        |
| `chgrp`         | Change group owner                  |
| `chmod`         | Change permissions                  |
| `ls -l`         | Display file details                |
| `ls -ld`        | Display directory details           |
| `namei -l`      | Inspect permissions along a path    |
| `sudo -u`       | Run a command as another user       |

## 13. Learning Outcomes

This lab practices:

* Linux user and group administration.
* Supplementary group membership.
* File ownership and permission management.
* Shared-directory access controls.
* The setgid bit on directories.
* Systematic investigation of permission-denied errors.

These are foundational skills for Linux administration and IT support troubleshooting.

