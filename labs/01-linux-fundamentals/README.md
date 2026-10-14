# Lab 01: Linux Fundamentals and File Operations

## Objective

Practice essential Linux commands for navigating directories, creating files, reading file contents, copying files, renaming files and removing files.

## Environment

* Operating system: Ubuntu 24.04 LTS
* Platform: WSL2
* Practice directory: `~/linux-practice`

## Tasks and Commands

### 1. Display the Current Directory

```bash
pwd
```

**Purpose:** Displays the full path of the current working directory.

### 2. Create and Enter a Directory

```bash
mkdir -p ~/linux-practice
cd ~/linux-practice
```

**Purpose:** Creates the practice directory and changes into it.

### 3. Create and List a File

```bash
touch notes.txt
ls -l
```

**Purpose:** Creates an empty file and lists the files with detailed information.

### 4. Write and Read File Contents

```bash
echo "Linux fundamentals practice" > notes.txt
cat notes.txt
```

**Purpose:** Writes text into the file and displays its contents.

### 5. Copy and Rename a File

```bash
cp notes.txt backup.txt
mv backup.txt notes-backup.txt
ls -l
```

**Purpose:** Copies a file, renames the copy and verifies the files in the directory.

### 6. Remove a Test File

```bash
rm notes-backup.txt
ls -l
```

**Purpose:** Removes the practice copy and checks that it no longer appears in the directory.

## Commands Learned

| Command | Purpose                       |
| ------- | ----------------------------- |
| `pwd`   | Display the current directory |
| `mkdir` | Create a directory            |
| `cd`    | Change directory              |
| `touch` | Create an empty file          |
| `ls -l` | List files with details       |
| `echo`  | Display or write text         |
| `cat`   | Display file contents         |
| `cp`    | Copy a file                   |
| `mv`    | Move or rename a file         |
| `rm`    | Remove a file                 |

## Learning Outcome

I practiced basic Linux filesystem navigation and file operations using Ubuntu on WSL2. These commands provide a foundation for Linux administration and troubleshooting.

