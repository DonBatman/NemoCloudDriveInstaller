# Nemo Cloud Drive Setup Wizard

A lightweight, beginner-friendly graphical utility for **Linux Mint / Cinnamon** (and any desktop running the **Nemo File Manager**) that connects **Google Drive, Microsoft OneDrive, Dropbox, Proton Drive, pCloud, Box, and MEGA** directly into Nemo's sidebar—**zero terminal commands required**.

![License](https://img.shields.io/badge/license-GPL--3.0-blue.svg)
![Platform](https://img.shields.io/badge/desktop-Cinnamon%20%2F%20Nemo-green.svg)

---

## Why Use This Instead of Built-In Online Accounts?

GNOME Online Accounts (GVFS) frequently suffers from slow directory refreshes, read-only lockouts, and file-transfer errors in Nemo. 

**Nemo Cloud Drive Setup** wraps the industry-standard `rclone` sync engine in a native GTK 3 desktop wizard that:
* **Requires Zero Terminal Usage:** Handles browser OAuth logins, folder creation, and background service setup with a few clicks.
* **Works Like Native Local Folders:** Mounts your cloud storage to `~/Cloud/<DriveName>` using full VFS caching so applications can open, edit, and save files smoothly.
* **Pins Automatically to Nemo:** Adds each connected drive directly to Nemo's left-hand sidebar bookmarks.
* **Auto-Reconnects on Boot:** Configures a `systemd` user service so your drives automatically mount in the background every time you log in.
* **Includes Nemo Auto-Detection:** Built specifically for Nemo—if you run it on a system without Nemo installed, it provides a one-click button to install Nemo for you.

---

## Installation (No Terminal Needed)

1. Go to the **[Releases](../../releases)** page and download the latest `nemo-cloud-setup.deb` file.
2. Double-click the downloaded `.deb` file in your file manager to open the graphical **Software Installer**.
3. Click **Install Package** and enter your password when prompted.
4. Open your system's **Applications Menu** and launch **Nemo Cloud Drive Setup** (found under *System* or *Accessories*).

*(Optional Terminal Install)*:
```bash
sudo apt install ./nemo-cloud-setup.deb
