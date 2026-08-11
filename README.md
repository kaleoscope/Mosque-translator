# Mosque Translator — Streaming Setup Guide

> 🌐 This guide is also available in [Deutsch (German)](README.de.md) and [العربية (Arabic)](README.ar.md).

This solution is meant to enable what is called as "Intranet Radio" It enables the streaming audio locally without the need of internet connection. Streaming is one through an internal network. 

Everything you need can be downloaded from this repository however here are the download links just in case:

1. https://icecast.org/download/
2. https://danielnoethen.de/butt/


This guide walks you through setting up **Icecast** (the streaming server) and **BUTT** (the tool that sends your microphone/audio to Icecast), step by step. No prior experience needed — just follow along in order. You need admin rights to configure this solution.

---

## Overview

- **Icecast** is a server that runs on your computer and broadcasts audio to listeners over the internet/network.
- **BUTT** (Broadcast Using This Tool) is a small app that captures audio (e.g. from a microphone) and streams it to your Icecast server.

You'll set up Icecast first, then connect BUTT to it.

---

## Part 1 — Install and Configure Icecast

### 1. Locate the Icecast folder

This repository already includes an `Icecast` folder with everything you need:

```
Icecast/
├── icecast.bat        ← double-click this to start the server
├── icecast.xml         ← the configuration file (passwords, ports, etc.)
└── bin/icecast.exe     ← the actual Icecast program
```

If you don't have this folder, run the installer `icecast_win64_2.5.0 (2).exe` inside the `Icecast` folder first, and it will set things up for you.

### 2. Set your password in `icecast.xml`

Before starting the server, you need to change the default passwords — this is important for security, since anyone who knows the default password (`hackme`) could connect to your server.

1. Open the `Icecast` folder.
2. Right-click **`icecast.xml`** → **Open with** → **Notepad** (or any text editor).
3. Find the `<authentication>` section. It looks like this:

   ```xml
   <authentication>
       <source-password>hackme</source-password>
       <relay-password>hackme</relay-password>
       <admin-user>admin</admin-user>
       <admin-password>hackme</admin-password>
   </authentication>
   ```

4. Replace each `hackme` with your own strong password:
   - **`source-password`** — the password BUTT will use to connect and stream audio. You'll need this in Part 2, so remember it.
  

   Example:

   ```xml
   <authentication>
       <source-password>MySecurePass123</source-password>
       <relay-password>MySecurePass123</relay-password>
       <admin-user>admin</admin-user>
       <admin-password>MyAdminPass456</admin-password>
   </authentication>
   ```

5. Check the `<hostname>`  and ` <port>8000</port>`
 value near the top of the file. It's set to `localhost` and 8000 by default. Leave it as it is. 

6. **Save the file** and close it.

### 3. Start Icecast

1. Go back to the `Icecast` folder.
2. **Double-click `icecast.bat`**.
3. A black console window will open and show Icecast starting up. Leave this window open — closing it will stop the server.
4. You should see a message like:

   ```
   Please open http://localhost:8000/ in your web browser to see the web interface.
   ```

 You might see some errors. This is fine as long as you can see the above sentence confirming that the server is running. 

5. Open your browser and go to **http://localhost:8000/**. If you see the Icecast status page, the server is running correctly.

> 💡 **Important:** Minimize this window only DO NOT close it  while you're broadcasting.

### 4. Allow Icecast Through the Firewall & Fix Microphone Echo


**A. Allow Icecast through Windows Firewall**

By default, Windows Firewall may block other devices from connecting to Icecast, even though it works fine on the same PC (`localhost`). You need to add a firewall rule to allow it.

1. Click the **Start** menu and search for **"Windows Defender Firewall"** → open it.
2. In the left sidebar, click **Allow an app or feature through Windows Defender Firewall**.
3. Click **Change settings** (top right — requires admin permission).
4. Click **Allow another app...** at the bottom.
5. Click **Browse...** and navigate to the Icecast folder, then select:
   ```
   Icecast\bin\icecast.exe
   ```
6. Click **Add**.
7. Find **icecast.exe** in the list and make sure **both** the **Private** and **Public** checkboxes are ticked (Private is enough if you're only on a home/mosque Wi-Fi network).
8. Click **OK** to save.



**If it's still blocked**, add a specific inbound port rule as well:

1. Search for **"Windows Defender Firewall with Advanced Security"** → open it.
2. Click **Inbound Rules** (left sidebar) → **New Rule...** (right sidebar).
3. Select **Port** → **Next**.
4. Select **TCP**, then choose **Specific local ports** and enter `8000` (or whatever port you set in `icecast.xml`) → **Next**.
5. Select **Allow the connection** → **Next**.
6. Leave Domain/Private/Public all checked (or just Private for a home/local network) → **Next**.
7. Give it a name, e.g. `Icecast Port 8000` → **Finish**.

Now other devices on the same network should be able to reach `http://<your-pc-ip>:8000/<mountpoint>`.


Do not procees with Step 2 before making sure that you can see Icecast status on `http://localhost:8000/`

**B. Turn off "Listen to this device" on your microphone**

If you hear your own voice echoing back through your speakers while streaming, Windows may be routing your microphone input directly to your speakers/headphones ("monitoring"). Turn this off:

1. Right-click the **speaker icon** in the Windows taskbar (bottom-right) → **Sounds** (or search **"Sound settings"** in the Start menu).
2. Go to the **Recording** tab.
3. Right-click your microphone (the one selected in BUTT) → **Properties**.
4. Go to the **Listen** tab.
5. **Uncheck** the box labeled **"Listen to this device"**.
6. Click **Apply**, then **OK**.

This stops Windows from playing your mic input out loud on your PC — BUTT will still capture and stream it normally, but you won't hear an echo locally.

---

## Part 2 — Install and Configure BUTT

### 1. Install BUTT

1. Locate the file **`butt-1.47.0_Win64_Portable.zip`** in this project.
2. Right-click it → **Extract All...** and choose a folder (e.g. `Desktop\butt`).
3. Open the extracted folder and double-click **`butt.exe`** to launch it.

   This is a portable version, so there's nothing else to install — it just runs.

### 2. Configure BUTT to connect to your Icecast server

1. In BUTT, click the **Settings** button (gear/wrench icon).
2. Go to the **Main** tab and click **Add** (to add a new server).
3. Fill in the server details to match your `icecast.xml`:

   | Field | Value |
   |---|---|
   | **Type** | Icecast |
   | **Address** | `localhost` (or your computer's IP if streaming to other devices on the network) |
   | **Port** | `8000` |
   | **Password** | the `source-password` you set in `icecast.xml` (originally `hackme`)|
   | **Mountpoint** | e.g. `/stream` (remember this — listeners will use it, e.g. `http://localhost:8000/stream`) |
   | **Icecast/Shoutcast user** | `source` |

4. Click **OK** to save the server entry.

### 3. Configure the audio input

1. Still in **Settings**, go to the **Audio** tab.
2. Under **Device**, select your microphone or audio input source (e.g. "Microphone (Realtek Audio)" or a virtual audio cable if you're capturing system audio).
3. Choose a **Codec** `MP3` and  **Bitrate** `64K` for both rows. this seemed to work best with us.
4. Click **OK** to close Settings.

### 4. Start streaming

1. On the main BUTT window, make sure your server is selected in the dropdown.
2. Click the **PLAY** button (▶) to start streaming.
3. If everything is configured correctly, BUTT will connect and you'll see a green "connected" indicator along with live audio levels, with time counter showing showing.

### 5. Verify the stream

1. Go back to **http://localhost:8000/** in your browser.
2. Under "Mount Points," you should now see your active mountpoint (e.g. `/stream`) listed as live.
3. Click on it, or open `http://localhost:8000/<mountpoint>` directly (e.g. `http://localhost:8000/stream`) to listen to the stream.

### 6. Connect from other devices on the network

`localhost` only works on the same PC that's running Icecast. To let other devices (phones, laptops, tablets) listen, you need to find your PC's local network address (IPv4) and share that instead.

1. On the PC running Icecast, open **Command Prompt** (search "cmd" in the Start menu).
2. Type the following and press **Enter**:

   ```
   ipconfig
   ```

3. Look for the section matching your active connection (usually **Wireless LAN adapter Wi-Fi** or **Ethernet adapter Ethernet**).
4. Find the line labeled **IPv4 Address**. It will look something like:

   ```
   IPv4 Address. . . . . . . . . . . : 192.168.1.25
   ```

   This is your PC's address on the local network. Write it down.

5. On any other device connected to the **same Wi-Fi/network**, open a web browser and go to:

   ```
   http://<IPv4-address>:8000/<mountpoint>
   ```

   For example, if your IPv4 address is `192.168.1.25` and your mountpoint is `/stream`:

   ```
   http://192.168.1.25:8000/stream
   ```

6. The stream should start playing. You can also visit `http://<IPv4-address>:8000/` to see the full Icecast status page from that device.

> 💡 **Tip:** The IPv4 address can change if your router reassigns it later (especially after a restart). If the stream stops being reachable, repeat steps 1–4 to check whether the address changed.

---

## Quick Troubleshooting

- **BUTT won't connect / "Connection failed":**
  - Double-check that Icecast is running (the `icecast.bat` window is still open).
  - Make sure the password in BUTT matches `source-password` in `icecast.xml` exactly.
  - Confirm the port (`8000` by default) matches what's in `icecast.xml`.

- **No sound / silence on the stream:**
  - Check the correct microphone/input device is selected in BUTT's Audio settings.
  - Make sure the input isn't muted in Windows sound settings.

- **Can't reach the stream from another device:**
  - Use your computer's local IP address (e.g. `192.168.x.x`) instead of `localhost`.
  - Make sure your firewall allows connections on port `8000`.

---

## Summary

1. Set a strong `source-password` (and other passwords) in `Icecast/icecast.xml`.
2. Double-click `Icecast/icecast.bat` to start the server.
3. Extract and run BUTT, then add a server pointing to `localhost:8000` with your `source-password`.
4. Pick your microphone in BUTT's Audio settings and hit ▶ to go live.
5. Listen at `http://localhost:8000/<mountpoint>`.
