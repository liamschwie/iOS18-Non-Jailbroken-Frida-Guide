# 📱 iOS 18+ Non-Jailbroken Frida Instrumentation Guide

This guide documents the exact process to get **Frida** and **Objection** running on a non-jailbroken iPad/iPhone running **iOS 18+**.

## 🛑 The Problem
On iOS 17 and 18, Apple introduced stricter security measures (Hardened Runtime) that kill apps instantly if they try to debug or instrument themselves without **JIT (Just-In-Time)** compilation privileges. Standard tools like `Sideloadly`'s built-in JIT implementation are currently broken for newer iOS betas (e.g., 18.5), leading to "Developer Disk Image" errors.

## ✅ The Solution
We use a combination of:
1.  **Sideloadly** (for signing and dylib injection).
2.  **SideJITServer** (to force-enable JIT via a separate python server).
3.  **Objection** (for runtime exploration).

---

## 🛠 Prerequisites

### Hardware
* Mac (Intel or Apple Silicon).
* iPad or iPhone running iOS 17 or 18.
* USB-C / Lightning Cable.

### Files & Tools
1.  **Decrypted IPA** of your target app (e.g., Spotify).
    * *Note: You cannot use an App Store .ipa. It must be decrypted.*
2.  **Frida Gadget:** Download `frida-gadget-ios-universal.dylib` from the [Frida Releases](https://github.com/frida/frida/releases) page.
3.  **Sideloadly:** Download and install.

---

## 📝 Step 1: Install Python Tools

Apple has restricted global pip installs, and dependency mismatches are common. Run these commands to set up the environment correctly.

### 1. Install the JIT Server
    sudo pip3 install SideJITServer --break-system-packages

### 2. Fix the `ipsw-parser` crash
*Why?* The latest version of `ipsw-parser` removed a module that `SideJITServer` relies on. We must downgrade it.

    sudo pip3 install "ipsw-parser==1.3.4" --break-system-packages --force-reinstall

### 3. Install/Update Objection
    sudo pip3 install objection frida-tools --upgrade --break-system-packages

---

## 🔓 Step 2: Prepare the IPA

If your app crashes immediately upon installation, it likely contains **App Extensions** (like Watch apps or Widgets) that mess up the signing process.

1.  **Change extension:** Rename `TargetApp.ipa` -> `TargetApp.zip`.
2.  **Unzip & Clean:**
    * Open `Payload/TargetApp.app`.
    * Delete the `Watch` folder.
    * Delete the `PlugIns` folder (if present).
3.  **Re-zip:**
    * Right-click the `Payload` folder -> **Compress**.
    * Rename `Payload.zip` -> `FixedApp.ipa`.

> **[Insert Screenshot Here: Finder window showing the "Payload" folder structure]**

---

## 💉 Step 3: Sideload & Inject

1.  Open **Sideloadly**.
2.  Drag your **FixedApp.ipa** into the window.
3.  Click **Advanced Options**.
4.  Under **Inject Dylibs/Frameworks**, add your `frida-gadget-ios-universal.dylib`.
5.  **Important:** Check "Remove limitation on supported devices" if available.
6.  Click **Start**.

> **[Insert Screenshot Here: Sideloadly Advanced Options with the Frida gadget loaded]**

---

## 🚀 Step 4: The JIT Bypass

This is the critical step. Sideloadly's "Enable JIT" button might fail on iOS 18. We use `SideJITServer` instead.

### 1. Pair the Device
Connect your device via USB and run:

    sudo SideJITServer --pair

*Trust the computer on your iOS device if prompted.*

### 2. Start the Server

    sudo SideJITServer

> **[Insert Screenshot Here: Terminal showing "Server started on http://..."]**

### 3. Launch with JIT
* Keep the server running in that terminal.
* Open a **new terminal tab**.
* Run this command to launch the app (replace with your app's bundle ID):

      sudo python3 -m pymobiledevice3 developer debugserver app-launch com.spotify.client

**Success Indicator:** The app will launch on your device and **freeze** (hang) on the splash screen. This means Frida has paused execution and is waiting for you.

---

## 🕵️ Step 5: Connect Objection

1.  With the app frozen on the screen, run:

      objection --gadget "Spotify" explore

    *(Note: You can find the correct gadget name by running `frida-ps -Ua`)*.

2.  **You're in!** You should see the prompt:

      com.spotify.client on (iPad: 18.5) [usb] #

> **[Insert Screenshot Here: The successful Objection ASCII art banner and prompt]**

---

## ⚡️ Common Commands to Try

* **Disable SSL Pinning:**
    ios sslpinning disable

* **Dump Keychain:**
    ios keychain dump

* **Browse Files:**
    ls
    env

---

## ⚠️ Troubleshooting

* **"Unable to connect to Frida server":**
    * Version mismatch. Ensure your computer's `frida-tools` version matches the `frida-gadget` version you injected. Update with `pip3 install frida-tools --upgrade`.
* **"ModuleNotFoundError: No module named 'ipsw_parser.img4'":**
    * You missed Step 1.2. Run the downgrade command for `ipsw-parser`.
* **App crashes instantly (no freeze):**
    * JIT failed. Make sure `SideJITServer` is running in the background.
    * The IPA wasn't decrypted properly.
