---
Date: 2025-03-14
Title: Streaming Audio from Raspberry Pi to iPad via Bluetooth
---

## Overview
This guide explains how to:
1. **Capture audio from a microphone connected to the Raspberry Pi.**
2. **Stream the audio data to the iPad over Bluetooth.**
3. **Ensure the audio stream automatically resumes when the Bluetooth connection is re-established.**

This setup guarantees a **continuous, automated audio stream** from the Pi to the iPad, even if the Bluetooth connection is temporarily lost.

---

## **1. Capturing and Streaming Audio to the iPad**

Since Bluetooth is already connected, we need to **capture and stream microphone audio** while ensuring that the stream resumes automatically if the connection is lost.

### **Step 1: Install Required Packages**
```bash
sudo apt update
sudo apt install bluez bluez-tools sox alsa-utils pulseaudio pulseaudio-module-bluetooth
```

### **Step 2: Enable Bluetooth Audio Support**
Ensure the Bluetooth audio module is loaded:
```bash
pactl load-module module-bluetooth-discover
```
To make it load at boot, add this line to `/etc/pulse/default.pa`:
```bash
load-module module-bluetooth-discover
```
Restart PulseAudio:
```bash
pulseaudio --start
```

### **Step 3: Create the Audio Streaming Script**
This script will continuously **check for a Bluetooth connection** and only start streaming **when the iPad is connected**.

#### **Create `stream_audio.sh`**
```bash
#!/bin/bash

IPAD_MAC="B8:49:6D:72:DA:44"  # Replace with your iPad's MAC Address

# Function to check if the iPad is connected
is_connected() {
    bluetoothctl info "$IPAD_MAC" | grep -q "Connected: yes"
}

echo "🎤 Waiting for iPad connection to stream audio..."

while true; do
    if is_connected; then
        echo "🔊 iPad connected! Starting audio stream..."
        arecord -D plughw:1,0 -f dat | aplay -D bluealsa
    else
        echo "❌ iPad not connected. Retrying..."
    fi
    sleep 5  # Wait before checking connection again
done
```

#### **Make the Script Executable**
```bash
chmod +x /home/pi/stream_audio.sh
```

### **Step 4: Run the Audio Streaming Script on Boot**
```bash
sudo nano /etc/systemd/system/stream_audio.service
```

Paste the following:
```ini
[Unit]
Description=Bluetooth Audio Streaming to iPad
After=bluetooth.target

[Service]
ExecStart=/bin/bash /home/pi/stream_audio.sh
Restart=always
User=pi

[Install]
WantedBy=default.target
```

Enable and start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable stream_audio.service
sudo systemctl start stream_audio.service
```

Check if the service is running:
```bash
sudo systemctl status stream_audio.service
```

---

## **2. Testing the Setup**
1. **Restart the Raspberry Pi:**
   ```bash
   sudo reboot
   ```
2. **Ensure the audio streaming service is running:**
   ```bash
   sudo systemctl status stream_audio.service
   ```
3. **Turn the iPad’s Bluetooth off and back on.**
   - The script should **automatically resume audio streaming** once the iPad reconnects.

---

## **3. Conclusion**
With this setup, your Raspberry Pi will:
✅ **Continuously stream microphone audio over Bluetooth.**  
✅ **Automatically resume streaming when the iPad reconnects.**  
✅ **Ensure the service runs automatically on boot.**  

This guarantees a **seamless, hands-free Bluetooth audio streaming experience** from your Raspberry Pi to your iPad. 🚀

If you run into issues, check logs using:
```bash
journalctl -xeu stream_audio.service
```

Let me know if you need further refinements! 🎤📡