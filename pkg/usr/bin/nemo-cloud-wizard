#!/usr/bin/env python3
import os
import shutil
import subprocess
import threading
import gi

gi.require_version("Gtk", "3.0")
from gi.repository import Gtk, GLib

PROVIDERS = {
    "Google Drive": "drive",
    "Microsoft OneDrive": "onedrive",
    "Dropbox": "dropbox",
    "Proton Drive": "protondrive",
    "pCloud": "pcloud",
    "Box": "box",
    "MEGA": "mega",
}

SYSTEMD_TEMPLATE = """[Unit]
Description=Rclone Mount for %i
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
ExecStartPre=/bin/mkdir -p %h/Cloud/%i
ExecStart=/usr/bin/rclone mount %i: %h/Cloud/%i \\
    --vfs-cache-mode full \\
    --vfs-cache-max-age 24h \\
    --dir-cache-time 5m \\
    --log-level INFO
ExecStop=/bin/fusermount -u %h/Cloud/%i
Restart=on-failure
RestartSec=10

[Install]
WantedBy=default.target
"""


class NemoCloudWizard(Gtk.Window):
    def __init__(self):
        super().__init__(title="Nemo Cloud Drive Installer")
        self.set_border_width(20)
        self.set_default_size(500, 440)
        self.set_position(Gtk.WindowPosition.CENTER)

        main_box = Gtk.Box(orientation=Gtk.Orientation.VERTICAL, spacing=14)
        self.add(main_box)

        # 1. Title & Clear Notice that this is for Nemo
        title = Gtk.Label()
        title.set_markup("<span size='x-large' weight='bold'>Nemo Cloud Drive Setup Wizard</span>")
        title.set_xalign(0)
        main_box.pack_start(title, False, False, 0)

        notice_frame = Gtk.Frame()
        notice_box = Gtk.Box(orientation=Gtk.Orientation.VERTICAL, spacing=8)
        notice_box.set_border_width(12)
        notice_frame.add(notice_box)

        lbl_notice = Gtk.Label()
        lbl_notice.set_markup(
            "<b>Designed for the Nemo File Manager</b>\n"
            "This program connects cloud storage drives directly into Nemo's sidebar so they work like normal local folders."
        )
        lbl_notice.set_line_wrap(True)
        lbl_notice.set_xalign(0)
        notice_box.pack_start(lbl_notice, False, False, 0)

        # Nemo Detection & One-Click Installer Bar
        self.nemo_status_box = Gtk.Box(spacing=10)
        self.lbl_nemo_status = Gtk.Label()
        self.lbl_nemo_status.set_xalign(0)
        self.btn_install_nemo = Gtk.Button(label="Install Nemo Now")
        self.btn_install_nemo.get_style_context().add_class("suggested-action")
        self.btn_install_nemo.connect("clicked", self.on_install_nemo_clicked)

        self.nemo_status_box.pack_start(self.lbl_nemo_status, True, True, 0)
        self.nemo_status_box.pack_end(self.btn_install_nemo, False, False, 0)
        notice_box.pack_start(self.nemo_status_box, False, False, 0)

        main_box.pack_start(notice_frame, False, False, 0)

        # 2. Input Fields Grid
        grid = Gtk.Grid(column_spacing=14, row_spacing=12)
        main_box.pack_start(grid, False, False, 4)

        lbl_service = Gtk.Label(label="1. Choose Cloud Service:")
        lbl_service.set_xalign(0)
        self.combo_provider = Gtk.ComboBoxText()
        for name in PROVIDERS.keys():
            self.combo_provider.append_text(name)
        self.combo_provider.set_active(0)
        self.combo_provider.connect("changed", self.on_provider_changed)

        grid.attach(lbl_service, 0, 0, 1, 1)
        grid.attach(self.combo_provider, 1, 0, 1, 1)

        lbl_folder = Gtk.Label(label="2. Folder Name in Nemo:")
        lbl_folder.set_xalign(0)
        self.entry_name = Gtk.Entry()
        self.entry_name.set_text("GoogleDrive")
        self.entry_name.set_hexpand(True)

        grid.attach(lbl_folder, 0, 1, 1, 1)
        grid.attach(self.entry_name, 1, 1, 1, 1)

        # Optional Checkbox for Auto-Start
        self.chk_autostart = Gtk.CheckButton(label="Automatically reconnect this drive every time I turn on my computer")
        self.chk_autostart.set_active(True)
        grid.attach(self.chk_autostart, 0, 2, 2, 1)

        # 3. Primary Action Button
        self.btn_setup = Gtk.Button(label="Log In & Connect Drive to Nemo")
        self.btn_setup.get_style_context().add_class("suggested-action")
        self.btn_setup.set_size_request(-1, 40)
        self.btn_setup.connect("clicked", self.on_setup_clicked)
        main_box.pack_start(self.btn_setup, False, False, 4)

        # 4. Progress & Status Feedback
        status_row = Gtk.Box(spacing=8)
        self.spinner = Gtk.Spinner()
        self.lbl_status = Gtk.Label(label="Fill out the fields above and click Connect to begin.")
        self.lbl_status.set_xalign(0)
        self.lbl_status.set_line_wrap(True)
        status_row.pack_start(self.spinner, False, False, 0)
        status_row.pack_start(self.lbl_status, True, True, 0)
        main_box.pack_start(status_row, False, False, 0)

        # 5. Connected Drives List
        lbl_list = Gtk.Label()
        lbl_list.set_markup("<b>Currently Connected Drives:</b>")
        lbl_list.set_xalign(0)
        main_box.pack_start(lbl_list, False, False, 0)

        self.list_box = Gtk.ListBox()
        self.list_box.set_selection_mode(Gtk.SelectionMode.NONE)
        scrolled = Gtk.ScrolledWindow()
        scrolled.set_min_content_height(90)
        scrolled.add(self.list_box)
        main_box.pack_start(scrolled, True, True, 0)

        self.check_nemo_installed()
        self.refresh_connected_drives()

    def check_nemo_installed(self):
        if shutil.which("nemo"):
            self.lbl_nemo_status.set_markup("<span foreground='#2ec27e'>✔ Nemo File Manager is installed and ready.</span>")
            self.btn_install_nemo.hide()
            self.btn_setup.set_sensitive(True)
        else:
            self.lbl_nemo_status.set_markup("<span foreground='#e01b24'>✖ Nemo File Manager was not found on this PC.</span>")
            self.btn_install_nemo.show()
            self.btn_setup.set_sensitive(False)

    def on_install_nemo_clicked(self, widget):
        self.btn_install_nemo.set_sensitive(False)
        self.set_busy(True, "Installing Nemo File Manager (enter your password in the popup)...")

        def _install():
            subprocess.run(["pkexec", "apt-get", "update"])
            res = subprocess.run(["pkexec", "apt-get", "install", "-y", "nemo"])

            def _done():
                self.btn_install_nemo.set_sensitive(True)
                self.check_nemo_installed()
                if res.returncode == 0:
                    self.set_busy(False, "Nemo installed successfully! You can now connect your drive.")
                else:
                    self.set_busy(False, "Nemo installation was cancelled or failed.")

            GLib.idle_add(_done)

        threading.Thread(target=_install, daemon=True).start()

    def on_provider_changed(self, combo):
        text = combo.get_active_text()
        if text:
            clean = text.replace("Microsoft ", "").replace(" ", "")
            self.entry_name.set_text(clean)

    def set_busy(self, busy, msg):
        def _ui():
            self.btn_setup.set_sensitive(not busy and bool(shutil.which("nemo")))
            self.lbl_status.set_text(msg)
            if busy:
                self.spinner.start()
            else:
                self.spinner.stop()
        GLib.idle_add(_ui)

    def on_setup_clicked(self, widget):
        provider_label = self.combo_provider.get_active_text()
        provider_type = PROVIDERS[provider_label]
        remote_name = "".join(c for c in self.entry_name.get_text() if c.isalnum() or c in ("-", "_"))
        autostart = self.chk_autostart.get_active()

        if not remote_name:
            self.lbl_status.set_text("Please enter a valid folder name using letters or numbers.")
            return

        threading.Thread(
            target=self.run_setup,
            args=(remote_name, provider_type, autostart),
            daemon=True
        ).start()

    def run_setup(self, remote_name, provider_type, autostart):
        # 1. Install rclone & fuse3 if missing
        if not shutil.which("rclone"):
            self.set_busy(True, "Installing background sync engine (enter your password in the popup)...")
            res = subprocess.run(["pkexec", "apt-get", "install", "-y", "rclone", "fuse3"])
            if res.returncode != 0:
                self.set_busy(False, "Setup cancelled: required background tools were not installed.")
                return

        # 2. Open browser for OAuth login
        self.set_busy(True, f"Opening your web browser to log into {remote_name}...")
        cmd = ["rclone", "config", "create", remote_name, provider_type]
        if provider_type == "drive":
            cmd.append("scope=drive")

        res = subprocess.run(cmd, capture_output=True, text=True)
        if res.returncode != 0:
            self.set_busy(False, f"Login failed or cancelled: {res.stderr.strip()}")
            return

        # 3. Create mount directory and systemd service
        self.set_busy(True, "Connecting folder to Nemo...")
        mount_path = os.path.expanduser(f"~/Cloud/{remote_name}")
        os.makedirs(mount_path, exist_ok=True)

        systemd_dir = os.path.expanduser("~/.config/systemd/user")
        os.makedirs(systemd_dir, exist_ok=True)
        with open(os.path.join(systemd_dir, "rclone@.service"), "w") as f:
            f.write(SYSTEMD_TEMPLATE)

        subprocess.run(["systemctl", "--user", "daemon-reload"])
        if autostart:
            subprocess.run(["systemctl", "--user", "enable", "--now", f"rclone@{remote_name}.service"])
        else:
            subprocess.run(["systemctl", "--user", "start", f"rclone@{remote_name}.service"])

        # 4. Pin to Nemo sidebar & open Nemo to the new folder
        self.add_nemo_bookmark(mount_path, remote_name)
        subprocess.Popen(["nemo", mount_path])

        GLib.idle_add(self.refresh_connected_drives)
        self.set_busy(False, f"All done! '{remote_name}' is now open in Nemo and pinned to your sidebar.")

    def add_nemo_bookmark(self, path, label):
        bookmarks_file = os.path.expanduser("~/.config/gtk-3.0/bookmarks")
        os.makedirs(os.path.dirname(bookmarks_file), exist_ok=True)
        uri = f"file://{path} {label}\n"
        existing = open(bookmarks_file).read() if os.path.exists(bookmarks_file) else ""
        if f"file://{path}" not in existing:
            with open(bookmarks_file, "a") as f:
                f.write(uri)

    def refresh_connected_drives(self):
        for child in self.list_box.get_children():
            self.list_box.remove(child)

        if not shutil.which("rclone"):
            return

        res = subprocess.run(["rclone", "listremotes"], capture_output=True, text=True)
        remotes = [r.rstrip(":") for r in res.stdout.strip().splitlines() if r.strip()]

        if not remotes:
            row = Gtk.ListBoxRow()
            lbl = Gtk.Label(label="No cloud drives connected yet.")
            lbl.set_margin_top(8)
            lbl.set_margin_bottom(8)
            row.add(lbl)
            self.list_box.add(row)
        else:
            for remote in remotes:
                row = Gtk.ListBoxRow()
                hbox = Gtk.Box(spacing=10)
                hbox.set_margin_top(6)
                hbox.set_margin_bottom(6)
                hbox.set_margin_start(6)
                hbox.set_margin_end(6)

                lbl = Gtk.Label(label=f"📁  {remote}  (~/Cloud/{remote})")
                lbl.set_xalign(0)
                hbox.pack_start(lbl, True, True, 0)

                btn_open = Gtk.Button(label="Open")
                btn_open.connect("clicked", lambda _, r=remote: subprocess.Popen(["nemo", os.path.expanduser(f"~/Cloud/{r}")]))
                hbox.pack_end(btn_open, False, False, 0)

                btn_remove = Gtk.Button(label="Remove")
                btn_remove.connect("clicked", self.on_disconnect_clicked, remote)
                hbox.pack_end(btn_remove, False, False, 0)

                row.add(hbox)
                self.list_box.add(row)

        self.list_box.show_all()

    def on_disconnect_clicked(self, widget, remote):
        subprocess.run(["systemctl", "--user", "disable", "--now", f"rclone@{remote}.service"])
        subprocess.run(["rclone", "config", "delete", remote])
        self.refresh_connected_drives()
        self.lbl_status.set_text(f"Removed {remote}.")


if __name__ == "__main__":
    win = NemoCloudWizard()
    win.connect("destroy", Gtk.main_quit)
    win.show_all()
    Gtk.main()
