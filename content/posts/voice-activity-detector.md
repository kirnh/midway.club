---
date: 2025-03-28
title: 🧠 Real-Time Voice Activity Detection on Raspberry Pi with Silero VAD
---

In this guide, we implement **real-time voice activity detection (VAD)** on a Raspberry Pi using the open-source [Silero VAD](https://github.com/snakers4/silero-vad) model. This allows us to listen continuously and trigger further actions **only when human speech is detected**, even in noisy environments.

---

## 🔥 Why Real-Time VAD Matters

When building smart voice systems, it's wasteful (and error-prone) to analyze audio all the time. We want to:

- ✅ Detect when someone is **actually speaking**
- ❌ Ignore background noise, silence, fans, dogs barking, etc.
- ✅ Trigger speaker recognition or transcription **only when voice is present**

---

## 🛠️ What We’ll Use

| Component     | Purpose                              |
|---------------|---------------------------------------|
| `arecord`     | Efficient audio capture on RPi        |
| `torchaudio`  | WAV file loading                      |
| `silero-vad`  | Tiny PyTorch-based voice detector     |
| Raspberry Pi  | Edge device to run everything locally |

---

## ⚙️ Setup

### Install Dependencies

```bash
pip install torchaudio soundfile silero-vad
sudo apt install sox alsa-utils
```

> ✅ Tip: Make sure your microphone shows up in `arecord -l`. Use the correct device name (e.g. `plughw:1,0`).

---

## 📄 The Python Script: `realtime_vad.py`

This script:
1. Continuously records 2-second chunks of audio
2. Analyzes each chunk for speech
3. Prints whether speech is present
4. Loops indefinitely

```python
# realtime_vad.py

import torch
from silero_vad import VoiceActivityDetector
import torchaudio
import subprocess
import os
import time

SAMPLE_RATE = 16000
DURATION = 2  # seconds per chunk
TEMP_FILE = "temp_vad.wav"
DEVICE = "plughw:1,0"  # adjust based on `arecord -l`

# Load Silero VAD once
print("🔁 Loading Silero VAD model...")
model, utils = torch.hub.load(repo_or_dir='snakers4/silero-vad', model='silero_vad', trust_repo=True)
(get_speech_timestamps, _, read_audio, _, _) = utils

def record_wav(filename=TEMP_FILE, duration=DURATION, device=DEVICE):
    subprocess.run([
        "arecord",
        "-D", device,
        "-f", "S16_LE",
        "-r", str(SAMPLE_RATE),
        "-c", "1",
        "-t", "wav",
        "-d", str(duration),
        filename
    ], stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)

def is_speech_present(filename=TEMP_FILE):
    wav = read_audio(filename, sampling_rate=SAMPLE_RATE)
    speech = get_speech_timestamps(wav, model, sampling_rate=SAMPLE_RATE)
    if speech:
        total_speech = sum([s["end"] - s["start"] for s in speech]) / SAMPLE_RATE
        print(f"🗣️ Speech detected: {total_speech:.2f} seconds")
        return True
    else:
        print("🔇 Silence or noise only.")
        return False

def main():
    print("🎙️ Real-Time Voice Activity Detection Started (Ctrl+C to stop)...")
    try:
        while True:
            record_wav()
            is_speech_present()
            if os.path.exists(TEMP_FILE):
                os.remove(TEMP_FILE)
            time.sleep(0.5)
    except KeyboardInterrupt:
        print("\n🛑 Exiting.")

if __name__ == "__main__":
    main()
```

---

## ✅ Example Output

```bash
🎙️ Real-Time Voice Activity Detection Started (Ctrl+C to stop)...
🔇 Silence or noise only.
🗣️ Speech detected: 1.58 seconds
🔇 Silence or noise only.
🗣️ Speech detected: 2.00 seconds
```

---

## 🧠 How It Works

- Silero VAD is a neural network trained to distinguish **voice vs noise**, even in difficult conditions.
- The model runs fast on a Raspberry Pi (no GPU needed).
- We record using `arecord` and process with `torchaudio`.

---

## 🧪 Use Cases

- ✅ Smart voice-triggered assistants
- ✅ Voice-controlled door locks
- ✅ Real-time transcription only when speech is detected
- ✅ Edge-powered speaker recognition

---

## 🔁 Next Steps

You can integrate this directly with speaker recognition like this:

```python
if is_speech_present():
    run_speaker_recognition()
else:
    print("⏭️ Skipping – no voice detected.")
```

---

## ✅ Conclusion

Silero VAD enables Raspberry Pi projects to **detect human voice in real-time**, reliably, and with very little resource usage. This is a foundational tool for anyone building **smart voice interfaces** in real-world environments.

---

**Midway Labs** – Building mindful voice tech for the real world.