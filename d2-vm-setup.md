# VM Setup — VirtualBox / VMware + Ubuntu Server

**Task:** Install a virtualization tool (VMware or VirtualBox) and set up an Ubuntu Server edition VM.

This is the foundation for the rest of the course — almost everything (Linux practice, Jenkins, Docker, Kubernetes) will run inside this VM or ones like it.

---

## Why a VM, and why Server (not Desktop)?

- A **VM** gives an isolated Linux environment on your Windows/Mac machine without dual-booting — safe to break things and reset.
- **Server edition** (not Desktop) is used because real DevOps work happens over the command line, not a GUI — servers in production never have a desktop environment. Learning to work entirely via terminal/SSH from day one builds the right habits.

---

## Option A: VirtualBox (recommended — free, lighter, widely used in training)

### 1. Install VirtualBox

1. Download from [virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads) for your OS (Windows/Mac).
2. Run the installer, accept defaults.
3. Also install the **VirtualBox Extension Pack** from the same page (adds USB support, etc. — optional but useful).

### 2. Download Ubuntu Server ISO

1. Go to [ubuntu.com/download/server](https://ubuntu.com/download/server).
2. Download the latest **LTS** version (LTS = Long Term Support, more stable — preferred for learning/production over the newest short-term release).

### 3. Create and configure the VM

1. Open VirtualBox → click **New**.
2. Name and OS:
   - Name: `devops-server-01`
   - Type: **Linux**
   - Version: **Ubuntu (64-bit)**
   - ISO Image: leave as **Not selected** for now — this avoids VirtualBox's automated "unattended install," which skips manual setup steps worth doing by hand while learning. Click **Next**.
3. Hardware: allocate at least **2048 MB RAM** and **2 CPUs**. Leave "Enable EFI" unchecked. Click **Next**.
4. Virtual Hard Disk: select **Create a Virtual Hard Disk Now**, set size to **40 GB**, format **VDI** (dynamically allocated). Click **Next**, then **Finish**.
5. Mount the ISO:
   - Select the VM → **Settings** (gear icon) → **Storage** tab.
   - Under "Storage Devices," click the empty optical drive icon.
   - Click the disk icon on the right → **Choose a disk file** → select the downloaded Ubuntu Server ISO.
6. Network configuration:
   - Go to the **Network** tab → ensure Adapter 1 is enabled and attached to **NAT**.
   - Click **Advanced → Port Forwarding** → add a new rule:
     - Name: `SSH`, Protocol: `TCP`, Host Port: `2222`, Guest Port: `22`
   - This lets you SSH into the VM from your host machine using `localhost:2222`, without needing Bridged networking.

### 4. Install Ubuntu Server

Click **Start** to boot the VM and step through the installer:

1. **Boot menu** — select "Welcome to Ubuntu Server," press Enter.
2. **Language & keyboard** — pick your preferences.
3. **Type of install** — Ubuntu Server (default).
4. **Networking** — the installer shows an IP from VirtualBox's NAT engine (usually `10.0.2.15`). Leave as default.
5. **Storage setup** — keep **Use an entire disk** checked, and ensure **Set up this disk as an LVM group** is selected (useful for resizing disk space later once Docker/Kubernetes start filling it up). Confirm the destructive action when prompted.
6. **Profile setup** — enter your name, server name, username, and password. Remember these.
7. **SSH setup** — press Spacebar to check **Install OpenSSH server**.
8. **Featured snaps** — leave unchecked; install DevOps tools manually later.
9. Let installation finish, then select **Reboot Now**. If prompted to remove installation media, just press Enter — VirtualBox unmounts the ISO automatically.

### 5. Post-install setup

**Update the system:**

```bash
sudo apt update && sudo apt upgrade -y
```

**Install VirtualBox Guest Utilities** — VirtualBox's equivalent of VMware's guest tools; manages system states, shared folders, and time sync inside the VM:

```bash
sudo apt install virtualbox-guest-utils -y
```

**Verify remote access** — from your actual host machine's terminal (CMD, PowerShell, or macOS Terminal):

```bash
ssh username@127.0.0.1 -p 2222
```

(Using the port forwarding rule set up in step 3 — replace `username` with your login account.)

---

## Hardware Requirements for a DevOps Lab VM

Since this VM will later run Docker, Kubernetes, and Ansible, size it accordingly:

| Resource | Minimum | Recommended |
| --- | --- | --- |
| Memory | 2048 MB | 4096 MB (for running containers comfortably) |
| Processors | 2 cores | 2 cores |
| Hard Disk | 20 GB | 40–50 GB, stored as a single file for better I/O performance |
| Network | NAT | NAT (internet access for packages, while keeping the VM isolated) |

---

## Option B: VMware Workstation (detailed walkthrough)

### 1. Create the VM

1. Open VMware Workstation → **Create a New Virtual Machine**.
2. Choose **Typical (recommended)**.
3. Select **I will install the operating system later** — this avoids VMware's "Easy Install," which auto-configures things and skips manual setup steps that are worth doing by hand while learning.
4. Guest OS: **Linux**, Version: **Ubuntu 64-bit**.
5. Name the VM (e.g. `devops-server-01`).
6. Set disk size to **40 GB+**, and select **Store virtual disk as a single file**.
7. Click **Customize Hardware** → under CD/DVD, select **Use ISO image file** and browse to the downloaded Ubuntu Server ISO.
8. Optional cleanup: remove the Sound Card and Printer from the hardware list — unnecessary overhead for a headless server VM.
9. Click **Finish**.

### 2. Install Ubuntu Server

Boot the VM and step through the installer (arrow keys, `TAB`, `Enter`):

1. **Language & keyboard** — pick your preferences.
2. **Type of install** — choose **Ubuntu Server** (not "minimized," unless resources are very constrained).
3. **Networking** — leave default; DHCP will assign an IP automatically via VMware NAT.
4. **Storage** — select **Use an entire disk**, and enable **Set up this disk as an LVM group**.
   - **Why LVM matters:** LVM (Logical Volume Manager) lets you resize partitions later without reinstalling — genuinely useful once you start filling disk space with Docker images and Kubernetes testing.
5. **Profile setup** — enter your name, server name, username, and password. Remember these.
6. **SSH setup** — check **Install OpenSSH server**. This is what lets you connect remotely later via terminal, PuTTY, MobaXterm, or VS Code's remote SSH extension, instead of working inside the VMware console window.
7. **Featured snaps** — leave unchecked; DevOps tools (Docker, etc.) get installed manually later, not via snap.
8. Select **Done**, let installation finish, then **Reboot Now**.
9. If prompted to remove the installation medium: go to VMware settings, disconnect the CD/DVD drive, then press `Enter`.

### 3. Post-install setup

**Update the system:**

```bash
sudo apt update && sudo apt upgrade -y
```

**Install VMware Guest Tools** — improves VM performance, resource handling, and power-state behavior (shutdown/restart signals working correctly from the host):

```bash
sudo apt install open-vm-tools -y
```

**Check network/IP and confirm SSH access:**

```bash
ip a
```

Look for the IP under your network interface (commonly named `ens*` or `eth*`), then from your host machine:

```bash
ssh username@your_vm_ip
```

---

## DevOps Best Practice: Take a Snapshot

Before installing Docker, Kubernetes, or any configuration-management agents, shut the VM down and take a snapshot:

- **VMware:** Right-click the VM → Snapshot → Take Snapshot. Name it `Clean OS Baseline`.
- **VirtualBox:** Click the hamburger menu next to the VM name → **Snapshots** → **Take**. Name it `Base_OS_Installed`.

Why this matters: once you start running setup scripts and experimenting (which is normal in DevOps practice), things *will* break occasionally. A clean baseline snapshot means you can roll back instantly instead of reinstalling the whole OS from scratch.

---

## Common Gotchas

- **Forgetting to remove the ISO after install** → VM boots back into the installer instead of the installed OS. Fix in Settings → Storage. (Usually automatic on both VMware and VirtualBox now, but worth checking if it boots wrong.)
- **Port forwarding rule missing or wrong** (VirtualBox) → if `ssh username@127.0.0.1 -p 2222` fails, double check the NAT port forwarding rule (Host Port 2222 → Guest Port 22) under Settings → Network → Advanced.
- **Not enough RAM allocated** → things like Docker/Kubernetes later in the course will struggle below 2GB; bump it up if your host machine allows.
- **Typing the password wrong during setup and not realizing** → Ubuntu Server install hides the password as you type it (no dots/asterisks) — easy to mistype without noticing.
- **Skipping the snapshot before installing Docker/Kubernetes** → if a setup script breaks the OS, there's no quick way back without a baseline snapshot taken beforehand.
- **Skipping LVM during storage setup** → makes resizing disk space later (once containers/images pile up) much harder.

---

## Next Steps

Once the VM is up and you can log in (either via the VirtualBox window or SSH), you're ready for hands-on Linux basics — navigating the filesystem, users/permissions, and basic shell commands.
