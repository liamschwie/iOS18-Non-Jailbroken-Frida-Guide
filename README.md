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
```bash
sudo pip3 install SideJITServer --break-system-packages
