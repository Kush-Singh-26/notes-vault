---
title: "Linux System Programming"
description: "Notes on system programming."
---

## Basics of System Programming

- System software acts like an interface between kernel and the C library.
- 3 key components :
  - `syscall`
  - C library (`glibc`)
  - C compiler toolchai / linker

---

- **System calls** are kernel functions invoked from user space to perform tasks like reading and writing.
- The C library provides wrappers around syscalls, allowing user applications to call kernel functions through standard function prototypes.

> **Kernel Space** : The highly privileged region of memory where the operating system kernel, core extensions, and essential device drivers reside. It has unrestricted access to all system memory and hardware components.

> **User Space** : The unprivileged memory area where regular user applications, libraries, and utilities execute 

---

## POSIX API (Portable Operating System Interface)

- **API** (Application Programming Interface) defines how software components interact at the source code level, ensuring source compatibility across architectures.
- **ABI** (Application Binary Interface) defines binary compatibility, including calling conventions and register use, which varies by hardware architecture.
- **POSIX** is a standardized API for Unix-like systems, created to unify different Unix variants and ensure portability of software across platforms.

> **POSIX** defines APIs for file and directory operations, clocks, timers, semaphores, shared memory, and threads.

---

## Everything is a File

Every component that is a file. Devices are also accessed similar to files.

- `/` : absolute path from root direcrory. eg. `/path/to/file`
- `No /` : relative path from working directory. eg. `path/to/file`

---


- ach file/directory has:

an inode number
an inode structure
- Files are referenced and accessed using `inode` : metadata used to track a file
- FIxed size of `128bytes`. 

|Inode Components|
|---|
|Accessed Time|
|Size|
|UID|
|GID|
|Permission|
|Location on Disk|

- Each file/directory has:
  - an inode number
  - an inode structure

> So, each dir is a file containing the mapping : filename -> inode number

---

## Blocks

- The smallest addressable unit in a Linux filesystem is a block, which is a power-of-two multiple of the sector size (commonly 512 bytes).
- Using blocks instead of addressing individual bytes improves efficiency in storage and data transfer.
- eg. : addressing every byte in a 1 terabyte filesystem requires 34 bits, but using 512-byte blocks reduces this to 30 bits, and 4K blocks reduce it further to 28 bits.
- Block-oriented storage reduces the amount of metadata needed and improves performance when accessing files.

## Links

### Hard Links

- Multiple filenames can point to same inode.

```bash
ln <og_file> <copy_file>
```

- Both names point to same inode.
- Deleting one filename does NOT delete file data until:
  - link count becomes 0
  - no process has file open

### Symbolic Links (symlink or soft link)

```bash
ls -s 
```

- Symlink gets its own inode.
- Its data contains pathname

---

- Every directory contains:
  - `.`   -> current directory
  - `..`  -> parent directory

## File Types

| Char | Type         |
| ---- | ------------ |
| `-`  | regular file |
| `d`  | directory    |
| `l`  | symlink      |
| `c`  | char device  |
| `b`  | block device |
| `s`  | socket       |
| `p`  | named pipe   |


## File Descriptors

| FD | Meaning |
| -- | ------- |
| 0  | stdin   |
| 1  | stdout  |
| 2  | stderr  |


## Filesystem application calls

```bash
open()
read()
write()
```

---

## Linux Processes

- A Linux process is executable code running on hardware, managed by the kernel via system calls for resources like timers and files.
- Each process has a unique process ID and runs on a virtualized CPU and memory, giving the illusion of dedicated resources.

## Linux Threads

- A process consists of one or more threads, which are units of activity within the process.
- Threads share the same virtual memory space but have individual stacks for local variables and call tracking, requiring synchronization for shared data access.

## Inter-Process Communication (IPC)

- Processes have isolated memory spaces, so sharing data requires IPC mechanisms like signals, pipes, semaphores, message queues, and shared memory.
- Signals are asynchronous notifications sent between processes or from the kernel, with signal handlers managing responses safely to avoid conflicts during asynchronous execution.

---

## User Groups 

### Users and User IDs (UIDs)

- Each Linux user account has a unique user ID (UID) that identifies it on the system.
- The root user has a special UID of 0, granting it full system privileges.

### Groups and Group IDs (GIDs)

- Users belong to one or more groups, each identified by a group ID (GID).
- The /etc/group file maps group names to their GIDs.

### Permisions

| Category | Permission | Symbol | Bit Position | Octal Value |
|----------|------------|--------|-------------|-------------|
| User (Owner) | Read | r | 8 | 0400 |
| User (Owner) | Write | w | 7 | 0200 |
| User (Owner) | Execute | x | 6 | 0100 |
| Group | Read | r | 5 | 0040 |
| Group | Write | w | 4 | 0020 |
| Group | Execute | x | 3 | 0010 |
| Others (Everyone) | Read | r | 2 | 0004 |
| Others (Everyone) | Write | w | 1 | 0002 |
| Others (Everyone) | Execute | x | 0 | 0001 |

### Explanation

Linux and Unix file permissions are represented using **three sets of permissions**:

1. **User (Owner)** – The file owner.
2. **Group** – Users belonging to the file's group.
3. **Others (Everyone)** – All other users.

Each set can have three permissions:

| Permission | Symbol | Meaning                                                    |
| ---------- | ------ | ---------------------------------------------------------- |
| Read       | `r`    | View file contents or list directory contents              |
| Write      | `w`    | Modify file contents or create/delete files in a directory |
| Execute    | `x`    | Run a file as a program or access a directory              |

#### Octal Values

Each permission has a numeric value:

| Permission    | Value |
| ------------- | ----- |
| Read (`r`)    | 4     |
| Write (`w`)   | 2     |
| Execute (`x`) | 1     |

The permissions for each category are added together:

| Permissions | Calculation | Octal |
| ----------- | ----------- | ----- |
| `rwx`       | 4 + 2 + 1   | 7     |
| `rw-`       | 4 + 2       | 6     |
| `r-x`       | 4 + 1       | 5     |
| `r--`       | 4           | 4     |
| `---`       | 0           | 0     |

### Example: `chmod 755`

```text
755 = 7 5 5
      │ │ └── Others: r-x
      │ └──── Group:  r-x
      └────── Owner:  rwx
```

Result:

```bash
rwxr-xr-x
```

### Example: `chmod 644`

```text
644 = 6 4 4
      │ │ └── Others: r--
      │ └──── Group:  r--
      └────── Owner:  rw-
```

Result:

```bash
rw-r--r--
```

### Bit Layout

```text
User           Group          Others
r   w   x      r   w   x      r   w   x
8   7   6      5   4   3      2   1   0  (Bit Positions)

0400 0200 0100 0040 0020 0010 0004 0002 0001
```

---

## Error Handling in Linux with `errno`

- In Linux system programming, many POSIX and C library functions report errors using a special variable called **`errno`**.
- `errno` is used to store an error code when a function fails.

```c
FILE *fp = fopen("data.txt", "r");
```

- If `data.txt` does not exist:
  - * `fopen()` returns `NULL`
  - * `errno` is set to an error code such as `ENOENT` ("No such file or directory")

```c
#include <stdio.h>
#include <errno.h>

int main() {
    FILE *fp = fopen("missing.txt", "r");

    if (fp == NULL) {
        perror("fopen");
    }

    return 0;
}
```

Possible output:

```text
fopen: No such file or directory
```

`perror()` automatically reads `errno` and prints a human-readable message.

---

### Converting `errno` to a String

- can use `strerror()`:

```c
#include <stdio.h>
#include <string.h>
#include <errno.h>

int main() {
    FILE *fp = fopen("missing.txt", "r");

    if (fp == NULL) {
        printf("Error: %s\n", strerror(errno));
    }

    return 0;
}
```

Output:

```text
Error: No such file or directory
```

---

### Important Consideration: Save `errno`

After an error occurs, another library function might change `errno`.

Bad:

```c
fopen(...);
printf("Something\n");   // may modify errno
printf("%s\n", strerror(errno));
```

Better:

```c
fopen(...);

int saved_errno = errno;

printf("Something\n");
printf("%s\n", strerror(saved_errno));
```

Saving the value ensures you report the correct error.

---

### `errno` in Multithreaded Programs

Although `errno` looks like a global variable, in modern systems it is **thread-local**.

- This means:
  - Each thread has its own copy of `errno`.
  - Errors in one thread do not overwrite errors in another thread.

```text
Thread A: errno = ENOENT
Thread B: errno = EACCES
```

Both values can exist simultaneously without conflict.

### Key Points to Remember

1. Many POSIX functions indicate failure through return values (`NULL`, `-1`, etc.).
2. When a failure occurs, `errno` contains the specific error code.
3. Use `perror()` or `strerror(errno)` to display readable error messages.
4. Save `errno` immediately if subsequent function calls may overwrite it.
5. In multithreaded programs, `errno` is thread-local, not truly shared globally.

---

## Embedded Linux Toolchain

- A toolchain is a set of programs that transform source code into an executable application.

Componenets include :
- **Compiler (GCC)** – Converts C source code into machine code.
- **Assembler** – Converts assembly language into object files.
- **Linker (ld)** – Combines object files and libraries into a final executable.
- **Runtime Libraries** – Libraries needed when the program runs (e.g., glibc).

```
main.c
   |
   v
Compiler (gcc)
   |
   v
main.o
   |
   v
Linker
   |
   v
Executable
```

- Toolchains can be manually created (download and build : `GCC`, `Binutils`, `glibc/uClibc/musl`) or use automated setup (preferred) like : `buildroot` or `yocto`.

### Native vs Cross Toolchain

- native compiler builds software for the same architecture it runs on.

```
Host: x86_64 PC
Compiler: x86_64 gcc
Output: x86_64 executable
```

- The generated program can run directly on the host.

---

- cross compiler runs on one architecture but generates code for another.

```
Host: x86_64 PC

Compiler:
aarch64-linux-gnu-gcc

Target:
ARM64 board
```

```
x86 PC
   |
   | Cross Compile
   v
ARM Executable
   |
   v
ARM Board
```

- This is the standard approach in embedded Linux.
  - emebedded devices have slow cpus, limited ram and small storage generally

---

### Compiler Prefix

```bash
aarch64-none-linux-gnu-gcc
```

| Part    | Meaning                          |
| ------- | -------------------------------- |
| aarch64 | Target CPU architecture          |
| none    | Vendor field                     |
| linux   | Target kernel                    |
| gnu     | C library / operating system ABI |
| gcc     | Compiler                         |

---

### `Sysroot`

> A copy of the target device's root filesystem stored inside the toolchain.

```
sysroot/
├── usr/include
├── usr/lib
├── lib
└── etc
```

It contains the following ***for the target architecture***
- Header files `(.h)`
- Shared Libraries `(.so)`
- Static Libraries `(.a)`

> It ensures target arch headers and libs are used by the compiler.

---

### Static Linking

- all required library code is copied into the executable.

```
Application
 + libc
 + libm
 + other libraries
------------------
Single executable
```

```bash
gcc -static main.c -o app
```

- self contained executable, no external libs required.
- executable size becomes large.

### Dynamic Linking

- keeps libraries separate.

```
Application
     |
     +--> libc.so
     +--> libm.so
```

- exe contains only reference to the libs
- Smaller binaries, Libraries shared by multiple applications

---

to inspect a binary's dependencies :

```bash
readelf -d app
```

```bash
ldd app
```

### Binary analysis tools 

- readelf

```bash
readelf -h app
```

- objdump

```bash
objdump -d app
```

- strip

```bash
strip app
```

---

### Complete flow

```
Source Code (C)
        |
        v
Cross Compiler
(aarch64-linux-gnu-gcc)
        |
        v
Uses Sysroot
(ARM headers + libraries)
        |
        v
ARM Executable
        |
        v
Copy to Target Device
        |
        v
Run on Embedded Linux
```

1. A toolchain consists of the compiler, assembler, linker, and runtime libraries.
2. Embedded Linux development primarily uses cross-compilation.
3. A sysroot contains the target system's headers and libraries and prevents mixing host and target files.
4. Static linking creates self-contained executables but increases size.
5. Dynamic linking reduces size by sharing libraries but requires those libraries on the target.
6. Tools like readelf, objdump, and strip help inspect, analyze, and optimize binaries.
7. Build systems such as Buildroot and Yocto automate toolchain creation and embedded Linux development.

---

## Logging in Linux System Programming

> A logging system stores messages in log files so developers can inspect what happened even after the event occurred.

### `Syslog`

It consists of:

1. Application Program
  - Generates log messages.
  - Uses the syslog API.

2. `syslogd` / `rsyslogd`
  - A background daemon that receives log messages.
  - Decides where to store them according to configuration rules.

3. Log Files
  - Usually stored under : `/var/log`

```bash
/var/log/syslog
/var/log/messages
/var/log/auth.log
```

```
Application
     |
     v
  syslog()
     |
     v
 syslogd/rsyslogd
     |
     v
 Configuration Rules
     |
     +----> Log File
     +----> Terminal
     +----> Remote Server
```

### Facility in Syslog

- facility identifies the source or category of a log message

| Facility      | Purpose                    |
| ------------- | -------------------------- |
| user          | User applications          |
| daemon        | System daemons             |
| kern          | Kernel messages            |
| mail          | Mail system                |
| auth          | Authentication             |
| local0–local7 | Custom application logging |

---

``` bash
LOG_LOCAL0
LOG_LOCAL1
...
LOG_LOCAL7
```

- reserved for custom programs.

---

### Priorities

Every log message has a priority indicating its importance.

| Priority    | Meaning                   |
| ----------- | ------------------------- |
| LOG_EMERG   | System unusable           |
| LOG_ALERT   | Immediate action required |
| LOG_CRIT    | Critical condition        |
| LOG_ERR     | Error                     |
| LOG_WARNING | Warning                   |
| LOG_NOTICE  | Important normal event    |
| LOG_INFO    | Informational             |
| LOG_DEBUG   | Debugging information     |

```c
syslog(LOG_ERR, "Sensor failed");
syslog(LOG_WARNING, "Temperature high");
syslog(LOG_DEBUG, "Current value=%d", value);
```

---

###  `Syslog` API

1. Open the Log

```c
openlog("myapp", LOG_PID, LOG_LOCAL0);
```

Parameters:
  - "myapp" : program name
  - LOG_PID : include process ID in logs
  - LOG_LOCAL0 : facility

2. Write Messages

```c
syslog(LOG_INFO, "Application started");
```

3. Close the Log

```c
closelog();
```

---


```c
#include <syslog.h>

int main()
{
    openlog("sensor_app", LOG_PID, LOG_LOCAL0);

    syslog(LOG_INFO, "Application started");

    syslog(LOG_WARNING,
           "Temperature exceeds threshold");

    syslog(LOG_ERR,
           "Sensor communication failed");

    closelog();

    return 0;
}
```

> Linux provides syslog as a centralized logging framework where applications send messages with a facility and priority, and the syslog daemon (syslogd or rsyslogd) stores or routes those messages according to configuration rules, usually into files under /var/log.

---

