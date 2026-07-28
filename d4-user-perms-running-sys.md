# Users, Permissions, and Reading a Running System

## Where Run Levels Actually Live

The run-level table from yesterday raises an obvious follow-up: where is that state actually configured?

On the older SysV init systems, it lived in a single file: `/etc/inittab`, which defined which run level the system booted into by default and what happened at each one.

On modern systemd-based systems like Ubuntu Server, that job has moved. The default boot target is defined by a symlink at `/etc/systemd/system/default.target`, pointing to one of the target unit files that live under `/lib/systemd/system/` (things like `multi-user.target` or `graphical.target`). Changing what a system boots into means changing what that symlink points to, either by hand or with `systemctl set-default <target>`.

---

## Reading Disk Usage: `df -h`

`df` stands for "disk free." The `-h` flag means human-readable — sizes shown in KB/MB/GB instead of raw byte counts.

A typical row of output looks like:

```text
Filesystem      Size  Used  Avail  Use%  Mounted on
/dev/sda1        40G   12G    26G   32%  /
```

Each column, left to right:

- **Filesystem** — the actual device or partition being measured, e.g. `/dev/sda1`.
- **Size** — total capacity of that filesystem.
- **Used** — how much space is currently occupied.
- **Avail** — how much space remains.
- **Use%** — used space as a percentage of total — the number worth watching, since a filesystem hitting 100% can cause real problems (failed writes, crashed services).
- **Mounted on** — where in the single Linux directory tree this filesystem is attached. `/` here means this is the root filesystem itself.

---

## Creating and Managing Users

### Checking `useradd`'s help and defaults

`useradd -h` prints the command's usage and full list of available flags — the quickest way to check exact syntax without leaving the terminal.

`useradd -D` prints the *default* values that get applied whenever a new user is created without overriding them explicitly:

```text
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/sh
SKEL=/etc/skel
CREATE_MAIL_SPOOL=no
```

Worth noting: `useradd --h` (double dash, single letter) isn't valid syntax — the correct long-form flag is `--help`, with two dashes and the full word. A single dash expects single-letter flags; a double dash expects full words.

### Creating a basic user and checking the result

```bash
useradd test
cat /etc/passwd | grep test
```

This creates the account, then filters the full user database down to just the line for `test`, confirming the account exists and showing its UID, home directory, and shell — the same seven-field structure covered yesterday.

### Setting a custom home directory

```bash
sudo useradd -d ~/test test
```

The `-d` flag sets a custom home directory instead of the default `/home/test`. One subtlety worth flagging here: `~` expands *before* `sudo` runs, in the shell of whoever is typing the command — so this doesn't necessarily point where you'd expect. If you're logged in as `chitti` and run this with `sudo`, `~/test` resolves to `/home/chitti/test`, not `/home/root/test` or anything tied to the new `test` account. For anything beyond quick experimentation, an explicit absolute path (`-d /home/test`) is safer and clearer.

To change an *existing* user's home directory after the fact, the equivalent command is `usermod` rather than `useradd`:

```bash
sudo usermod -d ~/test test
```

`useradd` creates; `usermod` modifies something that already exists.

### Groups: `/etc/group`

Just as `/etc/passwd` is the user database, `/etc/group` is the group database. A line looks like:

```text
developers:x:1003:chitti,test
```

Four colon-separated fields: group name, password placeholder (essentially unused today), GID, and a comma-separated list of member usernames.

```bash
cat /etc/group | grep test
```

filters that list down to any group where `test` appears — either as the group's own name, or as a listed member of some other group.

### Primary group vs secondary group

This distinction trips people up because it's partly invisible.

- A user's **primary group** is recorded in `/etc/passwd` itself, as the GID field (the fourth field in the line breakdown from yesterday). Every user has exactly one primary group.
- **Secondary groups** are additional group memberships, recorded in `/etc/group`'s member list for each group the user also belongs to. A user can belong to any number of secondary groups.

The part that isn't obvious: a user's primary group membership generally does **not** show up in `/etc/group`'s member list — it's implied entirely through the GID in `/etc/passwd`, not spelled out as a name in `/etc/group`. Secondary memberships, by contrast, are explicitly listed as names in `/etc/group`. Running `id <username>` is the cleanest way to see both at once — it prints the primary group first, then every secondary group after it.

```bash
sudo useradd -g 1003 dev
```

creates the user `dev` with primary group GID `1003` directly, rather than accepting the default.

### Adding a user to a group, and removing users/groups

```bash
sudo useradd dev1
sudo usermod -aG test dev1
sudo userdel dev1
sudo groupadd developers
```

- `usermod -aG test dev1` adds `dev1` to the `test` group as a *secondary* membership. The `-a` (append) flag matters more than it looks: `-G` alone would *replace* all of a user's existing secondary groups with the ones listed, which is rarely what you want. `-aG` adds to the list instead of overwriting it.
- `userdel dev1` deletes the user account. By default it leaves the home directory behind — deleting that too requires the `-r` flag.
- `groupadd developers` creates a brand-new group with no members yet.

---

## Reading `ls -la /home`

Running this against `/home` lists every user's home directory, in full detail. A single line of output looks like:

```text
drwxr-x--- 5 hari hari 4096 Jan 15 10:22 hari
```

Breaking it down left to right:

- **`drwxr-x---`** — a 10-character permission string. The very first character is the file type: `d` for directory, `-` for a regular file, `l` for a symbolic link. The remaining 9 characters come in three groups of three — **owner**, **group**, **others** — each group showing **r**ead, **w**rite, and e**x**ecute permissions, with a dash where that permission is absent. Here: the owner (`hari`) has full read/write/execute; the group has read and execute only; everyone else has nothing at all.
- **`5`** — the number of hard links (for a directory, this includes the directory itself and its subdirectories' references back to it — not something you need to worry about deeply yet).
- **`hari`** (first) — the owning user.
- **`hari`** (second) — the owning group.
- **`4096`** — size in bytes (directories themselves are typically a fixed small size like this, regardless of how much they contain — the size shown is the directory *entry* itself, not the total size of its contents).
- **`Jan 15 10:22`** — last modified date and time.
- **`hari`** (last) — the actual file or directory name.

Most terminals also color-code this output for a fast visual read, though exact colors vary by terminal theme — the common convention is: **blue** for directories, plain white/grey for ordinary files, **green** for anything executable, and **cyan** for symbolic links.

---

## Why Directories Need Execute Permission

This is one of the more counterintuitive things in the whole permission model, and worth sitting with.

**Read** permission on a directory lets you list its contents — run `ls` and see the names of what's inside. **Execute** permission on a directory is what actually lets you *enter* it (`cd`) or access anything inside it by path, even if you already know the exact filename.

These are genuinely separate:

- Read without execute: you can see the *names* of files inside, but can't `cd` into the directory or open any file inside it, even one you can see the name of.
- Execute without read: you can't list what's inside, but if you already know a specific filename, you can `cd` into the directory and access that file directly.

So yes — a directory essentially always needs execute permission to be usable in the way you'd expect. Without it, the directory becomes a dead end: you can't traverse into it or reach anything inside, regardless of what read permission says.

---

## Working Through a Real Permission-Denied Scenario

This sequence is worth walking through in order, because it's a genuinely common source of confusion.

```text
cd /home/ram
cd: /home/ram: No such file or directory
```

This fails simply because no user named `ram` exists yet, so `/home/ram` was never created. `cd` can't enter a directory that doesn't exist.

```bash
sudo useradd hari -m
cd /home/hari
```

```text
cd: permission denied: /home/hari
```

This time the directory *does* exist — but you still can't get in. Why the difference from the earlier `useradd test` (without `-m`)?

The `-m` flag explicitly tells `useradd` to create the home directory. Without it, no home directory gets created at all — which is exactly why `useradd test` earlier left nothing at `/home/test` to `cd` into. On Debian-based distributions like Ubuntu, this is the actual default behavior: `useradd` alone does *not* create a home directory unless told to. (This is distro-dependent — some other distributions default the other way.)

So with `-m`, `/home/hari` genuinely exists this time. But the permission error happens because of what was shown earlier in the `ls -la` breakdown: a newly created home directory is typically owned by the new user with permissions like `drwx------` (or `drwxr-x---`) — meaning only that user (and root) can enter it. If you're logged in as `chitti`, you are neither `hari` nor root, so the execute permission needed to `cd` into `/home/hari` simply isn't granted to you.

```bash
ls -la /home
```

confirms exactly this — showing `hari`'s directory owned by `hari:hari` with restrictive permissions, sitting right next to your own more permissive one.

```bash
newgrp hari
```

is an attempt to switch your *current group* to `hari` for this shell session — but this only works if you're already a member of that group. Since `chitti` isn't a member of `hari`'s group, this also fails with permission denied, for the same underlying reason.

**So why the permission denial, and how do you actually get in?**

Because Linux permissions are enforced strictly by user and group identity, not by whether a directory exists or whether you know its path. There are really three legitimate ways to access `hari`'s home directory:

1. **Become that user.** `sudo su - hari` or `su - hari` switches your session to `hari` entirely, at which point you're accessing your own home directory and every permission check passes naturally.
2. **Use root.** `sudo ls /home/hari` or `sudo cd` (note: `cd` itself can't be run through `sudo` directly since it's a shell builtin, not a program — but commands like `sudo ls` or `sudo cat` on files inside work fine, since root bypasses all normal permission checks).
3. **Deliberately change permissions or group membership** — e.g. adding `chitti` to `hari`'s group and adjusting the directory's group permissions to allow entry. This is the least common approach for a personal home directory, since it defeats the entire point of keeping one user's files private from another.

The underlying lesson: existing and being reachable by path are not the same thing as being accessible. Every step of a path gets its own permission check.

---

## `.` and `..`

`.` refers to the current directory. `..` refers to the parent directory — one level up. These aren't just command shortcuts; they're genuine entries that exist inside every directory on the filesystem, visible with `ls -la`.

The one place this gets slightly strange is at the very top of the tree. Inside `/`, both `.` and `..` point to the exact same place — `/` itself. This makes sense once you think about it: root has no parent to go up to, so "one level up from the top" is defined as staying exactly where you are.

---

## Reading the Machine's Vital Signs

### `du -sh`

`du` reports disk usage. `-s` summarizes into a single total instead of listing every file individually, and `-h` makes it human-readable. So:

```bash
du -sh Desktop
```

reports one number — the total size of everything inside the `Desktop` folder, combined.

### `free -m` and `free -h`

`free` reports memory usage. `-m` forces the output into megabytes; `-h` instead auto-picks whatever unit (KB/MB/GB) makes the number easiest to read. If both flags are given together, whichever is specified last generally wins — in practice, `free -h` on its own is the more commonly used form, since it adapts automatically rather than forcing one fixed unit.

Typical output has two rows — `Mem:` and `Swap:` — with columns for total memory, how much is used, how much is genuinely free, how much is shared, how much is held in buffers/cache (memory the kernel is using to speed up disk access, but which can be reclaimed instantly if an application needs it), and how much is actually available for new processes to use.

### `nproc`

Prints a single number: how many processing units (CPU cores, counting hyperthreads if present) the system has available. Useful both for quick reference and inside scripts that need to size something — like how many parallel jobs to run — based on available hardware.

### `uptime` and load average

```bash
uptime
```

produces a single line, something like:

```text
14:32:05 up 3 days, 4:12, 2 users, load average: 0.52, 0.61, 0.45
```

Reading it left to right: the current time, how long the system has been running continuously since its last boot, how many users are currently logged in, and then the three **load average** numbers — measured over the last 1, 5, and 15 minutes respectively.

**What load average actually measures:** roughly, the average number of processes that were either actively running on a CPU or waiting for their turn, over that time window. It is not a percentage on its own — it needs to be interpreted relative to how many cores the machine has (the same `nproc` number from above).

**Turning it into a percentage:** a load of `1.0` represents the equivalent of one CPU core running at full capacity. So on a single-core machine, a load of `1.0` means 100% utilization. On a 4-core machine, a load of `1.0` means only 25% overall utilization, because there are three other cores sitting idle — the same load of `4.0` on that machine would mean 100% utilization across all cores. The general formula is:

```text
utilization % = (load average / number of cores) × 100
```

**Reading the three numbers together — increasing vs decreasing patterns:** because the three values cover 1, 5, and 15 minutes, comparing them tells you which direction load is currently trending.

- If the 1-minute value is *higher* than the 5-minute, which is higher than the 15-minute (e.g. `2.1, 1.4, 0.9`), load has been **increasing** recently — something has ramped up in just the last minute or so.
- If the 1-minute value is *lower* than the 5-minute, which is lower than the 15-minute (e.g. `0.4, 0.9, 1.6`), load has been **decreasing** — the system is settling down after a busier period.

### `ps aux`

Where `top` gives you a live, continuously refreshing view, `ps aux` gives you a single **snapshot** of every process running on the system at the moment you run it — useful when you want output you can pipe into `grep`, save to a file, or read without the screen constantly redrawing.

Breaking down the flags: `a` shows processes belonging to all users, not just the one running the command; `u` displays them in a detailed, user-oriented format with resource usage; `x` includes processes that aren't attached to a terminal at all — background services and daemons that `ps` would otherwise hide.

A line of output looks like:

```text
USER   PID  %CPU  %MEM   VSZ   RSS TTY   STAT START   TIME COMMAND
root     1   0.0   0.1  1234   768 ?     Ss   09:01   0:02 /sbin/init
```

Most of these columns mean exactly what they meant in `top` — `PID`, `%CPU`, `%MEM` are the same measurements, just captured once instead of live. A few worth calling out specifically:

- **VSZ / RSS** — the `ps` equivalents of `top`'s VIRT and RES: total virtual memory claimed, versus actual physical RAM in use.
- **TTY** — which terminal session the process is attached to, if any. A `?` means it isn't attached to any terminal at all — typical for background services and daemons.
- **STAT** — the process state, using the same letters as `top`'s `S` column (`R` running, `S` sleeping, `Z` zombie, and so on), sometimes with an extra character appended for more detail (e.g. the `s` in `Ss` marks it as a session leader).
- **START** — when the process actually began running.

A common and genuinely useful pattern is piping this into `grep` to find a specific process by name, using exactly the same pipe mechanism covered with `/etc/passwd` earlier:

```bash
ps aux | grep nginx
```

This prints only the lines where "nginx" appears — a fast way to check whether a specific service is running and, if so, grab its PID for something like `kill`.

---

### `top`, and the jargon that comes with it

`top` is an interactive, continuously refreshing view of everything running on the system — the live equivalent of `ps` and `free` combined into one screen. Pressing **`1`** while it's running toggles the CPU summary between a single aggregated line and a separate line per individual core — genuinely useful for spotting whether load is spread evenly or piled onto just one core.

The column headers `top` displays are dense with shorthand worth knowing cold:

| Column | Meaning |
| --- | --- |
| PID | Process ID — the unique number identifying this running process |
| USER | Which user account owns this process |
| PR | Priority — how favorably the kernel's scheduler treats this process |
| NI | "Niceness" — a user-adjustable modifier to priority; a higher nice value means the process is more willing to yield CPU time to others |
| VIRT | Total virtual memory the process has claimed, including memory not actually in physical RAM |
| RES | Resident memory — the actual physical RAM currently being used |
| SHR | Shared memory — RAM shared with other processes (e.g. shared libraries) |
| S | Process state — the single most useful column at a glance: `R` running, `S` sleeping (idle, waiting for something), `D` uninterruptible sleep (usually waiting on disk I/O), `Z` zombie (covered below), `T` stopped |
| %CPU | Percentage of CPU currently being used by this process |
| %MEM | Percentage of total RAM this process is using |
| TIME+ | Total accumulated CPU time this process has consumed since it started |
| COMMAND | The actual program/command name |

### `stress`

`stress` is a small utility that does exactly what its name suggests — deliberately loads the system to test how it behaves under pressure. It doesn't do anything useful on its own; its entire job is to consume CPU, memory, disk I/O, or some combination, on demand. A typical invocation like `stress --cpu 4` spins up four worker processes doing pointless CPU-bound busywork, purely to push load average and CPU usage up in a controlled, repeatable way — useful for testing monitoring setups, watching how `top` and `uptime` behave under real load, or verifying that resource limits and alerts actually trigger the way they're supposed to.

### Zombie processes and deadlocks

A **zombie process** is one that has already finished running, but still has an entry sitting in the process table, shown as state `Z` in `top`. This happens because a process's exit status has to be read by its parent process (via a system call called `wait()`) before the kernel fully removes its entry — until that happens, the finished process lingers as a zombie, technically dead but not yet cleaned up. A small number of short-lived zombies is normal and harmless; large numbers accumulating usually points to a parent process that's misbehaving and never reaping its children.

A **deadlock** is a different kind of problem entirely — it's not a single process's state, but a standoff between two or more processes. Each one is waiting on a resource (a lock, a file, memory) that's currently held by another process in the same waiting group, and none of them can proceed until the others release what they're holding — which never happens, because they're all stuck the same way. Nothing crashes; everything just permanently stops making progress. This is a classic concept from operating systems and concurrent programming, and it's exactly the kind of failure automated monitoring and timeouts are designed to catch, since a deadlocked process won't necessarily show up as an error — it'll just quietly stop responding.
