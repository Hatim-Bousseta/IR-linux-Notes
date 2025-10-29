### Ⅰ. Process Fundamentals 

* **Program vs. Process:** A **program** is an executable file; a **process** is the program **executed** by the OS.
* **PID:** Every active process gets a unique **Process Identifier** (PID). PIDs are recycled after termination.
    * **Max PID:** Check system limit: `cat /proc/sys/kernel/pid_max`
* **Init/systemd:** The **first process** created by the kernel. Always has **PID 1** and no parent.
* **Creation:** New processes are created via **`fork()`** (clones the parent) followed by **`exec()`** (replaces the cloned code with a new program).
* **Process Tree:** All processes (except PID 1) have a **Parent Process (PPID)**. Tracing the tree helps understand the attacker's path (e.g., examining a shell spawned by `sshd`).

| State | Meaning |
| :--- | :--- |
| **Running** | Running or ready to run. |
| **Waiting** | Waiting for an event/resource. |
| **Stopped** | Suspended. |
| **Zombie** | **Finished** but the **parent hasn't cleaned up** its record (`task_struct`) via `wait()`/`waitpid()`. Stays in the process table. |

---

### Ⅱ. Listing and Viewing Processes

Use these tools to triage processes based on suspicious activity (high CPU, unusual names, unknown PPIDs).

| Tool | Purpose | Key Commands & Parameters |
| :--- | :--- | :--- |
| **`ps`** | Static snapshot of running processes. | `ps aux` (All user processes, with user and terminal) |
| | Show **Process Tree** (critical for IR). | `ps aux --forest` |
| | List by **PPID** or **User**. | `ps --ppid PPID` / `ps -u username` |
| **`pstree`** | Dedicated process tree visualization. | `pstree` |
| **`top`** | **Real-time** process monitoring. | Run `top`, then hit **`P`** (sort by CPU), **`N`** (sort by PID), **`C`** (show full command path). |

---

### Ⅲ. Detailed Analysis (`/proc` Filesystem)

The `/proc` directory is a **virtual filesystem** created by the kernel. Every process has its own subdirectory: `/proc/PID/`.

| File/Location | Data Retrieved | IR Value |
| :--- | :--- | :--- |
| **`/proc/PID/cmdline`** | **Full command line** used to start the process. | *CRITICAL:* Reveals parameters, hidden scripts, or encoded commands. |
| **`/proc/PID/status`** | Process status, PID, **PPID**, UID/GID. | Maps the process to the Process Tree. |
| **`/proc/PID/exe`** | Symbolic link to the original **binary** file location. | Reveals if the executable is from a suspicious location (e.g., `/tmp/reverse`). |
| **`/proc/PID/environ`** | **Environment variables** in effect. | May contain dropped keys, paths, or malicious variables. |

#### **Example Lookup**

1. Find target PID: `ps aux | grep suspicious_process`
2. Read command line: `cat /proc/PID/cmdline`

### Ⅳ. Binary Triage and Reputation

If a suspicious process is identified, analyze the executable file (binary).

| Action | Command | Purpose |
| :--- | :--- | :--- |
| **Get Binary Path** | Check `/proc/PID/exe` (symlink) | Determine if the file is in a high-risk location like `/tmp` or a user directory. |
| **Calculate Hash** | `md5sum /path/to/binary` or `sha256sum /path/to/binary` | Use hash for reputation checks (VirusTotal, abuse databases). |

### Ⅴ. Eradication (Termination)

End malicious processes using their PID or name.

| Tool | Usage | Purpose |
| :--- | :--- | :--- |
| **`kill`** | `kill -9 PID` (SIGKILL) | Sends a signal to immediately terminate a single process by its **PID**. |
| **`killall`** | `killall PROCESS_NAME` | Terminates **all** running processes that share the specified **name** (useful for mass malware cleanup). 

