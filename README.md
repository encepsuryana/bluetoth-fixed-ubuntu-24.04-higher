# 🛠️ Bluetooth Auto-Fix on Ubuntu (Systemd + Shell Script)

If your Bluetooth does not automatically start after booting Ubuntu, this guide will help you fix it by creating a custom shell script and a systemd service that runs on every boot.

---

## ✅ What This Does

This script will:

- Stop and unmask the Bluetooth service
- Start and enable it again
- Mask it if needed
- Reload the `btusb` kernel module
- Unblock Bluetooth via `rfkill`

---

## 🧩 Step 1: Create the Bluetooth Reset Script

Open terminal and create a shell script:

```bash
sudo nano /usr/local/bin/bluetooth_reset.sh
```

Paste the following content:

```bash
#!/bin/bash

set -e

echo "==> Stopping bluetooth.service if running..."
systemctl stop bluetooth.service || true

echo "==> Unmasking bluetooth.service..."
systemctl unmask bluetooth.service

echo "==> Starting bluetooth.service..."
systemctl start bluetooth.service
systemctl enable bluetooth.service

echo "==> Masking bluetooth.service again..."
systemctl mask bluetooth.service

echo "==> Removing btusb module..."
rmmod btusb || true

echo "==> Loading btusb module..."
modprobe btusb

echo "==> Unblocking all Bluetooth devices..."
rfkill unblock all

echo "==> Bluetooth reset completed."
```

Save the file and exit Nano (press `CTRL+O`, then `ENTER`, then `CTRL+X`).

Make the script executable:

```bash
sudo chmod +x /usr/local/bin/bluetooth_reset.sh
```

✅ **Script created!**

---

## ⚙️ Step 2: Create a Systemd Service

Now, create a new systemd unit file:

```bash
sudo nano /etc/systemd/system/bluetooth-fix.service
```

Paste this content:

```ini
[Unit]
Description=Auto-fix Bluetooth on boot
After=multi-user.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/bluetooth_reset.sh

[Install]
WantedBy=multi-user.target
```

Save the file and exit.

✅ **Service file created!**

---

## 🚀 Step 3: Enable the Service

Reload systemd to recognize the new service:

```bash
sudo systemctl daemon-reload
```

Enable the service to run automatically at every boot:

```bash
sudo systemctl enable bluetooth-fix.service
```

(Optional) Run it immediately to test:

```bash
sudo systemctl start bluetooth-fix.service
```

✅ **Your Bluetooth fix should now run at every boot!**

---

## 📋 Step 4: Verify the Service

Check if the service is enabled:

```bash
systemctl is-enabled bluetooth-fix.service
```

Check the service status:

```bash
systemctl status bluetooth-fix.service
```

Example successful output:

```
● bluetooth-fix.service - Auto-fix Bluetooth on boot
     Loaded: loaded (/etc/systemd/system/bluetooth-fix.service; enabled; preset: enabled)
     Active: inactive (dead) since Wed 2025-07-02 08:14:02 WIB; 5min ago
    Process: 1234 ExecStart=/usr/local/bin/bluetooth_reset.sh (code=exited, status=0/SUCCESS)
```

Note: `inactive (dead)` is normal for oneshot services after they finish running.

To view logs for troubleshooting:

```bash
journalctl -u bluetooth-fix.service
```

✅ **You can now confirm the service runs successfully at boot!**

---

## ❌ Optional: Uninstall the Fix

If you ever want to remove the Bluetooth fix:

1. Stop the service:

    ```bash
    sudo systemctl stop bluetooth-fix.service
    ```

2. Disable the service:

    ```bash
    sudo systemctl disable bluetooth-fix.service
    ```

3. Remove the service file:

    ```bash
    sudo rm /etc/systemd/system/bluetooth-fix.service
    ```

4. Remove the Bluetooth reset script:

    ```bash
    sudo rm /usr/local/bin/bluetooth_reset.sh
    ```

5. Reload systemd:

    ```bash
    sudo systemctl daemon-reload
    ```

✅ **Uninstall complete!**

---

## 🧠 Notes

- This service is of type `oneshot` — it runs once at boot and then stops.
- Make sure you run these steps with root privileges (or use `sudo`).
- If `rmmod` or `modprobe` fail, check if Secure Boot is enabled in your BIOS/UEFI. You may need to disable Secure Boot.

---

## ✅ Done!

Your Bluetooth should now automatically reset and work properly after every boot.

Happy hacking!
