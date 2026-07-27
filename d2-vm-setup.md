# Building Your First Server — VirtualBox / VMware + Ubuntu Server

## Why This Comes First

Every DevOps engineer needs somewhere to practice that isn't the production internet. Before touching Docker, Jenkins, or Kubernetes, you need a machine you're free to break, misconfigure, and rebuild without consequence. That's what a virtual machine gives you — a real Linux server, fully isolated inside your own laptop, that you can wipe and start over as many times as it takes.

Two choices matter here, and both are deliberate:

- **A VM, not a dual-boot install.** Dual-booting Linux alongside Windows/Mac works, but a mistake can cost you your whole machine. A VM is disposable by design — if something breaks, you delete it and start again in ten minutes.
- **Server edition, not Desktop.** Real infrastructure doesn't have a screen or a mouse attached to it. Production servers are managed entirely through the terminal, over SSH, often by engineers who've never seen the machine's actual hardware. Starting on Server edition from day one builds that habit early, rather than leaning on a GUI you won't have later.

Everything below sets up exactly that: an isolated Ubuntu Server VM, reachable over SSH, sized to eventually run Docker and Kubernetes without struggling.

---

## Sizing the VM

Since this machine will later run containers and orchestration tools, size it with that in mind rather than the bare minimum:

| Resource | Minimum | Recommended |
| --- | --- | --- |
| Memory | 2048 MB | 4096 MB — containers add up fast |
| Processors | 2 cores | 2 cores |
| Hard disk | 20 GB | 40–50 GB, stored as a single file for better I/O performance |
| Network | NAT | NAT — gives internet access for package installs while keeping the VM isolated |

---

## Option A: VirtualBox

VirtualBox is the more common choice for training environments — it's free, lighter on resources, and well documented.

### 1. Install VirtualBox

1. Download it from [virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads) for your OS.
2. Run the installer, accepting the defaults.
3. Install the **VirtualBox Extension Pack** from the same page — optional, but adds USB support and a few other conveniences worth having.

### 2. Download the Ubuntu Server ISO

1. Go to [ubuntu.com/download/server](https://ubuntu.com/download/server).
2. Download the latest **LTS** release. LTS stands for Long Term Support — it's the more stable, longer-maintained line, and the standard choice for both learning and production over the newest short-term release.

### 3. Create and configure the VM

1. Open VirtualBox and click **New**.
2. Name and OS:
   - Name: `devops-server-01`
   - Type: **Linux**
   - Version: **Ubuntu (64-bit)**
   - ISO Image: leave as **Not selected** for now. This skips VirtualBox's automated "unattended install," which quietly configures things for you and bypasses steps that are worth doing manually while you're still learning what they mean. Click **Next**.
3. Hardware: allocate at least **2048 MB RAM** and **2 CPUs**. Leave "Enable EFI" unchecked. Click **Next**.
4. Virtual hard disk: choose **Create a Virtual Hard Disk Now**, set the size to **40 GB**, format **VDI** (dynamically allocated). Click **Next**, then **Finish**.
5. Mount the ISO:
   - Select the VM, open **Settings** (gear icon), go to the **Storage** tab.
   - Click the empty optical drive icon under "Storage Devices."
   - Click the disk icon on the right, choose **Choose a disk file**, and select the Ubuntu Server ISO you downloaded.
6. Configure networking:
   - Go to the **Network** tab, confirm Adapter 1 is enabled and attached to **NAT**.
   - Click **Advanced → Port Forwarding**, and add a rule:
     - Name: `SSH`, Protocol: `TCP`, Host Port: `2222`, Guest Port: `22`
   - This single rule is what lets you SSH into the VM later from your own terminal using `localhost:2222`, without needing to fuss with bridged networking.

### 4. Install Ubuntu Server

Start the VM and work through the installer:

1. **Boot menu** — select "Welcome to Ubuntu Server," press Enter.
2. **Language & keyboard** — set your preferences.
3. **Type of install** — Ubuntu Server (the default).
4. **Networking** — the installer will already show an IP assigned by VirtualBox's NAT engine, typically `10.0.2.15`. Leave it as-is.
5. **Storage setup** — keep **Use an entire disk** checked, and make sure **Set up this disk as an LVM group** is selected. LVM (Logical Volume Manager) lets you resize disk partitions later without reinstalling — genuinely useful once Docker images and Kubernetes testing start eating disk space. Confirm the destructive action when prompted.
6. **Profile setup** — enter your name, server name, username, and password. These are the credentials you'll use for the rest of the course, so make sure they're memorable.
7. **SSH setup** — press Spacebar to check **Install OpenSSH server**. Without this, there's no way to connect to the machine remotely at all.
8. **Featured snaps** — leave everything unchecked. Tools like Docker get installed manually later, deliberately, not bundled in automatically here.
9. Let the installation finish, then select **Reboot Now**. If it prompts you to remove the installation media, just press Enter — both VirtualBox and VMware handle this automatically in most recent versions.

### 5. Post-install setup

Update the system first, before anything else:

```bash
sudo apt update && sudo apt upgrade -y
```

Install VirtualBox's guest utilities — the equivalent of VMware's guest tools, handling shared folders, time sync, and general integration between host and VM:

```bash
sudo apt install virtualbox-guest-utils -y
```

Then verify remote access actually works, from your host machine's own terminal (not inside the VM window):

```bash
ssh username@127.0.0.1 -p 2222
```

This uses the port forwarding rule set up earlier — replace `username` with the account you created during install.

---

## Option B: VMware Workstation

The overall flow is the same as VirtualBox; the interface and a few defaults differ.

### 1. Create the VM

1. Open VMware Workstation, select **Create a New Virtual Machine**.
2. Choose **Typical (recommended)**.
3. Select **I will install the operating system later**. This avoids VMware's "Easy Install," which — like VirtualBox's unattended install — auto-configures the OS and skips manual steps worth doing by hand while learning.
4. Guest OS: **Linux**, Version: **Ubuntu 64-bit**.
5. Name the VM, e.g. `devops-server-01`.
6. Set disk size to **40 GB or more**, and select **Store virtual disk as a single file**.
7. Click **Customize Hardware**, then under CD/DVD select **Use ISO image file** and browse to the Ubuntu Server ISO.
8. Optional: remove the Sound Card and Printer from the hardware list. Neither is needed on a headless server VM, and trimming them reduces unnecessary overhead.
9. Click **Finish**.

### 2. Install Ubuntu Server

Boot the VM and step through the installer using the arrow keys, `TAB`, and `Enter`:

1. **Language & keyboard** — set preferences.
2. **Type of install** — choose **Ubuntu Server**, not "minimized," unless you're working under severe resource constraints.
3. **Networking** — leave as default; DHCP assigns an IP automatically through VMware's NAT.
4. **Storage** — select **Use an entire disk**, and enable **Set up this disk as an LVM group**, for the same resizing flexibility described above.
5. **Profile setup** — enter your name, server name, username, and password.
6. **SSH setup** — check **Install OpenSSH server**. This is what enables remote connections later via terminal, PuTTY, MobaXterm, or VS Code's remote SSH extension, instead of working inside VMware's console window indefinitely.
7. **Featured snaps** — leave unchecked.
8. Select **Done**, wait for installation to finish, then choose **Reboot Now**.
9. If prompted to remove the installation medium, go into VMware's settings, disconnect the CD/DVD drive, then press `Enter`.

### 3. Post-install setup

Update the system:

```bash
sudo apt update && sudo apt upgrade -y
```

Install VMware's guest tools, which improve performance and make shutdown/restart signals from the host behave correctly:

```bash
sudo apt install open-vm-tools -y
```

Check the VM's IP address:

```bash
ip a
```

Look for the address under the network interface, usually named `ens*` or `eth*`. Then, from your host machine's own terminal:

```bash
ssh username@your_vm_ip
```

---

## Take a Snapshot Before Going Further

Before installing Docker, Kubernetes, or any configuration-management tooling, shut the VM down and take a snapshot of it in its clean state:

- **VMware:** right-click the VM, select Snapshot → Take Snapshot, name it `Clean OS Baseline`.
- **VirtualBox:** open the hamburger menu next to the VM's name, go to Snapshots, click Take, name it `Base_OS_Installed`.

The reasoning is simple. Setup scripts break things — that's not a hypothetical, it's a near-certainty once you start experimenting with configuration tools. A clean baseline snapshot means a broken environment is a two-minute rollback instead of a full reinstall.

---

## Common Gotchas

- **ISO still mounted after install.** The VM boots back into the installer instead of the OS itself. Check Settings → Storage — most recent versions of both tools handle this automatically, but it's worth confirming if the boot looks wrong.
- **Port forwarding misconfigured (VirtualBox).** If `ssh username@127.0.0.1 -p 2222` fails, recheck the NAT port forwarding rule under Settings → Network → Advanced — Host Port 2222 should map to Guest Port 22.
- **Insufficient RAM.** Docker and Kubernetes will struggle below 2GB. Increase the allocation if your host machine has the headroom.
- **Silent password typos during install.** Ubuntu Server's installer hides password input completely — no dots, no asterisks — so a mistyped password can go unnoticed until login fails.
- **No snapshot before installing Docker/Kubernetes.** Without one, a broken setup script means reinstalling from scratch rather than rolling back.
- **Skipping LVM during storage setup.** Makes resizing disk space later, once containers and images accumulate, considerably harder than it needs to be.

---

## What Comes Next

Once the VM is running and reachable — either through the console window or over SSH — the environment is ready for the actual Linux work: navigating the filesystem, understanding users and permissions, and building fluency with the shell.
