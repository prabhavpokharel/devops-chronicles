# Package Management with APT

## What APT Is

**APT** stands for **Advanced Package Tool** — the package management system used on Debian-based Linux distributions, which includes Ubuntu, Debian itself, and Linux Mint among others. It's the tool responsible for installing, updating, upgrading, and removing software on the system, handling dependency resolution along the way so you don't have to manually track down every library a piece of software needs.

This is the tool behind commands that have already come up in earlier notes without being explained directly — `sudo apt install iputils-ping`, `sudo apt install curl`, and similar. Worth having the full picture in one place.

## Core Commands

| Command | What it does |
| --- | --- |
| `sudo apt update` | Refreshes the local index of available packages and their versions from configured repositories. This doesn't install or upgrade anything by itself — it just updates APT's knowledge of what's available, and is almost always run before `install` or `upgrade` to make sure that knowledge is current. |
| `sudo apt upgrade` | Upgrades every installed package to its latest available version, based on the index `update` just refreshed. |
| `sudo apt install <package>` | Installs a specific package, along with anything it depends on. |
| `sudo apt remove <package>` | Removes a package, but leaves its configuration files behind — useful if you might reinstall it later and want to keep existing settings. |
| `sudo apt purge <package>` | Removes a package *and* its configuration files — a genuinely complete removal, as opposed to `remove`'s partial one. |
| `sudo apt search <term>` | Searches package names and descriptions for a keyword, without installing anything — useful for finding the exact package name when you're not sure of it. |
| `sudo apt show <package>` | Displays detailed information about a package — version, dependencies, description — without installing it. |
| `sudo apt autoremove` | Removes packages that were automatically installed as dependencies of something else, but are no longer needed by anything currently installed. Worth running occasionally to clean up leftover dependency clutter. |

## The `update` vs `upgrade` Distinction, Worth Repeating

This trips people up constantly because the names sound almost interchangeable: `update` refreshes the *list* of what's available; `upgrade` actually *installs* newer versions of what's already on the system. Running `upgrade` without having run `update` first risks upgrading against a stale, outdated picture of what versions even exist — which is exactly why the two are almost always run back to back:

```bash
sudo apt update && sudo apt upgrade -y
```
