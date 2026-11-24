# Linux & Git Administration Learning Log — 24/11/2026

A structured summary of hands-on labs and Linux system administration concepts covered on **24/11/2026**.

## Table of Contents
- [Hands-On Labs](#hands-on-labs)
  - [Lab 1: Implementing Semantic Versioning Principles](#lab-1-implementing-semantic-versioning-principles)
  - [Lab 2: Use Basic Commands for the Linux Command Line](#lab-2-use-basic-commands-for-the-linux-command-line)
- [System Management and File Security](#system-management-and-file-security)
  - [1. Managing Users and Groups](#1-managing-users-and-groups)
  - [2. Object Permissions and Ownership](#2-object-permissions-and-ownership)
  - [3. Offboarding and Removing User Accounts](#3-offboarding-and-removing-user-accounts)
  - [4. Managing System Processes and Resource Health](#4-managing-system-processes-and-resource-health)
  - [5. System Logging (`rsyslog` & `journald`)](#5-system-logging-rsyslog--journald)

---

## Hands-On Labs

### Lab 1: Implementing Semantic Versioning Principles

#### Concepts & Workflow Practiced
* **Repository Initialization:** Configured global Git user authorship (`user.name` and `user.email`) and initialized a versioned shell utility project (`globomantics-resource-utils`).
* **Semantic Versioning (`MAJOR.MINOR.PATCH`):** Tracked software maturity using a `VERSION` file and documented every release in `CHANGELOG.md`:
  * **`0.0.0` (Initial Development):** Signals the library is unstable and its public interface may change at any time.
  * **`1.0.0` (Initial Stable Release):** Marks the first stable public API (`list_resources`, `resource_count`).
  * **`1.0.1` (Patch Release):** Increments `PATCH` for backward-compatible bug fixes (changing `wc -l` to `wc -w` in `resource_count`) while keeping the public interface unchanged.
  * **`1.1.0` (Minor Release):** Increments `MINOR` and resets `PATCH` to `0` when adding backward-compatible functionality (`resource_filter`).
  * **`2.0.0` (Major Release):** Increments `MAJOR` and resets `MINOR` and `PATCH` to `0` for breaking changes (renaming `list_resources` to `get_resources` and requiring an environment argument).
* **Annotated Release Tagging:** Created self-documenting annotated tags (storing tagger, date, and release message) using the standard `v` prefix.

#### Commands Practiced

| Category | Commands |
| :--- | :--- |
| **Git Setup & Tracking** | `git --version`, `git config --global`, `git init`, `git add`, `git commit`, `git diff` |
| **Git Tagging & History** | `git tag -a <tag> -m "<msg>"`, `git tag -l`, `git show <tag>`, `git describe --tags` |
| **Shell & File Utilities** | `mkdir`, `cd`, `cat <<'EOF'` (literal here-documents), `echo`, `wc -l`, `wc -w`, `tr`, `grep`, `bash -c` |

---

### Lab 2: Use Basic Commands for the Linux Command Line

#### Concepts & Workflow Practiced
* **Built-in Help & Paging:** Used `--help`, `man` pages, and `info` documentation alongside `less` to navigate command manuals and inspect flags (`-l` long listing, `-a` hidden dotfiles, `-h` human-readable sizes).
* **Filesystem Indexing & Search:** Installed `plocate`, searched the file inventory with `locate`, and manually refreshed the database with `sudo updatedb` to index newly created files.
* **Text Filtering & Inspection:** Searched recursively for strings across directories using `grep` and inspected the top and bottom 10 lines of configuration files (`.bashrc`) with `head` and `tail`.
* **Network Inspection:** Queried Ethernet NIC IPv4/IPv6 configurations and checked active listening TCP sockets and associated PIDs.
* **Bash Shell Scripting:** Created a script in `nano` with a `#!/bin/bash` shebang, granted execution rights using `chmod +x`, and executed it from the terminal.

#### Commands Practiced

| Category | Commands |
| :--- | :--- |
| **Help & Pagers** | `<cmd> --help`, `man`, `info`, `less` |
| **Files & Text Search** | `ls -lah`, `touch`, `sudo apt install`, `locate`, `sudo updatedb`, `grep -r`, `cat`, `head`, `tail` |
| **Networking** | `ip help`, `ip addr`, `netstat -nltp` |
| **Scripting & Permissions** | `nano`, `#!/bin/bash`, `chmod +x`, `./<script>.sh` |

---

## System Management and File Security

### 1. Managing Users and Groups
* **Authentication & Accountability:** Enforcing individual user accounts ensures credential verification and allows administrators to attribute system events or security incidents to specific accounts.
* **Red Hat vs. Ubuntu Workflows:**
  * *Red Hat / Fedora:* Uses `useradd -m` (to create a `/home` directory) and `-s` (to specify a login shell), followed by `passwd` to activate the account with a temporary password.
  * *Ubuntu / Debian:* Provides the interactive `adduser` and `addgroup` utilities, which automatically create home directories, copy skeleton files, and prompt for passwords.
* **Groups & Privilege Escalation:** Managed multi-user access by creating groups and appending users with `usermod -aG` (including adding users to the `sudo` group for administrative privileges).
* **System Files & User Templates (`/etc/skel`):**
  * `/etc/passwd` stores user metadata (UID, home directory, shell), `/etc/shadow` stores hashed passwords, and `/etc/group` tracks group memberships.
  * `/etc/skel` acts as a skeleton template—files like `.bashrc` (non-login shell settings and command aliases) and `.profile` (login shell settings) placed here are automatically copied into every new user's home directory.
* **Commands:** `useradd`, `adduser`, `passwd`, `groupadd`, `addgroup`, `usermod -aG`, `groups`, `id`, `su`.

### 2. Object Permissions and Ownership
* **Permission Notation Systems:**
  * Permissions apply to three principals: **User/Owner (`u`)**, **Group (`g`)**, and **Others (`o`)**.
  * **Symbolic (`rwx`) vs. Octal (`4`, `2`, `1`):** Read (`r=4`), Write (`w=2`), and Execute (`x=1`) combine into octal digits (e.g., `771`, `770`, `750`, `640`).
  * **Directory Execution (`x`):** On a directory, the execute bit allows users to traverse (`cd`) through it to access nested files.
* **Ownership Management:** Transferred file/directory ownership with `chown` (using `-R` for recursive directory updates) and group ownership with `chgrp`.
* **Special Access Bits:**
  * **Set-User-ID (`SUID`):** Allows normal users to execute a compiled binary (like `/usr/bin/passwd`) with the temporary privileges of the file owner (`root`), though Linux blocks SUID on shell scripts for security.
  * **Set-Group-ID (`SGID` — `g+s` or octal `2xxx`):** Ensures all new files and subdirectories created inside a shared directory inherit the parent directory's group ownership rather than the creator's primary group.
  * **Sticky Bit (`+t`):** Prevents users in a shared directory from deleting or renaming files owned by other users.
* **Commands:** `chmod`, `chown` (`-R`), `chgrp`, `which`.

### 3. Offboarding and Removing User Accounts
* **Secure Account Termination Workflow:**
  1. **Stop Active Processes:** Identify processes belonging to the departing user (`ps -u <user>`) and terminate them (`kill` / `pkill`).
  2. **Lock Account:** Disable logins immediately with `usermod -L` (which prepends `!` to the password hash in `/etc/shadow`).
  3. **Backup Data:** Archive the user's home directory into a compressed tarball using `tar -czf` and verify the archive.
  4. **Revoke Group Access & Delete:** Remove secondary group memberships with `gpasswd -d` and delete the account along with its home directory via `userdel -r`.
* **Commands:** `ps -u`, `usermod -L`, `tar -czf`, `gpasswd -d`, `userdel -r`.

### 4. Managing System Processes and Resource Health
* **Process Monitoring & System Health:**
  * `ps aux` and `ps -ef` list system-wide processes, resource percentages, and parent/child relationships, while `pstree -p` visualizes the process hierarchy under `systemd`.
  * `top` provides an interactive real-time view (sorting by CPU `P` or memory `M`, filtering by user `u`, viewing individual cores `1`, and killing PIDs `k`), alongside `watch` for periodic command execution.
  * `free -h` inspects RAM, cache, and swap usage, while `df -h` monitors filesystem disk space utilization.
* **Signals & Job Control:**
  * **Termination Signals:** `SIGTERM` (`kill -15`, default) requests a graceful shutdown, `SIGQUIT` (`kill -3`) interrupts a process, and `SIGKILL` (`kill -9`) forces immediate termination. `killall` terminates all instances of a program by name.
  * **Background Jobs:** Suspended foreground tasks with `Ctrl+Z`, inspected stopped/running tasks via `jobs`, resumed execution in the background with `bg %<job>`, and returned tasks to the foreground with `fg %<job>`.
* **Commands:** `ps` (`aux`, `-ef`, `--sort`), `pstree -p`, `top`, `watch`, `free -h`, `df -h`, `kill`, `kill -9`, `killall`, `pkill`, `jobs`, `bg`, `fg`.

### 5. System Logging (`rsyslog` & `journald`)
* **`rsyslog` (Plain-Text Logging & Forwarding):**
  * Uses **facility.priority** pairs (e.g., `auth`, `cron`, `kern`, `user`; priorities ranging from `debug` up to `emerg`) to route messages to `/var/log/syslog`, `/var/log/auth.log`, or custom files configured in `/etc/rsyslog.d/`.
  * Tested custom rules using the `logger -p` utility and monitored live log streams via `tail -f`.
  * Configured centralized **remote logging** over TCP port `514` (`@@<server-ip>:514` on clients, `imtcp` module and `/var/log/remote/` templates on the server) and verified listening sockets with `ss`.
* **`journald` (Indexed Binary Logging):**
  * Managed by `systemd-journald` and queried via `journalctl`.
  * Configured persistence (`Storage=persistent` to write to `/var/log/journal`), compression (`Compress=yes`), and rotation caps (`SystemMaxUse`, `SystemMaxFileSize`) in `/etc/systemd/journald.conf`.
  * Filtered binary logs by reverse chronological order (`-r`), boot session (`-b -1`), systemd unit (`-u`), priority level (`-p err`), live stream (`-f`), timeframe (`--since`), and disk footprint (`--disk-usage`).
