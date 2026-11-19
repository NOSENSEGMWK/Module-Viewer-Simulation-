<p align="center">
  <img src="docs/logo.svg" width="128" alt="Module Viewer Simulation logo">
</p>

<h1 align="center">Module Viewer Simulation</h1>

<p align="center">
  <b>Inspect, analyze and simulate root modules without installing them.</b><br>
  Version <b>1.0.0</b> · codename <b>R</b> · Android &amp; Windows · 🌐 English / Indonesia
</p>

<p align="center">
  <a href="https://github.com/NOSENSEGMWK/Module-Viewer-Simulation-/releases/latest">⬇️ Download</a> ·
  <a href="#features">Features</a> ·
  <a href="#how-to-use">How to use</a> ·
  <a href="#windows-edition-100-w">Windows</a> ·
  <a href="#limitations--notes">Limitations</a> ·
  <a href="README.md">🇮🇩 Bahasa Indonesia</a>
</p>

<p align="center">
  Made by <b>Towa</b> with AI (<b>Claude</b>)
</p>

---

## About

Installing an unknown root module is risky: a single `rm -rf` or a `dd` to the wrong partition can leave a device that
no longer boots. **Module Viewer Simulation (MVS)** lets you *look first* at what a module does before you install it for real.
Every module script runs only inside an in-memory simulated mini shell inside the app.

- ✅ **No root needed**
- ✅ **Nothing runs on your device or PC**: modules are only extracted to the app's private storage
- ✅ **No internet**: the app has no internet access of its own, and a module's WebUI runs with the network blocked
- ✅ **No external dependencies**: the UI and the simulator are plain HTML/JavaScript inside a WebView
- ✅ **Two languages**: English and Bahasa Indonesia, switchable any time in Settings

## Features

### 📦 Inspect a module
| Feature | Details |
|---|---|
| **Pick a module `.zip`** | Supports Magisk / KernelSU / APatch module format (needs `module.prop`). Zip size limit 350 MB, with *zip-slip* protection and a limit on the extracted size. |
| **Info card** | Name, version, author, description, banner, size, file count, and **WEBUI** / **ACTION** badges. |
| **Action button** | A module that has an `action.sh` gets an **Action** button to the left of the WebUI button, like in a root manager. The script runs in the simulator with terminal-style output and an exit code. |
| **Installation process (simulation)** | A KSU-style terminal log: `ui_print`/`echo` from `customize.sh` are shown, other commands are marked `[SIM]`, and dangerous lines get a risk label. Nothing is really run. |
| **View contents** | Full file tree + a line-numbered text viewer. Binary files are marked, and ELF files can be opened in the analyzer. |
| **APK icon** | `.apk` files inside a module show their own icon and label. |

### 🛡️ Static security analysis
Scans scripts for dangerous patterns without executing them, then gives a **score and a risk level**
(*safe · review · low · medium · high · critical*):

- Destructive operations on **partitions** (`dd`, `flash_image`, `fastboot`, format, writes to `/dev/block/...`), with per-partition impact (bootloader, boot, modem, nvram, system, …)
- Deleting / changing permissions of important directories (`rm -rf`, `chmod -R`, `find -delete`, `chown`)
- Download-and-run (`curl … | sh`), possible *reverse shells*, raw network connections
- Obfuscation: `base64 -d`, `eval`, hex runs, command names built from variables, sensitive words split apart
- Fork bombs, `setenforce 0`, `remount rw`, `pm install/uninstall`, `reboot`
- **Variable tracking**: `X=/system; rm -rf "$X"` is still caught because the variable value is resolved
- WebUI code (`.js`/`.html`) is scanned too

### ⚙️ Native Binary Analyzer
An **ELF** reader for binaries inside a module (ARM64, ARM32, x86, x86-64), opened from the file list or from an analysis finding.

### 🌐 Module WebUI
The module's own WebUI opens **inside the app** with a fake `ksu.*` API:
`exec` / `spawn` (simulated by the mini shell), `moduleInfo`, `toast`, `fullScreen`, plus KernelSU-style installed-app APIs:
`listPackages`, `listUserPackages`, `listSystemPackages`, `listAllPackages`, `getPackagesInfo`, `getPackagesIcons`, and app icons through `ksu://icon/<package>`.
The network is blocked. The **←** button in the top-left corner goes back.

### 📱 Android simulation
A fake device whose values can be **read and written** through virtual `/sys` and `/proc`, including CPU (clusters, governors, frequencies), GPU, thermal, battery, memory, system properties, and `dumpsys` / `getprop` / `settings`.

| Chipset | Status |
|---|---|
| **MediaTek Helio G99 (MT6789)**, Mali-G57 | Default |
| **Qualcomm Snapdragon 695 5G (SM6375)**, Adreno 619 | <img alt="BETA" src="https://img.shields.io/badge/-BETA-d32f2f?style=flat-square"> |

You can choose the simulated root manager (**KernelSU / Magisk / APatch**), the device name and the codename.

### 🔁 Daemon simulation <img alt="BETA" src="https://img.shields.io/badge/-BETA-d32f2f?style=flat-square">
`post-fs-data.sh`, `service.sh` and `boot-completed.sh` run as **real processes in the simulator's scheduler** on **virtual time**:

- A realistic boot sequence, `system.prop` applied, and `sys.boot_completed` turning `1` at the right moment
- Daemons keep running: `while true; do …; sleep N; done &`, `nohup`, `setsid`, `( … ) &`, loop functions, pidfiles
- Processes & signals: `$!`, `$$`, `pidof`, `pgrep`, `pkill`, `kill`, `trap`, `wait`, `ps`, `top`, `/proc/<pid>`
- Triggers: `inotifyd`, the foreground app (`dumpsys window`), screen on/off, charger state
- Controls: time speed 1x/10x/60x/600x, time skip, pause, reboot, per-process logs, history of exited processes, logcat, *busy-loop* warning
- Can be turned off with the **Simulate daemons?** setting

### 🌍 Language & updates
- **English / Bahasa Indonesia** in Settings, applied instantly across the whole app (UI, analysis results, install log, daemon screen, and the Native Binary Analyzer). A module's own name, description, file contents and script output are intentionally not translated.
- **Check for updates**: Settings has buttons that open the [project page](https://github.com/NOSENSEGMWK/Module-Viewer-Simulation-) and the [latest release](https://github.com/NOSENSEGMWK/Module-Viewer-Simulation-/releases/latest) in your browser. The app itself does not connect to the internet.

## How to use

### Android
1. Download the latest APK from **[Releases](https://github.com/NOSENSEGMWK/Module-Viewer-Simulation-/releases/latest)** and install it (Android 8.0 / API 26 or newer; allow installs from this source if asked).
2. Open the app → **Select Module** → choose a module `.zip` file.
3. Use **View contents**, **Installation process**, **Analysis results**, **Daemon simulation**, **Action** (when `action.sh` exists) and **Open** (WebUI).
4. The ⚙️ icon in the top-right corner opens Settings (language, chipset, root manager, daemon simulation, device name, check for updates).

### Windows edition 1.0.0 (W)
Available in **[Releases](https://github.com/NOSENSEGMWK/Module-Viewer-Simulation-/releases/latest)** as `ModuleViewerSim_Windows_1.0.0-W.zip`.

1. double-click `ModuleViewerSim.exe` (it opens through Microsoft Edge in app mode).
2. Press **Select Module** (or `Ctrl+O`), or **drag a `.zip` file** onto the window.
3. The layout follows the window size (minimum 420 × 520). Shortcuts: `Esc` / `Alt+←` / the mouse "back" button to go back.

Requirements: Windows 10/11 64-bit, a recent Microsoft Edge or Google Chrome. The `.exe` is not signed by a publisher, so SmartScreen may show a warning ("More info" → "Run anyway").

## Limitations & notes

- **The analysis is text-pattern matching, not a security guarantee.** A "safe" result does not mean a module is definitely safe; read a module's code before installing it for real.
- **The mini shell is not real Android.** The behavior of some commands, sensor values and frequency tables is simulated data.
- **Snapdragon 695 and the daemon simulation are still BETA**: the frequency/level tables and some nodes are reasonable approximations, not dumps from a real device. Bug reports help a lot.
- **Native ELF binaries are not executed.** A binary started in the background only shows up as a process labeled "native ELF".
- **App icons in the WebUI are generic** (a letter + a color per package), not the real app logos.
- **The English translation** works by matching the displayed text against a dictionary; text that is not in the dictionary stays in Indonesian. Translation fixes are very welcome.
- On Windows, a module WebUI split into many ES-module files (`import './x.js'`) may not run; single-file / single-bundle WebUIs run normally.

## Built with

- **Android:** Kotlin + WebView; the UI and the simulator (mini shell, process scheduler, virtual filesystem) are plain HTML/JavaScript
- **Windows:** a single standalone HTML file + a tiny `.exe` launcher (PE64) that opens Edge in app mode; a module's WebUI runs in a sandboxed `iframe` with the network blocked
- **Build:** GitHub Actions produces the release APK and the Windows package automatically

## Credits

Made by **Towa** with AI (**Claude**, an AI model by Anthropic). Design, direction and testing by Towa; the code was developed together with Claude.

## Disclaimer

This project is not affiliated with, or endorsed by, Magisk, KernelSU, APatch, MediaTek, Qualcomm, Google or Microsoft.
All brand names and trademarks belong to their respective owners and are mentioned only to describe compatibility.
Use at your own risk. This app inspects and simulates; it does **not** guarantee the safety or success of installing a module.
