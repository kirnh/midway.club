---
Date: 2025-03-14
title: Persistent Bluetooth Connection Between Raspberry Pi and iPad
---

## Overview
This guide walks you through the process of:
1. **One-time pairing of the Raspberry Pi with the iPad.**
2. **Setting up a Bash script to maintain a persistent Bluetooth connection.**
3. **Running the script automatically on boot using `systemd`.**

With this setup, the Raspberry Pi will continuously monitor the Bluetooth connection and automatically reconnect to the iPad if disconnected.

---

## **1. One-Time Bluetooth Pairing**

Before setting up the automatic connection, you need to **pair your iPad with the Raspberry Pi manually**.

### **Step 1: Enable Bluetooth on Both Devices**
Ensure Bluetooth is enabled on both the Raspberry Pi and iPad.

### **Step 2: Start Bluetoothctl on Raspberry Pi**
Run:
```bash
bluetoothctl
```

### **Step 3: Put Bluetooth in Pairing Mode**
Inside `bluetoothctl`, enable scanning and make the Raspberry Pi discoverable:
```bash
power on
agent on
default-agent
scan on
```

### **Step 4: Find the iPad’s MAC Address**
Look for a device named `iPad` in the scan results. It will have a MAC address like this:
```
[NEW] Device B8:49:6D:72:DA:44 iPad
```
Take note of the **MAC address** (`B8:49:6D:72:DA:44` in this example).

### **Step 5: Pair with the iPad**
```bash
pair B8:49:6D:72:DA:44
trust B8:49:6D:72:DA:44
connect B8:49:6D:72:DA:44
```
Replace `B8:49:6D:72:DA:44` with your actual iPad MAC address.

Once the pairing is successful, the Raspberry Pi will remember the iPad for future connections.

Exit `bluetoothctl`:
```bash
exit
```

---

## **2. Persistent Bluetooth Connection Script**
Once paired, we need to ensure that the Raspberry Pi **automatically reconnects** if the Bluetooth connection is lost.

### **Step 1: Create the Connection Script**

Create a new script:
```bash
nano /home/pi/connect_bluetooth.sh
```

Paste the following script:
```bash
#!/bin/bash

IPAD_MAC="B8:49:6D:72:DA:44"  # Replace with your iPad's MAC Address

# Function to check if the iPad is connected
is_connected() {
    bluetoothctl info "$IPAD_MAC" | grep -q "Connected: yes"
}

echo "🔄 Monitoring Bluetooth connection to iPad ($IPAD_MAC)..."

while true; do
    if is_connected; then
        echo "✅ iPad is connected."
    else
        echo "⚠️ iPad is disconnected. Attempting to reconnect..."
        bluetoothctl connect "$IPAD_MAC"
    fi
    sleep 5  # Check every 5 seconds
done
```

### **Step 2: Make the Script Executable**
```bash
chmod +x /home/pi/connect_bluetooth.sh
```

---

## **3. Running the Script on Boot with systemd**

To ensure the Bluetooth connection script **runs automatically at startup**, we use `systemd`.

### **Step 1: Create a systemd Service File**
```bash
sudo nano /etc/systemd/system/ipad_bluetooth.service
```

Paste the following:
```ini
[Unit]
Description=Bluetooth Auto-Reconnect to iPad
After=bluetooth.target

[Service]
ExecStart=/bin/bash /home/pi/connect_bluetooth.sh
Restart=always
User=pi

[Install]
WantedBy=default.target
```

### **Step 2: Enable and Start the Service**
```bash
sudo systemctl daemon-reload
sudo systemctl enable ipad_bluetooth.service
sudo systemctl start ipad_bluetooth.service
```

### **Step 3: Verify the Service**
Check if the service is running:
```bash
sudo systemctl status ipad_bluetooth.service
```
You should see output confirming that the service is **active and running**.

---

## **4. Testing the Setup**
1. **Restart the Raspberry Pi:**
   ```bash
   sudo reboot
   ```
2. **Ensure the Bluetooth connection is active:**
   ```bash
   sudo systemctl status ipad_bluetooth.service
   ```
3. **Turn the iPad’s Bluetooth off and back on.**
   - The script should **automatically reconnect** when Bluetooth is turned back on.

---

## **5. Conclusion**
With this setup, your Raspberry Pi will:
✅ **Automatically reconnect to the iPad when Bluetooth is lost.**  
✅ **Ensure the Bluetooth connection is persistent and stable.**  
✅ **Run the connection script automatically on startup.**  

This ensures a **hands-free** and **automated** Bluetooth connection between your Raspberry Pi and iPad. 🚀

If you run into issues, check logs using:
```bash
journalctl -xeu ipad_bluetooth.service
```
