# Lab 03: Linux Process, CPU and Memory Management

## 1. Objective

The objective of this lab is to learn how to identify running processes, monitor CPU and memory usage, investigate resource consumption, and safely stop a process.

## 2. Lab Environment

* **Operating system:** Ubuntu 24.04 LTS
* **Platform:** Windows Subsystem for Linux 2 (WSL2)
* **Tools:** `ps`, `top`, `free`, `kill`, `sleep`

## 3. Understand Linux Processes

A **process** is a running instance of a program.

Each process has a Process ID (PID), which can be used to identify and manage it.

Common process-related terms:

* **PID:** Process ID
* **PPID:** Parent Process ID
* **CPU usage:** The amount of processor capacity a process uses
* **Memory usage:** The amount of RAM a process uses
* **Foreground process:** A process running in the current terminal session
* **Background process:** A process running without blocking the terminal

## 4. List Running Processes

Run:

```bash
ps aux
```

This displays running processes, including the user, PID, CPU usage, memory usage, and command.

To show processes using the most CPU:

```bash
ps aux --sort=-%cpu | head
```

To show processes using the most memory:

```bash
ps aux --sort=-%mem | head
```

**Explanation:**

* `ps` displays process information.
* `a` includes processes associated with terminals from different users.
* `u` displays user-oriented details.
* `x` includes processes without a controlling terminal.
* `--sort=-%cpu` sorts processes by CPU usage, highest first.
* `--sort=-%mem` sorts processes by memory usage, highest first.
* `head` displays the first few lines.

## 5. Monitor Processes in Real Time

Run:

```bash
top
```

This provides a continuously updated view of system activity and running processes.

Observe:

* CPU usage
* Memory usage
* Process IDs
* Process states
* Resource consumption by individual processes

Press `q` to exit.

## 6. Check Memory Usage

Run:

```bash
free -h
```

This displays memory statistics in a human-readable format.

Important fields include:

* **total:** Total memory available to the system
* **used:** Memory currently in use, according to the tool's calculation
* **free:** Memory that is currently unused
* **available:** An estimate of memory available for new applications without swapping

Linux often uses available memory for caching, so low `free` memory alone does not necessarily indicate a memory problem. Review the `available` value and the overall system behaviour.

## 7. Create a Safe Test Process

Start a process that waits for five minutes:

```bash
sleep 300 &
```

The `&` starts the process in the background.

Find it:

```bash
ps aux | grep '[s]leep 300'
```

Record its PID from the output.

I inspect it using:

```bash
ps -p <PID> -o pid,ppid,stat,cmd
```

Replace `<PID>` with the actual process ID.

**Purpose:** Practice identifying a process before managing it.

## 8. Stop the Test Process

Use the PID from the previous step:

```bash
kill <PID>
```

Replace `<PID>` with the actual process ID.

Verify whether it has stopped:

```bash
ps -p <PID> -o pid,stat,cmd
```

If no process details are returned, the process is no longer present.

The `kill` command sends a signal to a process. By default, it sends `SIGTERM`, which requests graceful termination. Avoid terminating unknown system processes without understanding their purpose.

## 9. Troubleshooting Scenario: High CPU Usage

### Scenario

A user reports that a Linux system is running slowly.

### Investigation

Check the processes using the most CPU:

```bash
ps aux --sort=-%cpu | head
```

Alternatively:

```bash
top
```

Investigate:

1. Which process is consuming the most CPU?
2. Is the process expected to be running?
3. Does the process show sustained high CPU usage?
4. Are there relevant application or service logs?
5. Did the issue begin after a recent change?

Do not immediately terminate a high-CPU process. Identify its purpose and investigate the cause first.

## 10. Troubleshooting Scenario: High Memory Usage

### Scenario

A user reports that an application is slow or the system is running out of memory.

### Investigation

Check system memory:

```bash
free -h
```

Identify processes using the most memory:

```bash
ps aux --sort=-%mem | head
```

You can also inspect live resource usage using:

```bash
top
```

Investigate the process, its normal resource requirements, any recent changes, and relevant application logs before taking action.

## 11. Troubleshooting Workflow

When investigating a performance issue:

1. Understand the reported symptom.
2. Check overall CPU and memory usage.
3. Identify resource-intensive processes.
4. Investigate the process and its purpose.
5. Review relevant logs and application behaviour.
6. Take an appropriate action if justified.
7. Monitor the system and confirm whether performance improves.

## 12. Key Commands Reference

| Command                       | Purpose                                     |
| ----------------------------- | ------------------------------------------- |
| `ps aux`                      | List running processes                      |
| `ps aux --sort=-%cpu \| head` | Display processes with high CPU usage       |
| `ps aux --sort=-%mem \| head` | Display processes with high memory usage    |
| `top`                         | Monitor processes and system resources live |
| `free -h`                     | Display memory statistics                   |
| `sleep 300 &`                 | Start a background test process             |
| `ps -p PID`                   | Inspect a specific process                  |
| `kill PID`                    | Send a signal to a process                  |

## 13 Learning Outcomes

This lab provides practice in:

* Identifying and inspecting Linux processes.
* Monitoring CPU and memory consumption.
* Understanding process IDs and signals.
* Safely stopping a test process.
* Applying a structured approach to Linux performance troubleshooting.

