## Source:
[The Missing Semester](https://missing.csail.mit.edu/2020/)
[The Linux Journey](https://labex.io/linuxjourney)
How Linux Works – Brian Ward (Ch 1-2-3-7-8)
## Shell  

- We will focus on the Bourne Again SHell, or “bash” for short. This is one of the most widely used shells.  
- The shell parse the command by whitespace and run program indicated with first word, others as arguments can access.  
- Can use \ to escape special character and provide to the arguments.  

- On Linux and macOS, the path / is the “root” of the file system.  
- A path that starts with / is called an absolute path. Any other path is a relative path.  
- In a path, . refers to the current directory, and .. to its parent directory.  
- PATH: shell uses $PATH to find commands. If not in PATH → run with full path (/bin/echo).  

**File types (from ls -l)**  
- First character = file type:  
  - `-` regular file  
  - `d` directory  
  - `l` symlink (link)  
  - `c` character device (/dev/tty)  
  - `b` block device (/dev/sda)  
  - `p` pipe (FIFO)  
  - `s` socket (used by services)  
- Next 9 chars = permissions: rwx (owner)(group)(others)  

- The cd is a shell builtin -> change its parent's current working directory.

- A shebang (#!) is the first line of a script that tells the system which interpreter to use.  
- Used only when executing script directly (`./script.sh`).  

**Scripts:**  
- Need execute permission → `chmod +x script.sh`  
- If use `sh script.sh` -> run `/bin/sh` and feed the script as data → only needs read (r)  
- Need shebang → `#!/usr/bin/env bash`  

**Quotes:**  
- `'text'` = literal (no variable expand, prevent history expansion)  
- `"text"` = expands variables ($HOME)  


---

## System Overview  
*How Linux Works – Brian Ward (Ch 1)*

- A process is a program in execution.  
- The kernel runs in kernel mode (privileged) -> memory area only access: kernel space.  
- User mode restricts access to hardware and memory -> user space. If a process crashes -> cleaned up by kernel.  

**Main memory areas:**  
- User space: where user processes run.  
- Kernel space: where the kernel and kernel extensions run.  
- Buffer cache: caches disk blocks for faster access.  
- Slab allocator: manages memory for kernel objects.  

**The kernel**  
- Split memory into subdivisions, each process gets its own virtual memory space -> kernel ensures each process can't access memory of another process.  
- Managing task in four areas:  
  - Process: determine which process allowed to use CPU.  
  - Memory: Track all of memory: what is allocated to a process, what might be shared between processes, what is free.  
  - Device drivers: acts as an interface between hardware and software -> operate the hardware.  
  - System call and support: Processes normally use system calls to communicate with the kernel.  

**Process Management**  
- Only one process can use the CPU at a time -> Context switch: saving the state of the current process and loading the state of the next process.  
- Each piece of time call time slice -> give processors enough time to each process. The slices so small -> give the illusion that multiple processes are running simultaneously -> multitasking.  
When a process is running in user mode and its slice time is up:  
  1. The CPU (hardware) interrupts the current process based on internal timer, switches to kernel mode and hands control to the kernel.  
  2. The kernel records current state of CPU and the process -> essential to resume later.  
  3. The kernel performs any task during preceding time slice (e.g., handling I/O).  
  4. The kernel analyzes list of runnable processes and selects the next process to run.  
  5. The kernel prepares the memory for the new process and prepares the CPU.  
  6. The kernel tells the CPU how long the time slice needed.  
  7. The kernel switches CPU to user mode and hands control of CPU to the new process.  
-> The kernel runs between each time slice, during context switches.  

**Memory Management**  
- The kernel must have its own memory space -> protected from user processes.  
- Each user process needs its own memory space -> virtual memory. Can share memory.  
- Thanks to memory management unit (MMU) -> Process don’t need to know the actual physical address of memory -> use virtual address (like it had an entire machine to itself).  
- The kernel keeps track of memory usage with a data structure called a page table -> map virtual, when a process accesses memory -> the MMU translates it.  

**Device Drivers and Management**  
- A device is accessible only in kernel mode.  
- Device driver had been part of the kernel, they strive to present a uniform interface to user processes.  

**System Calls and Support**  
Since user programs can’t just poke the hardware, they need a safe gate to ask the kernel to do privileged things on their behalf.  
That gate is a system call (syscall).  
- `fork()`: kernel create an identical copy of the process.  
- `exec()`: replace the current process with a new program called.  
When call a new process -> `fork()` to create copy (shell) -> `exec()` to run new program of this child shell. (except init)  
- A pseudodevice is a software-only device that behaves like a hardware device from the user’s point of view (lives in /dev).  
A process must use system calls (`open()`) to interact with pseudodevices.  

When something breaks, this user space layering helps you know where to look:  
Top (user-facing applications) → application crash.  
Middle (utility services) → service misconfiguration.  
Bottom (basic user-space components) → system daemon failure.  
Kernel (core OS) → device driver or syscall problem.  

Root still runs in user mode, not kernel mode means even root can’t directly execute kernel instructions.  
It still has to use syscalls like everyone else, just never gets permission denied errors.

---

## File Permissions & Ownership
- Some executable files have s instead of x -> setuid bit -> run as the file owner.
 
`chmod` changes permissions:
    - `chmod go+r file` — add read for group and others (g and o)
    - To remove permission, use - instead 
    - Can use numeric mode: rwx = 4+2+1 = 7
      - `chmod 755 file` → owner=rwx(7), group=rx(5), others=rx(5)
      - `chmod 644 file` → owner=rw(6), group=r(4), others=r(4)

- Can list contents of dir if it's executable (x), but can only access files if have (r) permission.

Change ownership:
    - `sudo chown user file` — change owner
    - `sudo chgrp group file` — give group permission of group
    - `sudo chown user:group file` — combine

- Sticky bit on dir (t) → only owner of file in that dir can delete/rename the file.
- `umask` sets default permissions for new files/dirs. Default is 022 
- Setuid (s) on executable file → run as file owner.
- Setgid (s) on executable file → run as file group.

`/etc/sudoers` — define which users can run sudo and what commands they can run.


## FHS

**Root Directory Structure:**

- `/bin` — Essential executables (ls, cp) available to all users. Binary + shell scripts.
- `/sbin` — System executables for system management. Requires root privileges.
- `/boot` — Kernel boot loader files. First-stage startup only.
- `/dev` — Device files (character, block devices).
- `/etc` — System configuration files (passwords, networking, services).
- `/home` — User home directories.
- `/lib` — Shared libraries used by executables. Static libs in `/usr/lib`.
- `/proc` — Virtual filesystem exposing process and kernel info. Not real files.
- `/run` — Runtime data (PIDs, sockets, status). Replaces old `/var/run`.
- `/sys` — Device and system interface (similar to `/proc`).
- `/tmp` — Temporary files. Cleared on boot. Any user can write. Shared space.
- `/usr` — User-space programs and data (bulk of system). Mirrors root structure.
- `/var` — Variable data (logs, caches, user tracking). `/var/tmp` persists across boots.

**The `/usr` directory:**

- `/usr/bin` — User executables.
- `/usr/sbin` — System executables (non-essential).
- `/usr/lib` — Libraries for `/usr/bin` and `/usr/sbin`.
- `/usr/local` — Admin-installed software (mimics `/` structure).
- `/usr/include` — C header files.
- `/usr/share` — Architecture-independent data (man pages, docs).

**Kernel Location:**

- Binary: `/vmlinuz` or `/boot/vmlinuz` — loaded by boot loader into memory.
- Modules: `/lib/modules` — loadable kernel modules (drivers) loaded on demand. 

---

## Process Management
- Each process has a unique Process ID (PID).

$ ps
  PID  TTY  STAT  TIME  COMMAND
  520  p0   S     0:00  -bash
  545  ?    S     3:59  /usr/X11R6/bin/ctwm -W
  548  ?    S     0:10  xclock -geometry -0-0
 2159  p3   R     0:00  ps

- **PID**  
  Process ID — the unique identifier for each process.

- **TTY**  
  The terminal (device) where the process is attached or running.  
  If you see `?`, the process is not connected to any terminal (e.g., a background or system process).

Two main types: terminal devices and pseudo-terminals (pty).  
  - Terminal devices: physical terminals (text-based) or virtual consoles (e.g., `/dev/tty1`).  
  - Pseudo-terminals: used by terminal emulators (e.g., `/dev/pts/0`).

- **STAT**  
  Process status.  
  Common codes:
  - `R` — Running 
  - `S` — Sleeping
  - `D` — Uninterruptible sleep
  - `T` — Stopped
  - `Z` — Zombie
  - `<` — High-priority process
  - `N` — Low-priority (nice) process  

- **TIME**  
  Total CPU time the process has consumed (in minutes:seconds).  
  **Note:** This is not “wall-clock time” since the process started, only the time it has actually run on the CPU.

- **COMMAND**  
  The command (and arguments) used to start the process.  
  - The shell may display the expanded version of the command (after globbing or variable expansion).  
  - A process can also change its own command string after starting.

- PIDs are unique while running but can be reused. ($$ gives current shell PID).
- ps option: x(all running), a(all users), u(more details), w(wide output).
- `top` — interactive process viewer, real-time updates.

**Termination**
Termination has two steps:
	1.	Child calls `_exit()` → marks itself finished.
	2.	Parent calls `wait()` → cleans it up.
- If parent dies first -> init (PID 1) adopts the orphaned child and calls `wait()` on it. (orphan process)
- If child dies first -> becomes a zombie until parent calls `wait()`.

**Kill process:**
- `kill PID` — send SIGTERM (graceful shutdown)
- `kill -9 PID` — send SIGKILL (force kill)
- `pkill name` — kill by name

**Job Control:**
- `&` — run command in background
- `jobs` — list background jobs
- `fg %n` — bring job n to foreground
- `bg %n` — resume stopped job n in background

- If a background process needs input, it can free or terminate. Also, if it tries to write to terminal, it can appear in terminal with no regard. (redirect its output to avoid).

**Notes:**
- Higher niceness = more polite process (less CPU priority).  
- Lower niceness = more aggressive process (more CPU priority).  
- Only **root** can set negative niceness (increase priority).  

**The `init` process:**
- The original ancestor.
- PID 1, started by the kernel during boot, only terminated on shutdown/reboot.

**Signals:**
- A **signal** is a software interrupt sent to a process to notify it of an event.  
- Used for **inter-process communication (IPC)** and **process control**.  

**Common uses:**
- **User interaction:**  
  - `Ctrl + C` → `SIGINT` (interrupt/terminate)  
  - `Ctrl + Z` → `SIGTSTP` (suspend)  
- **Kernel notifications:** e.g. `SIGSEGV` for invalid memory access.  
- **Process management:** `kill -15 PID` (SIGTERM), `kill -9 PID` (SIGKILL).  

**Signal handling:**
- **Ignore** — discard the signal.  
- **Catch** — execute a custom handler.  
- **Block** — delay signal delivery.  
- **Default action** — perform predefined behavior (often terminate).  

**Common signals:**
- `SIGHUP (1)` — Hangup, daemons reload configs.  
- `SIGINT (2)` — Interrupt (`Ctrl + C`).  
- `SIGKILL (9)` — Force kill, cannot be caught or ignored.  
- `SIGSEGV (11)` — Segmentation fault.  
- `SIGTERM (15)` — Polite termination, can be handled.  
- `SIGSTOP` — Stop/pause, cannot be caught or ignored.  

**SIGTERM vs SIGKILL:**
- `SIGTERM (15)` — Graceful request; process may clean up and exit.  
- `SIGKILL (9)` — Immediate termination; kernel kills the process directly.  

**Notes:**
- Signals are **asynchronous** notifications between kernel and processes.  
- Each signal has a **default action** but can often be **handled or ignored**.  
- Always try `SIGTERM` first; use `SIGKILL` only when necessary.  

**`/proc` Filesystem:**
- A **virtual filesystem** created by the **kernel in memory** (not stored on disk).  
- Reflects the **live state** of the system — updated dynamically.  
- Implements the Linux principle: **“everything is a file.”**  

**Exploring `/proc`:**
- `ls /proc` → lists numbered directories (each = **PID** of a running process).  
- Common files:  
  - `/proc/cpuinfo` — CPU details  
  - `/proc/meminfo` — memory usage  
  - `/proc/uptime`, `/proc/loadavg`, `/proc/version` — system info  
- `/proc` acts as the **data source** for system monitoring utilities.
- Provides **real-time kernel and process info**.  
- Enables **custom monitoring or debugging scripts**.   

___
## Packages
- A **package** is a **bundle of software** (executables, configs, libraries, metadata) that the system can install, update, or remove in a controlled way.  
- Every Linux program — from Chrome to the kernel — is a package. 
- **Package Manager:** Tool that installs/removes packages (`apt`, `dnf`, `yum`, `brew`).  
- **Software:** The actual application (e.g., Chrome, VS Code).  
- **Package:** The container that delivers the software.  
- **OS (Linux Distribution):** A complete collection of packages (kernel + tools + apps).  
```bash
tar -xzvf package.tar.gz
```
use tar to extract files from a tarball (package).

**Package Repositories:**  
- A **package repository** is a central, online storage location for Linux software packages.  
- Package managers (`apt`, `dnf`, etc.) connect to these repositories automatically instead of requiring manual downloads.  
- Ensure secure, organized, and up-to-date software distribution.  
- Each repository is hosted on a **server** that stores signed packages.
- The package manager use repo's URL to find and download packages.

**`rpm` and `dpkg`(low level):**
- Like installing a .exe in Windows -> don’t automatically check dependencies or updates.
```bash
# Debian
sudo dpkg -i package.deb
# Red Hat
sudo rpm -i package.rpm
-i = install
-r = remove (Debian)
-e = erase (Red Hat)
-l = list, q = query, a = all
```

**`apt` and `yum`(high level):**
- Find and download from repositories, automatically handle dependencies.
- Built on top of low-level tools:
  - `apt` → uses `dpkg`
  - `yum` → uses `rpm`

**Compile source code:**
- Before you can compile anything, your system needs build tools:
```bash
# Debian
sudo apt install build-essential
# Red Hat
sudo yum groupinstall "Development Tools"
```
- 3 step to compile:
  1. `./configure` — prepare the build system
  2. `make` — compile the source code
  3. `sudo make install` — install the compiled program
- Use `checkinstall` instead of `make install` so the package manager can track it -> always prefer.