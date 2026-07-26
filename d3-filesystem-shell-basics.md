# Linux Fundamentals — Filesystem, Shell & Essential Commands

---

## 1. Terminal vs Shell vs Bash vs Zsh

These four words get used interchangeably a lot, but they're distinct layers:

| Term | What it actually is |
| --- | --- |
| **Terminal** | The program/window that gives you a text-based interface to type into — just the display/input layer (e.g. GNOME Terminal, iTerm2, or the console screen itself). It doesn't interpret commands; it just shows text. |
| **Shell** | The program that actually reads and interprets the commands you type, and talks to the operating system on your behalf. The terminal is just the window it runs inside. |
| **Bash** | One specific *implementation* of a shell — "Bourne Again SHell." It's the default shell on most Linux distributions, including Ubuntu Server. |
| **Zsh** | Another shell implementation — "Z Shell." |

So the relationship is: **Terminal (window) → runs a Shell (interpreter) → which might be Bash, Zsh, or something else.**

### Is Zsh different from Bash? If so, how?

Yes, though they're both POSIX-compatible shells and share most basic syntax. Key differences:

- **Autocompletion** — Zsh's tab-completion is generally more powerful and configurable out of the box (e.g. completes flags, suggests corrections for typos).
- **Plugins & frameworks** — Zsh has a large ecosystem (notably **Oh My Zsh**) for themes, plugins, and Git-branch-aware prompts, which Bash doesn't have natively.
- **Globbing (wildcard matching)** — Zsh's pattern matching for filenames is more advanced by default.
- **Scripting compatibility** — Bash is more universal for scripts (`#!/bin/bash`) since it's the default almost everywhere, including servers; Zsh scripts are less portable across systems.

**Practical takeaway for DevOps:** Bash is what you'll use on servers and in scripts almost all the time — it's the safe, portable default. Zsh is more of a personal daily-use preference (common on macOS, where it's now default), but production scripting should still target Bash unless there's a specific reason not to.

---

## 2. Is Ubuntu Server Fundamentally Different from Ubuntu Desktop?

Not fundamentally — they share the same underlying kernel and package base. The real differences:

- **No GUI** — Server edition ships without a desktop environment (no GNOME, no windowing system) by default. Everything is done via the command line.
- **Installed packages** — Server comes with server-oriented packages (SSH server, etc.) and skips desktop-oriented ones (browsers, media players, GUI apps).
- **Resource footprint** — Server uses less RAM/CPU/disk since it's not running a graphical environment.
- **Intended use case** — Desktop is built for interactive personal use; Server is built to be installed once and then managed remotely (via SSH) indefinitely, often headless (no monitor attached at all).

You *can* install a desktop environment on top of Ubuntu Server, and conversely strip a Desktop install down close to server-like — but the defaults and philosophy differ: Server assumes you'll drive it entirely through the terminal.

---

## 3. How Do Windows and Linux Differ?

| Aspect | Windows | Linux |
| --- | --- | --- |
| **Source model** | Closed source (proprietary) | Open source (kernel and most tooling) |
| **Filesystem structure** | Drive letters (`C:\`, `D:\`), separate trees per drive | Single unified tree starting at `/`, everything mounted under it |
| **Philosophy** | GUI-first, historically | "Everything is a file," CLI-first, scriptable by design |
| **Permissions model** | ACL-based (Access Control Lists), somewhat more complex/GUI-driven | User/Group/Other + read/write/execute bits — simpler, terminal-native |
| **Package management** | Installers/.exe/.msi, more recently `winget` | Native package managers (`apt`, `yum`, `dnf`) with dependency resolution |
| **Server usage** | Windows Server exists but far less common for web/cloud infra | Dominant in servers, cloud, and containers — most of the internet's infrastructure runs Linux |
| **Cost/licensing** | Licensed, often paid | Free and open source (most distributions) |

**Why this matters for DevOps specifically:** the overwhelming majority of servers, cloud infrastructure, and container base images are Linux — which is why this course is built entirely around it.

---

## 4. Filesystem Overview

### `ls /` — the Root Directory

Running `ls /` lists the top-level folders of the entire filesystem. Unlike Windows, Linux has **one single tree**, starting at `/` (the root) — there's no `C:\`/`D:\` split. Every drive, partition, or device gets *mounted* somewhere inside this one tree.

### "Everything is a file"

This is a core Linux philosophy: not just documents and photos, but devices (`/dev`), running processes (`/proc`), and even sockets/pipes are represented and interacted with as files. This uniformity is what lets the same basic tools (`cat`, `ls`, redirection with `>`) work across wildly different kinds of things.

### Binary files

A **binary file** contains data not meant to be read as plain text (unlike a `.txt` or `.md` file) — machine code, compiled programs, images, etc. Most commands you run (`ls`, `cat`, `cp`) are themselves binary executable files sitting in folders like `/bin` or `/usr/bin`.

### `ls -la`

Breaking down the flags:

- `-l` = **long format** — shows permissions, owner, group, size, modified date
- `-a` = **all** — includes hidden files (anything starting with `.`, like `.bashrc`)
- Combined: `ls -la` shows a detailed listing of everything in a directory, hidden files included.

### Key top-level folders (`ls /`)

| Folder | What it is | Notes |
| --- | --- | --- |
| `/bin` | **Essential user binaries** | Core commands needed even in single-user/recovery mode (`ls`, `cp`, `mv`, etc.) |
| `/boot` | **Bootloader files** | The kernel image and GRUB (bootloader) config — what actually gets the system starting before the rest of the OS even loads. |
| `/dev` | **Device files** | Represents hardware as files — e.g. `/dev/sda` (a disk), `/dev/null` (a "black hole" device). This is where "everything is a file" becomes very literal — even your hard drive is a file here. |
| `/etc` | System-wide **configuration files** | Contains things like network config, DNS settings (`/etc/resolv.conf`), hostname, and the user database (`/etc/passwd`) — covered more below. No IP addresses are "in" `/etc` by default, but DNS resolver settings and static network configs live here. |
| `/home` | **User home directories** | Each regular (non-root) user gets a folder here, e.g. `/home/chitti`. Root's home directory is deliberately *not* here — see below. |
| `/lib`, `/lib64` | **Shared libraries** | Code that binaries depend on to run (similar to `.dll` files on Windows). `/lib64` specifically holds 64-bit libraries; `/lib` traditionally held 32-bit or architecture-generic ones. On most modern 64-bit systems, `/lib` is symlinked to `/lib64` (or vice versa) since 32-bit support is increasingly dropped. |
| `/media` | **Auto-mounted removable media** | Where USB drives, CDs, etc. get automatically mounted when plugged in. |
| `/mnt` | **Manual mount point** | Conventional location for temporarily mounting a filesystem by hand (as opposed to `/media`'s automatic behavior). |
| `/opt` | **Optional/third-party software** | Self-contained packages that don't follow the standard `/usr` layout — often used by larger third-party applications that bundle their own dependencies. |
| `/proc` | **Virtual filesystem for process/kernel info** | Doesn't hold real files on disk — it's generated live by the kernel. Each running process gets a folder named after its **PID** (Process ID), e.g. `/proc/1234/`, containing live info about that process. |
| `/root` | **Root user's home directory** | Deliberately kept outside `/home` — see the dedicated section below for why. |
| `/run` | Runtime data since last boot | Holds volatile runtime state, cleared on every reboot (unlike `/tmp`, which may persist depending on config). |
| `/sbin` | **System binaries** | Admin/root-level commands (e.g. tools for network interfaces, disk management) — historically separate from `/bin` because these are typically only run by root. Commands affecting things like network interfaces (which *can* include turning WiFi off via `ip link set` or similar) live here. On modern Ubuntu, `/bin` and `/sbin` are often symlinked into `/usr/bin` and `/usr/sbin`, but the conceptual split remains. |
| `/snap` | **Snap packages** | Ubuntu's self-contained package format (sandboxed apps with their own dependencies bundled in) — an alternative to traditional `apt` packages. |
| `/srv` | **Service data** | Data served by the system for services like FTP or web servers — e.g. an FTP server's files might live under `/srv/ftp`. Not heavily used on every system, but conventionally the place for service-hosted data. |
| `/sys` | **Virtual filesystem for kernel/hardware info** | Similar spirit to `/proc` but more structured — used by tools that need to query or tune kernel devices and hardware parameters directly. |
| `/tmp` | **Temporary files** | World-writable — any user can create files here, and its contents are typically cleared on reboot. Useful for short-lived scratch files, but not for anything you need to persist. |
| `/usr` | **User programs & libraries** | Despite the plain name, this is typically the **largest** directory on the system — most installed programs, their libraries, and documentation live under here (`/usr/bin`, `/usr/lib`, `/usr/share`, etc.). Easy to underestimate at a glance. |
| `/var` | **Variable data** | Data that changes frequently while the system runs — logs (`/var/log`), caches, mail queues, and often database files. |

### CPU, hard drive, NVIDIA driver — how do these relate to `/dev`?

- Hardware devices show up as files under `/dev` — e.g. `/dev/sda` for a disk, `/dev/nvidia0` if an NVIDIA driver/GPU is present.
- The **CPU** itself isn't a "file" in `/dev` in the same way — CPU info is exposed instead through `/proc/cpuinfo` (a virtual file showing live CPU details: cores, model, speed).
- **How critical are the files listed in `ls /`?** Extremely — this is core OS structure. Deleting or misconfiguring folders like `/etc`, `/bin`, `/lib`, or `/dev` can render a system unbootable. This is exactly why root-level changes are dangerous and require `sudo` — a safeguard against casual/accidental damage to these directories.

---

## 5. Users, Home Directories & Permissions Basics

### `whoami`

Prints the currently logged-in username. Quick sanity check for "who am I in this shell session."

### The Login Screen — Reading `Ubuntu 26.04 LTS ubuntu-server tty1` / `ubuntu-server login:`

Breaking this down line by line:

- **`Ubuntu 26.04 LTS`** — the OS name and version (26.04 = year.month release convention; LTS = Long Term Support release).
- **`ubuntu-server`** — the **hostname** of this machine (the name you gave it during setup).
- **`tty1`** — the specific **terminal/console** you're logged into. `tty` stands for "teletypewriter" (a historical term from physical terminal hardware) — `tty1` is the first virtual console. Multiple `tty`s (tty1–tty6 typically) let multiple local login sessions exist on the same machine.
- **`ubuntu-server login:`** — the actual login prompt, waiting for a username (then password).

### `~` vs `/`

- **`/`** — the root of the *entire* filesystem.
- **`~`** — shorthand for the **current user's home directory**. It's not a fixed path — it expands differently per user.
- **Is `~` equivalent to `/home/username`?** — Yes, for regular users. `~` is literally an alias that expands to `/home/<your-username>`. For the root user specifically, `~` expands to `/root` instead (root's home directory is not under `/home`).

### Root, Root User, Custom Users

- **"Root" (the directory `/`)** and **"root" (the user)** are two different things sharing a name — easy to confuse.
- **Root user** — the superuser/administrator account, with unrestricted access to the entire system. UID (user ID) `0`.
- **Custom user** — any regular account created afterward (like the one you set up during Ubuntu install) — restricted permissions by default, needs `sudo` to perform admin actions.

### Why is root's home directory (`/root`) outside of `/home`?

Two real, practical reasons:

1. **Boot/recovery independence** — In many real server setups, `/home` is mounted as a **separate partition or even a separate disk/network volume**. If that partition fails to mount (disk issue, fstab misconfiguration, network storage down), a system with `/root` *inside* `/home` would leave the root user unable to log in or fix anything. Keeping `/root` on the main root filesystem (`/`) guarantees the superuser can always get in to repair the system, even when everything else is broken.
2. **Privilege separation by convention** — root is not "just another user" — it's the account with unrestricted system access. Keeping its home directory structurally separate from ordinary user accounts reinforces that root operates at a different tier, and avoids accidentally bundling root's config/files into whatever policies (quotas, permissions, backups) apply to `/home`.

### Run Levels (in detail)

Run levels are a traditional way of describing **what state/mode the system is running in** — historically used in SysV-init systems.

| Run Level | Meaning |
| --- | --- |
| 0 | Halt (shutdown) |
| 1 | Single-user mode (rescue/maintenance, minimal services, usually root only, no networking) |
| 2 | Multi-user mode, no networking (Debian/Ubuntu specific — varies by distro) |
| 3 | Multi-user mode, with networking, no GUI (**typical for servers**) |
| 4 | Undefined/custom (rarely used) |
| 5 | Multi-user mode with GUI (typical for Desktop) |
| 6 | Reboot |

**Why this is largely legacy now:** modern Ubuntu (and most major distros) use **systemd** instead of classic SysV init. Systemd replaces numbered run levels with named **targets**, but keeps backward-compatible aliases so the old numbers still resolve:

| Old run level | systemd target |
| --- | --- |
| 0 | `poweroff.target` |
| 1 | `rescue.target` |
| 3 | `multi-user.target` |
| 5 | `graphical.target` |
| 6 | `reboot.target` |

Check the current target with:

```bash
systemctl get-default
```

On Ubuntu Server, this returns `multi-user.target` — the systemd equivalent of "run level 3," confirming no GUI is expected.

**Why this matters:** Ubuntu Server effectively lives at `multi-user.target` (multi-user, networked, no GUI) by design — consistent with the Server vs Desktop distinction covered above. It's also a useful sanity check on a real server: if it somehow boots into `rescue.target` or a graphical target unexpectedly, that's a sign something's misconfigured.

### What is `grep`?

`grep` is a command that **searches text for lines matching a pattern** and prints only the matching lines. Think of it as "find me every line containing this word."

Basic usage: `grep "pattern" filename`

Example:

```bash
grep "hello" myfile.txt
```

This prints every line in `myfile.txt` that contains the word "hello," and ignores every line that doesn't.

`grep` is almost always used together with a **pipe** (`|`). A pipe takes the output of one command and feeds it as input into the next command, instead of printing it to the screen. So `command1 | grep "pattern"` means: "run `command1`, then only show me the lines from its output that match `pattern`."

### `/etc/passwd` — What a Line Actually Looks Like

Running `cat /etc/passwd` prints the whole file, one line per user account. A single line looks like this:

```text
chitti:x:1000:1000:Chitti,,,:/home/chitti:/bin/bash
```

This is **7 fields separated by colons (`:`)**. Breaking it down field by field, left to right:

| Field # | Value in example | What it means |
| --- | --- | --- |
| 1 | `chitti` | **Username** — the login name |
| 2 | `x` | **Password placeholder** — always shows `x` here; the *real* encrypted password is stored separately in `/etc/shadow` (a file only root can read), never in `/etc/passwd` itself, for security |
| 3 | `1000` | **UID (User ID)** — a unique number identifying this user. `0` is always root. Regular human users on Ubuntu typically start at `1000` |
| 4 | `1000` | **GID (Group ID)** — the user's primary group ID |
| 5 | `Chitti,,,` | **GECOS field** — usually just the user's full name (the commas separate other optional info like room number/phone, rarely used) |
| 6 | `/home/chitti` | **Home directory** — where this user lands when they log in, and what `~` expands to for them |
| 7 | `/bin/bash` | **Login shell** — the program that runs when this user logs in. Normally a real shell like `/bin/bash`, but for service accounts this is often `/usr/sbin/nologin` instead (covered below) |

So when you run:

```bash
cat /etc/passwd
```

You get the *entire* list — every user account on the system, one line each, in this same 7-field format.

When you instead run:

```bash
cat /etc/passwd | grep leapfrog
```

Here's exactly what happens, step by step:

1. `cat /etc/passwd` prints every line of the file (every user account).
2. The `|` (pipe) takes that entire output and hands it to `grep` instead of showing it on screen.
3. `grep leapfrog` filters through all those lines and prints **only** the ones containing the text "leapfrog."

So if there's a user account named `leapfrog` on the system, running this would show just its one line — e.g.:

```text
leapfrog:x:1001:1001:Leapfrog User:/home/leapfrog:/bin/bash
```

— instead of scrolling through every other user account to find it manually.

### "Nologin" Users, `/bin/false`, `/sbin/nologin`

Some entries in `/etc/passwd` are **service/system accounts**, not real human logins — created automatically for things like mail delivery or system daemons (background services). Their line looks like this:

```text
sshd:x:105:65534::/run/sshd:/usr/sbin/nologin
```

Same 7 fields as before — the important difference is the last field: instead of `/bin/bash`, it says `/usr/sbin/nologin`.

- **`/sbin/nologin`** (or `/usr/sbin/nologin`) — if someone tries to log in as this user, it prints a polite message ("This account is currently not available.") and immediately exits, refusing the login.
- **`/bin/false`** — a similar effect, but more blunt — it's a tiny program that does nothing and immediately reports "false" (failure), silently declining the login with no message at all.

**Why this matters:** accounts like `sshd` above exist purely so a background service can run under its own restricted identity (rather than as root), for security separation. They were never meant for a human to actually log into — setting the shell to a "nologin" program is what enforces that, even if someone tried.

### `useradd`

`useradd test` creates a new user account named `test`. Notes:

- On its own, `useradd` is fairly bare-bones — it often doesn't create a home directory or set a password by default (that's what the friendlier `adduser` wrapper does on Ubuntu, which is interactive and more beginner-friendly).
- After creating a user, you'd typically still need to set a password (`passwd test`) before that account is usable for login.

### `sudo su` and `$` vs `#`

- **`sudo su`** — runs `su` (switch user) with elevated privileges via `sudo`, effectively dropping you into a full **root shell**. Different from `sudo <command>` (which runs just one command as root) — `sudo su` gives you a persistent root session until you exit.
- **`$` vs `#` prompt** — a shell convention, not a command:
  - **`$`** — you're logged in as a regular (non-root) user.
  - **`#`** — you're logged in as **root**. This is a strong visual warning: anything typed here runs with full system privileges, no safety net.

---

## 6. Essential Commands

| Command | What it does |
| --- | --- |
| `pwd` | **P**rint **w**orking **d**irectory — shows your current location in the filesystem. Good habit: run this as a "sanity check" whenever you're unsure where you are before running something destructive. |
| `ls` | List directory contents |
| `cd` | Change directory |
| `mkdir` | Make a new directory |
| `rmdir` | Remove an **empty** directory only — fails if it has contents |
| `rm` | Remove files (or directories, with flags) |
| `mv` | Move (or rename) a file/directory |
| `cp` | Copy a file/directory |
| `man` | Manual — shows the full documentation/help page for a command (e.g. `man ls`) |

### `cd -`

Takes you back to the **previous** directory you were in before your last `cd` — a quick way to toggle between two locations without typing the full path again.

### The `-p` flag

Context-dependent, but common uses:

- `mkdir -p some/nested/path` — creates all intermediate directories as needed, without erroring if some already exist. Without `-p`, `mkdir nested/path` fails if `nested` doesn't already exist.
- (Other commands have their own `-p` meanings — e.g. `cp -p` preserves file attributes like timestamps — but the "create parent directories as needed" meaning under `mkdir` is the one most commonly used early on.)

### `rm -rf`

Breaking down the flags:

- **`-r`** — recursive; deletes directories and everything inside them, not just a single file.
- **`-f`** — force; skips confirmation prompts and ignores nonexistent files silently.

**This is one of the most dangerous commands in Linux.** Combined, `rm -rf` will silently and irreversibly delete an entire directory tree with no confirmation and no recycle bin. A classic catastrophic mistake is running it with the wrong path (or an accidental leading `/`), which can wipe out the entire filesystem. Always double-check the target path (`pwd` first!) before running it.

### Filenames with Spaces or Special Characters

Linux allows spaces and most special characters in filenames, but the shell treats spaces as **argument separators** by default, which causes problems:

```bash
# This is interpreted as TWO arguments: "my" and "file.txt"
rm my file.txt
```

To handle this correctly:

- **Quote the filename:** `rm "my file.txt"`
- **Escape the space:** `rm my\ file.txt`
- **Tab-completion** in the terminal automatically adds the necessary escaping/quoting for you — generally the safest approach in practice.

Special characters (like `$`, `*`, `&`, `(`, `)`) can carry specific meaning to the shell (wildcards, variables, backgrounding, etc.), so filenames containing them should almost always be quoted to avoid the shell misinterpreting them.
