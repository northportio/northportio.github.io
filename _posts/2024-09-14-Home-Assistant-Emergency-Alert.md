---
layout: post
title: "Home Assistant Fire/EMS Alert Automation"
date: 2024-09-14 18:00:00 -0400
categories: homeassistant
tags: fire ems tones automation alerts python homeassistant homekit lights
image:
  path: /assets/img/web/fire-alert.jpg
---

# How to get Home Assistant to know when I have to respond to a call?
Many volunteer first responders know that getting to a call as quickly and smoothly as possible is of the upmost importance. Often times with volunteer agencies, you can be home going about your day when you get an audible alert on your pager that there is a call you have to respond to. This sometimes includes in the middle of the night. However, I wanted to incorporate a visual alert as well. I use [Home Assistant](https://www.home-assistant.io/) for automating many things throughout my home and I have smart lights in about every room. I thought it would be great if I can have my lights flash whenever a call comes through on my pager. So this is how it got started.

## What are the ways Home Assistant can get alerts about a new call?

There are a couple ways of going about this:
1. Leveraging an API
   - Some (not all) departments leverage an iOS/Android application to alert members of a call. This way, Home Assistant can constantly check that API for changes. However, not all apps have an API for members. This also requires an internet connection, so if the power, cloud, or internet is out, the automation will not work.
2. Levraging the same radio communications the pager already listens to
   - This can be a bit more complex, however, is the most reliable in internet, cloud, and power outages (if you have a home generator).

I decided to go with option 2.

## 🛠️ Parts Needed
- Home Assistant: does not matter the installation type
- Raspberry Pi 4
- RTL-SDR
- Antenna for RTL-SDR
- Optional - Programing cradle for your pager (just to easily read the tones)

## How do the pagers work?

We will need to understand this piece so we know how to setup the RTL-SDR. 

I can't speak for every department but of the ones that I know, this is how it works.

The pagers are always listening on a specific frequency for a set of tones. These tones tell the pager to turn on the speaker for the call (ex: Signal X for a fire at X address). For this we will need some info from [RadioReference.com](https://www.radioreference.com/db/browse/) or reading from the pager itself via the programming cradle.
For Example:
```
Frequency = 483.800 MHz
Main Alert Tone 1: 900.3Hz
Alert Tone 2: 950.5Hz
```

For reverence, [this](https://www.youtube.com/watch?v=jWqy05IgQGs) is what it sounds like.

Once you have this info, this is where the Raspberry Pi, RTL-SDR, and a little python comes in. I used ChatGPT to get this to work flawlessly.

---

### **Overview**

We'll use the following tools:

- **`rtl_fm`**: Command-line tool to receive FM radio signals using RTL-SDR.
- **Python**: For processing the audio stream and detecting the tones.
- **Goertzel Algorithm**: Efficient algorithm for detecting specific frequencies in a signal.
- **`requests`**: Python library to send HTTP requests to your Home Assistant webhook.

### **Prerequisites**

1. **Raspberry Pi 4** running Raspbian (Debian 12-based).
2. **RTL-SDR** dongle connected to the Raspberry Pi.
3. **Local Network Connection** on the Raspberry Pi to send webhooks.

### **Step-by-Step Guide**

#### **1. Install Necessary Packages**

Open a terminal on your Raspberry Pi and run:

```bash
sudo apt-get update
sudo apt-get install -y rtl-sdr sox python3-pip
pip3 install numpy requests
```
Note, to get this part to work, you may have to activate a python virtual environment.

#### **2. Test Your RTL-SDR Dongle**

Ensure your RTL-SDR dongle is recognized:

```bash
rtl_test
```

You should see output indicating that the device was found.

#### **3. Create the Python Script**

Create a file named `detect_tones.py`:

```bash
nano detect_tones.py
```

Paste the following code into the file:

```python
import sys
import numpy as np
import requests
import time

# Configuration
SAMPLE_RATE = 22050  # Sampling rate
BLOCK_SIZE = 1024    # Number of samples per block
TONE1_FREQ = 900.3   # Frequency of the first tone in Hz
TONE2_FREQ = 950.5  # Frequency of the second tone in Hz
TONE_THRESHOLD = 1000000  # Magnitude threshold for tone detection (adjust as needed)
MAX_TONE_GAP = 5     # Maximum time gap between tones in seconds

# Webhook URL (replace with your actual webhook URL)
WEBHOOK_URL = 'https://your_home_assistant_url/api/webhook/your_webhook_id'

def goertzel(samples, target_freq, sample_rate):
    num_samples = len(samples)
    k = int(0.5 + ((num_samples * target_freq) / sample_rate))
    omega = (2.0 * np.pi * k) / num_samples
    sine = np.sin(omega)
    cosine = np.cos(omega)
    coeff = 2 * cosine
    q1 = 0.0
    q2 = 0.0
    for sample in samples:
        q0 = coeff * q1 - q2 + sample
        q2 = q1
        q1 = q0
    real = (q1 - q2 * cosine)
    imag = (q2 * sine)
    magnitude = np.sqrt(real * real + imag * imag)
    return magnitude

def trigger_webhook():
    try:
        response = requests.post(WEBHOOK_URL)
        if response.status_code == 200:
            print('Webhook triggered successfully')
        else:
            print(f'Webhook trigger failed: {response.status_code}')
    except Exception as e:
        print(f'Error triggering webhook: {e}')

def main():
    state = 0  # 0 = waiting for tone1, 1 = waiting for tone2
    tone1_detected_time = None

    while True:
        data = sys.stdin.buffer.read(BLOCK_SIZE * 2)
        if not data or len(data) < BLOCK_SIZE * 2:
            continue
        samples = np.frombuffer(data, dtype=np.int16)

        mag1 = goertzel(samples, TONE1_FREQ, SAMPLE_RATE)
        mag2 = goertzel(samples, TONE2_FREQ, SAMPLE_RATE)

        if state == 0:
            if mag1 > TONE_THRESHOLD:
                print(f'Tone 1 detected with magnitude {mag1}')
                state = 1
                tone1_detected_time = time.time()
        elif state == 1:
            if mag2 > TONE_THRESHOLD:
                print(f'Tone 2 detected with magnitude {mag2}')
                # Check if tone2 detected within the allowed time window
                if time.time() - tone1_detected_time < MAX_TONE_GAP:
                    print('Both tones detected in sequence. Triggering webhook...')
                    trigger_webhook()
                # Reset state
                state = 0
                tone1_detected_time = None
            else:
                # If time since tone1 is too long, reset
                if time.time() - tone1_detected_time > MAX_TONE_GAP:
                    print('Timeout waiting for Tone 2. Resetting...')
                    state = 0
                    tone1_detected_time = None

if __name__ == '__main__':
    main()
```

**Important:** Replace `'http://your_home_assistant_ip/api/webhook/your_webhook_id'` with your actual Home Assistant webhook URL.

#### **4. Adjust Thresholds and Parameters**

- **`TONE_THRESHOLD`**: This value may need adjustment based on your environment. Start with `1000000` and adjust as needed.
- **`MAX_TONE_GAP`**: Maximum time in seconds between the two tones. Adjust if your tones are spaced differently.

#### **5. Make the Script Executable**

```bash
chmod +x detect_tones.py
```

#### **6. Run the Script**

Use the following command to start the script:

```bash
rtl_fm -f 460.3625M -M fm -s 22050 -A fast -l 0 -E deemp | python3 detect_tones.py
```

**Explanation of the `rtl_fm` parameters:**

- `-f 460.3625M`: Sets the frequency to 460.3625 MHz.
- `-M fm`: Sets the modulation to narrowband FM.
- `-s 22050`: Sets the sample rate to 22,050 Hz.
- `-A fast`: Uses fast auto gain control.
- `-l 0`: Sets squelch level to 0 (adjust if needed).
- `-E deemp`: Applies de-emphasis filter to audio.

#### **7. Monitor the Output**

The script will output messages when tones are detected and when the webhook is triggered. Use these messages to adjust `TONE_THRESHOLD` as needed.

#### **8. Run the Script on Startup (Optional)**

If you want the script to run continuously, you can set it up as a service or add it to your crontab:

```bash
crontab -e
```

Add the following line to run the script at reboot:

```bash
@reboot rtl_fm -f 483.800M -M fm -s 22050 -A fast -l 0 -E deemp | python3 /path/to/detect_tones.py
```

---

### **Notes and Tips**

- **Testing Thresholds**: You may need to adjust `TONE_THRESHOLD` by observing the magnitude values when the tones are present versus when they are not.
- **Logging**: Consider logging the magnitudes to a file for analysis.
- **Squelch Level**: Adjusting the `-l` parameter in `rtl_fm` can help filter out background noise.
- **Audio Quality**: Ensure your antenna and RTL-SDR dongle are receiving clear signals for accurate tone detection.

---

### **Understanding the Code**

- **Goertzel Algorithm**: Efficient for detecting specific frequencies within a block of samples.
- **State Machine**: The script uses a simple state machine to track whether it's waiting for the first or second tone.
- **Webhook Trigger**: When both tones are detected in sequence within the specified time window, the webhook is triggered.

---

### **Troubleshooting**

- **No Tones Detected**: Check your RTL-SDR setup and ensure that you can receive signals at the specified frequency.
- **False Positives**: If the script triggers the webhook unexpectedly, you may need to increase `TONE_THRESHOLD`.
- **Script Crashes**: Ensure that all dependencies are installed and that the RTL-SDR dongle is properly connected.

---

### **Security Considerations**

- Ensure your webhook URL is kept secure, especially if it's accessible over the internet.
- Consider using HTTPS and proper authentication methods for your Home Assistant instance. If the Raspberry Pi and the Home assistant instance is on the same network, this isn't really required.

---

### **Conclusion**

By following the steps above, you should have a working system that listens for specific tones and triggers a webhook to your Home Assistant instance when those tones are detected. This setup leverages the capabilities of the RTL-SDR dongle and the processing power of your Raspberry Pi to create a custom alerting system for your fire department pages.

Once you have this done, you can configure an automation to do whatever you like. You can even add the function to only perform the operation when you mark yourself as on duty with home Assistant via a virtual switch.
