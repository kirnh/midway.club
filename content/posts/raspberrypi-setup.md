---
Date: 2025-03-14
Title: Raspberry Pi Headless Setup Guide
---

# Raspberry Pi Headless Setup Guide

Setting up a Raspberry Pi without a monitor, keyboard, or mouse (headless setup) is a convenient way to configure your Pi for remote use. This guide will walk you through the entire process, from flashing the OS to connecting via SSH and setting up Wi-Fi.

## Prerequisites
- A Raspberry Pi (any model, preferably with built-in Wi-Fi like the Raspberry Pi 4, 3B+)
- A microSD card (16GB or larger, Class 10 recommended)
- A microSD card reader
- A power adapter suitable for your Raspberry Pi
- A computer (Windows, macOS, or Linux)
- A stable internet connection

---

## Step 1: Download Raspberry Pi OS
Raspberry Pi OS (formerly Raspbian) is the recommended operating system. You can download it from the official site:
- [Raspberry Pi OS Download](https://www.raspberrypi.com/software/)
- Choose **Raspberry Pi OS Lite** (for headless setup, no desktop environment required)

Alternatively, use Raspberry Pi Imager to download and flash the OS.

---

## Step 2: Flash Raspberry Pi OS to MicroSD Card
Use one of the following methods:

### Option 1: Using Raspberry Pi Imager (Recommended)
1. Download and install [Raspberry Pi Imager](https://www.raspberrypi.com/software/).
2. Insert your microSD card into your computer.
3. Open Raspberry Pi Imager and select:
   - **Choose OS** → "Raspberry Pi OS Lite (64-bit)"
   - **Choose Storage** → Select your microSD card
4. Click **Next** and then select **Settings (⚙️ icon)** to configure headless setup:
   - **Enable SSH** (Use password authentication or set up an SSH key)
   - **Set Wi-Fi SSID & Password** (For Wi-Fi connection)
   - **Set Hostname** (Optional, but recommended)
5. Click **Write** and wait for the process to complete.

### Option 2: Using Balena Etcher
1. Download and install [Balena Etcher](https://www.balena.io/etcher/).
2. Open Balena Etcher and select the downloaded Raspberry Pi OS `.img` file.
3. Choose your microSD card as the target.
4. Click **Flash!** and wait for the process to finish.

---

## Step 3: Enable SSH and Configure Wi-Fi Manually
If you used Raspberry Pi Imager’s advanced settings, skip this step.

1. After flashing, eject and reinsert the microSD card into your computer.
2. Open the `boot` partition of the microSD card.
3. Create an empty file named `ssh` (without any extension). This enables SSH access.
4. Create a file called `wpa_supplicant.conf` and add the following content:
   ```plaintext
   country=US  # Change to your country code
   ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
   update_config=1
   network={
       ssid="Your_WiFi_SSID"
       psk="Your_WiFi_Password"
       key_mgmt=WPA-PSK
   }
   ```
5. Save the file and eject the microSD card safely.

---

## Step 4: First Boot and SSH Connection
1. Insert the microSD card into the Raspberry Pi and power it on.
2. Find the Pi's IP address:
   - Check your router’s admin page
   - Use a network scanner like [Angry IP Scanner](https://angryip.org/)
   - Use `ping raspberrypi.local` (only works on some networks)
3. Open a terminal (or Command Prompt on Windows) and connect via SSH:
   ```bash
   ssh pi@raspberrypi.local
   ```
   OR
   ```bash
   ssh pi@<Raspberry_Pi_IP>
   ```
   Default password: `raspberry`

---

## Step 5: Basic Configuration
Once connected via SSH, update and secure your system:

1. Change the default password:
   ```bash
   passwd
   ```
2. Update the system:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
3. Configure the Pi:
   ```bash
   sudo raspi-config
   ```
   - **Change Hostname** (System Options → Hostname)
   - **Enable SSH and Wi-Fi** (if not already set)
   - **Expand Filesystem** (Advanced Options → Expand Filesystem)
4. Reboot the Raspberry Pi:
   ```bash
   sudo reboot
   ```

---

## Step 6: Optional Enhancements
### Set Up Static IP
To avoid IP changes on reboot:
```bash
sudo nano /etc/dhcpcd.conf
```
Add at the end:
```plaintext
interface wlan0
static ip_address=192.168.1.100/24  # Change to desired IP
static routers=192.168.1.1  # Change to router's IP
static domain_name_servers=8.8.8.8  # Google DNS or your ISP’s
```
Save (`CTRL+X`, `Y`, `Enter`), then reboot.

### Set Up SSH Key Authentication
For added security:
1. On your local machine, generate an SSH key (if not already done):
   ```bash
   ssh-keygen -t rsa -b 4096
   ```
2. Copy the key to the Raspberry Pi:
   ```bash
   ssh-copy-id pi@<Raspberry_Pi_IP>
   ```
3. Disable password authentication for SSH:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
   Set `PasswordAuthentication no`, then restart SSH:
   ```bash
   sudo systemctl restart ssh
   ```

### Install Useful Tools
```bash
sudo apt install vim htop git curl -y
```

---

## Conclusion
Your Raspberry Pi is now fully set up for headless operation! You can SSH into it anytime and start using it for various projects like a media server, home automation, or a web server.
