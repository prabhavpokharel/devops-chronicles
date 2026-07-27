# Linux Fundamentals — Filesystem, Shell & the Commands You'll Use Every Day

## Setting the Stage

By this point you have a running Ubuntu Server VM and SSH access into it. Today is about understanding the machine you're now sitting inside — how it's organized, who's allowed to do what, and the small set of commands you'll type hundreds of times over the rest of this course.

None of this is exciting on the surface. It's the equivalent of learning your way around a new city before you start building anything in it. But almost every mistake a beginner makes on Linux — deleting the wrong thing, running a command as the wrong user, getting lost in the wrong directory — traces back to skipping this exact foundation. Worth doing carefully once.

---

## Terminal, Shell, Bash, Zsh — Untangling Four Words That Get Used Interchangeably

These four terms get thrown around as if they mean the same thing. They don't.

- **Terminal** is just the window you type into. It displays text and takes input — nothing more. It has no idea what a command even is.
- **Shell** is the program that actually reads what you type, interprets it, and talks to the operating system on your behalf. The terminal is the window; the shell is what's running inside it.
- **Bash** ("Bourne Again SHell") is one specific implementation of a shell — the default on nearly every Linux distribution, including Ubuntu Server.
- **Zsh** ("Z Shell") is a different shell implementation.

So the chain is: a terminal window runs a shell, and that shell might be Bash, Zsh, or something else entirely.

### Is Zsh actually different from Bash?

Yes, though both are POSIX-compatible and share most of their basic syntax. The differences that matter:

- Zsh's tab-completion is more capable out of the box — completing flags, correcting typos, and generally being smarter about guessing what you meant.
- Zsh has a large plugin and theming ecosystem, most notably **Oh My Zsh**, giving you things like Git-aware prompts. Bash doesn't have this natively.
- Zsh's wildcard/pattern matching for filenames is more advanced by default.
- Bash is more universal for scripting — it's the default nearly everywhere, including every server you'll touch professionally. Zsh scripts are less portable across systems.

In practice: Bash is what you'll use on servers and in scripts almost all the time, because it's the safe, portable default. Zsh is more of a personal daily-driver preference (it's the default shell on macOS now), but production scripting should still target Bash unless there's a specific reason not to.

---

## Ubuntu Server vs Ubuntu Desktop

They're not fundamentally different systems — same kernel, same package base underneath. What actually differs:

- **No GUI.** Server ships without a desktop environment. Everything happens on the command line.
- **Different default packages.** Server includes things like an SSH server by default and skips desktop software entirely — no browsers, no media players.
- **Lighter footprint.** Without a graphical environment running, Server uses noticeably less RAM and CPU.
- **Different assumed use case.** Desktop is built for someone sitting in front of the machine. Server is built to be installed once and then managed remotely, indefinitely, often with no monitor ever attached to it at all.

You can technically install a desktop environment on Server, or strip Desktop down to something server-like, but the defaults reflect two different philosophies: Server assumes you'll drive it entirely through a terminal.

---

## Windows vs Linux

| | Windows | Linux |
| --- | --- | --- |
| Source model | Closed source | Open source (kernel and most tooling) |
| Filesystem | Drive letters (`C:\`, `D:\`), separate trees | One unified tree starting at `/`, everything mounted into it |
| Philosophy | Historically GUI-first | CLI-first, "everything is a file," built to be scripted |
| Permissions | ACL-based, more GUI-driven | User/Group/Other with read/write/execute bits — simpler, terminal-native |
| Package management | Installers (.exe/.msi), more recently `winget` | Native package managers (`apt`, `yum`, `dnf`) with dependency resolution |
| Server usage | Exists, but far less common for web/cloud infrastructure | Dominant — most of the internet's infrastructure runs on it |
| Licensing | Usually paid | Free and open source, in most distributions |

The practical reason this matters for you: nearly every server, cloud platform, and container base image you'll ever touch professionally runs Linux. That's the entire reason this course is built around it rather than Windows.

---

## Reading the Filesystem

### The root directory

Running `ls /` lists the top-level folders of the entire filesystem. Unlike Windows, Linux has exactly **one tree**, starting at `/`. There's no split between drives — every disk, partition, or device gets mounted somewhere inside this single structure.

### "Everything is a file"

This is one of Linux's defining ideas. Not just documents — devices (`/dev`), running processes (`/proc`), even sockets and pipes are all represented and interacted with as files. That uniformity is what lets the same small set of tools (`cat`, `ls`, output redirection with `>`) work across wildly different kinds of things.

### Binary files

A **binary file** holds data that isn't meant to be read as plain text — compiled machine code, images, executables. Most of the commands you run day to day (`ls`, `cat`, `cp`) are themselves binary files, sitting in directories like `/bin`.

### Reading `ls -la`

Two flags doing two separate jobs:

- `-l` — long format. Shows permissions, owner, group, size, and modification date.
- `-a` — all. Includes hidden files, meaning anything starting with a dot, like `.bashrc`.

Together, `ls -la` gives you a complete, detailed listing of a directory, hidden files included — the version you'll actually reach for most of the time.

### The top-level directories, one by one

| Directory | What it holds |
| --- | --- |
| `/bin` | Essential user commands needed even in a minimal recovery environment — `ls`, `cp`, `mv`, and similar. |
| `/boot` | The kernel image and bootloader (GRUB) configuration — what gets the system starting before the rest of the OS even loads. |
| `/dev` | Hardware represented as files. `/dev/sda` might be a disk, `/dev/null` is a "black hole" that discards anything written to it. If an NVIDIA driver is present, its device files show up here too. This is "everything is a file" taken completely literally — even a physical disk is a file you can point commands at. |
| `/etc` | System-wide configuration. Network settings, DNS resolver configuration (`/etc/resolv.conf`), the hostname, and the user account database (`/etc/passwd`) all live here. No IP addresses are stored here permanently by default, but static network config and DNS settings are. |
| `/home` | Home directories for regular, non-root users — each user gets their own folder here. |
| `/lib`, `/lib64` | Shared libraries — code that other programs depend on to run, roughly analogous to `.dll` files on Windows. `/lib64` holds 64-bit libraries specifically; `/lib` traditionally held 32-bit or architecture-generic ones. On most modern 64-bit systems, one is simply symlinked to the other. |
| `/media` | Where removable media — USB drives, CDs — gets automatically mounted when plugged in. |
| `/mnt` | The conventional location for manually mounting a filesystem, as opposed to `/media`'s automatic behavior. |
| `/opt` | Third-party, self-contained software that doesn't follow the standard layout the rest of the system uses. |
| `/proc` | A virtual filesystem, generated live by the kernel rather than stored on disk. Every running process gets a folder named after its PID (process ID) — `/proc/1234/`, for instance — containing live details about that process. |
| `/root` | The root user's home directory. Deliberately kept outside `/home` — worth understanding why, covered below. |
| `/run` | Runtime state since the last boot, cleared every time the system restarts. |
| `/sbin` | System-level binaries, historically reserved for root — tools for managing network interfaces or disks. Commands that touch things like network interfaces, including turning WiFi off entirely, live here. On modern Ubuntu, `/bin` and `/sbin` are often symlinked into `/usr/bin` and `/usr/sbin`, but the conceptual split still holds. |
| `/snap` | Ubuntu's self-contained package format — sandboxed applications bundled with their own dependencies, as an alternative to traditional `apt` packages. |
| `/srv` | Data served by services running on the machine — an FTP server's files, for instance, would conventionally live under `/srv/ftp`. Not heavily used on every system, but that's its intended purpose. |
| `/sys` | Another virtual filesystem, similar in spirit to `/proc` but more structured — used by tools that need to query or tune kernel and hardware parameters directly. |
| `/tmp` | Temporary storage, world-writable — any user on the system can create files here, and its contents are typically wiped on reboot. Fine for short-lived scratch files, not for anything you need to keep. |
| `/usr` | Despite the unassuming name, usually the largest directory on the system. Most installed programs, their libraries, and their documentation live here. |
| `/var` | Data that changes constantly while the system runs — logs (`/var/log`), caches, mail queues, often database files. |

A word on the CPU specifically, since it comes up naturally alongside `/dev`: the processor itself isn't represented as a device file the way a disk is. Its details are instead exposed through `/proc/cpuinfo` — a virtual file showing live information about cores, model, and clock speed.

And a word on how seriously to take all of this: these directories aren't decoration. They're the operating system itself. Deleting or misconfiguring `/etc`, `/bin`, `/lib`, or `/dev` can leave a system unable to boot. That's precisely why changes at this level require `sudo` — it's a deliberate speed bump against casual, accidental damage.

---

## Users, Login, and Home Directories

### `whoami`

Prints the username of whoever is currently logged into the shell. A quick, reflexive sanity check for "who am I right now."

### Decoding the login screen

The first thing you see when a fresh Ubuntu Server boots looks something like:

```text
Ubuntu 26.04 LTS ubuntu-server tty1
ubuntu-server login:
```

Each piece means something specific:

- **`Ubuntu 26.04 LTS`** — the OS name and version. The numbering follows a year.month convention; LTS marks it as a Long Term Support release.
- **`ubuntu-server`** — the hostname, the name given to this specific machine during setup.
- **`tty1`** — the specific console session you're on. "tty" is short for "teletypewriter," a holdover term from the era of physical terminal hardware. `tty1` is the first of typically six virtual consoles (`tty1`–`tty6`) that can exist simultaneously on one machine.
- **`ubuntu-server login:`** — the actual prompt, waiting for a username before it asks for a password.

### `~` vs `/`

`/` is the root of the entire filesystem. `~` is shorthand for the current user's home directory — it isn't a fixed path, it expands differently depending on who's logged in.

For a regular user, `~` is genuinely equivalent to `/home/<username>` — it's a direct alias for it. The one exception is root: for the root user, `~` expands to `/root` instead, which is not under `/home` at all.

### Root the directory, root the user

Two different things sharing a name, and it's an easy mix-up.

- **Root, the directory (`/`)** is the top of the filesystem tree.
- **Root, the user** is the superuser account — unrestricted access to the entire system, with a user ID of `0`.
- A **custom user** is any regular account created afterward, restricted by default, needing `sudo` to perform anything administrative.

### Why root's home directory sits outside `/home`

Two practical reasons, not just convention for its own sake:

1. **Recovery independence.** In real server setups, `/home` is often mounted as a separate partition, or even separate network storage. If that mount fails — a disk problem, a misconfiguration, storage being unreachable — a system where root's home lived inside `/home` would leave the one account capable of fixing things unable to log in at all. Keeping `/root` on the main filesystem guarantees root can always get in, even when everything else is broken.
2. **Deliberate separation.** Root isn't just another user account — it carries unrestricted system access. Keeping its home directory structurally apart from ordinary users' directories reflects that difference in privilege, and avoids accidentally folding root's files into whatever backup policies or quotas apply to `/home`.

### Run levels, properly explained

Run levels describe what state the system is booted into — a concept from the older SysV init system:

| Level | Meaning |
| --- | --- |
| 0 | Halt — shutdown |
| 1 | Single-user / rescue mode — root only, no networking, used for emergency repairs |
| 2 | Multi-user, no networking (handled slightly differently across distributions) |
| 3 | Multi-user, with networking, no GUI — the standard state for a server |
| 4 | Undefined, rarely used |
| 5 | Multi-user, with networking and GUI — the standard state for a desktop |
| 6 | Reboot |

Modern Ubuntu, like most current distributions, actually runs on **systemd** rather than classic SysV init. Systemd replaces numbered run levels with named targets, though it keeps the old numbers working as aliases:

| Old run level | systemd target |
| --- | --- |
| 0 | `poweroff.target` |
| 1 | `rescue.target` |
| 3 | `multi-user.target` |
| 5 | `graphical.target` |
| 6 | `reboot.target` |

You can check the current target directly:

```bash
systemctl get-default
```

On Ubuntu Server, this returns `multi-user.target` — the systemd equivalent of run level 3, confirming there's no graphical environment expected. It's a genuinely useful sanity check on a real machine: if a server somehow boots into `rescue.target` or a graphical target unexpectedly, that's a sign something is misconfigured.

### `/etc/passwd` — what a line actually looks like

`/etc/passwd` is the system's user account database — one line per account. A real line looks like this:

```text
chitti:x:1000:1000:Chitti,,,:/home/chitti:/bin/bash
```

Seven fields, separated by colons:

| Field | Value here | Meaning |
| --- | --- | --- |
| 1 | `chitti` | Username |
| 2 | `x` | A placeholder — the real, encrypted password lives separately in `/etc/shadow`, which only root can read. `/etc/passwd` itself is readable by everyone, so it never stores an actual password. |
| 3 | `1000` | UID — a unique numeric ID for this user. `0` is always root; regular human accounts on Ubuntu typically start at `1000`. |
| 4 | `1000` | GID — the user's primary group ID. |
| 5 | `Chitti,,,` | The GECOS field, generally just the user's full name — the extra commas separate other rarely-used optional fields like room number or phone. |
| 6 | `/home/chitti` | Home directory — where this user lands on login, and what `~` expands to for them. |
| 7 | `/bin/bash` | Login shell — the program that runs when they log in. For real human users this is a genuine shell; for service accounts, it's often deliberately set to something that refuses login entirely, covered next. |

### `grep`, and what actually happens with `cat /etc/passwd | grep leapfrog`

`grep` searches text for lines matching a pattern and prints only the matches. Given a file, `grep "pattern" filename` prints every line containing that pattern and silently skips the rest.

It's almost always paired with a **pipe** (`|`), which takes the output of one command and feeds it directly into the next, instead of printing it to the screen. So `command1 | grep "pattern"` reads as: run `command1`, then show me only the lines from its output that match `pattern`.

Walking through `cat /etc/passwd | grep leapfrog` step by step:

1. `cat /etc/passwd` prints every line in the file — every user account on the system.
2. The pipe hands that entire output to `grep` instead of displaying it directly.
3. `grep leapfrog` filters through those lines and prints only the ones containing the text "leapfrog."

If an account named `leapfrog` exists, the result is a single line, something like:

```text
leapfrog:x:1001:1001:Leapfrog User:/home/leapfrog:/bin/bash
```

— rather than scrolling through the entire account list by hand to find it.

### Accounts that can't log in

Not every line in `/etc/passwd` represents a human. Many are service accounts, created automatically for background processes — mail delivery, SSH itself, and similar. A line for one of these looks like:

```text
sshd:x:105:65534::/run/sshd:/usr/sbin/nologin
```

Same seven fields as before. The meaningful difference is the last one: instead of `/bin/bash`, it's `/usr/sbin/nologin`.

- **`/sbin/nologin`** (or `/usr/sbin/nologin`) — if anyone attempts to log in as this account, it prints a short message explaining the account isn't available, and exits immediately.
- **`/bin/false`** — does effectively the same job, more bluntly. It's a tiny program that does nothing at all and reports failure, declining the login with no message.

Accounts like `sshd` exist so a background service can run under its own restricted identity rather than as root — a security boundary. They were never meant for a human to log into interactively, and setting the shell to a "nologin" program is what enforces that even if someone tries.

### Creating a user, and what `sudo su` actually gives you

`useradd test` creates a new account named `test`. Worth knowing going in: on its own, `useradd` is fairly bare — it often doesn't create a home directory or set a password automatically. That's what the friendlier `adduser` command handles interactively on Ubuntu. After creating an account with `useradd`, you'd typically still need `passwd test` before it's actually usable.

`sudo su` runs `su` ("switch user") with elevated privileges, dropping you into a full root shell. It's different from running `sudo <command>` once — that only elevates a single command. `sudo su` gives you a persistent root session that lasts until you explicitly exit it.

And the prompt itself tells you which mode you're in — this is a shell convention, not a command:

- **`$`** means you're a regular user.
- **`#`** means you're root. It's a deliberate visual warning: anything typed at this prompt runs with full system privileges, with no safety net underneath it.

---

## The Commands You'll Reach for Constantly

| Command | What it does |
| --- | --- |
| `pwd` | Print working directory — shows exactly where you are in the filesystem. Worth running as a reflexive check before anything destructive. |
| `ls` | List a directory's contents. |
| `cd` | Change directory. |
| `mkdir` | Create a new directory. |
| `rmdir` | Remove a directory, but only if it's completely empty — it fails otherwise. |
| `rm` | Remove files, or directories with the right flags. |
| `mv` | Move a file or directory — also how you rename something, since renaming is just moving it to a new name in the same place. |
| `cp` | Copy a file or directory. |
| `man` | Opens the full manual page for a command — `man ls`, for instance — genuinely the fastest way to check exact syntax rather than guessing. |

### `cd -`

Jumps back to whichever directory you were in immediately before your last `cd`. A quick way to toggle between two locations without retyping the full path each time.

### The `-p` flag

Meaning depends on the command, but the one you'll use constantly is with `mkdir`: `mkdir -p some/nested/path` creates every intermediate directory needed along the way, and doesn't complain if some of them already exist. Without `-p`, `mkdir nested/path` fails outright if `nested` doesn't already exist.

### `rm -rf` — and why it deserves respect

Two flags, each doing real work:

- `-r` — recursive. Deletes a directory and everything inside it, not just a single file.
- `-f` — force. Skips confirmation prompts entirely, and silently ignores files that don't exist.

Combined, this is one of the more dangerous commands you'll ever run. It deletes an entire directory tree instantly, with no confirmation and no recycle bin to recover from. The classic disaster scenario is running it against the wrong path — an accidental leading `/`, a typo in the target directory — which can wipe out far more than intended. Running `pwd` first, as an actual habit, is the cheapest insurance against this.

### Filenames with spaces and special characters

Linux allows spaces and most special characters in filenames, but the shell treats a space as a separator between arguments by default, which causes exactly the problem you'd expect:

```bash
# Interpreted as TWO separate arguments, "my" and "file.txt" —
# not one file called "my file.txt"
rm my file.txt
```

Handling it correctly means either quoting the name — `rm "my file.txt"` — or escaping the space directly — `rm my\ file.txt`. In practice, letting the terminal's tab-completion fill in the filename does this automatically and is the safest habit to build.

Characters like `$`, `*`, `&`, `(`, and `)` carry their own meaning to the shell — wildcards, variables, backgrounding a process — so filenames containing them should almost always be quoted too, to stop the shell from interpreting them instead of treating them as plain text.
