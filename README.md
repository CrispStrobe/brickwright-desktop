Here is a professional, updated **README.md** tailored for your specific fork. It highlights the LEGO capabilities and, crucially, explains the unique "side-by-side" folder structure required to build it.

You can copy and paste this directly into your repository.

---

# Custom TurboWarp Desktop (LEGO Edition)

**A specialized build of TurboWarp Desktop featuring advanced, unsandboxed extensions for LEGO hardware.**

**[⬇️ Download Latest Release](https://www.google.com/search?q=https://github.com/CrispStrobe/turbowarp-desktop/releases)**

---

## 🧱 Features

This version includes exclusive extensions not available in the standard TurboWarp Desktop:

* **LEGO EV3 (Mindstorms):** Direct control via Bluetooth/USB and NXC transpilation.
* **LEGO NXT:** Full support for motors, sensors, and screen drawing.
* **LEGO Spike Prime / Robot Inventor:** Advanced control via BLE.
* **LEGO WeDo 2.0 & Boost:** Unified support.
* **Utilities:** Gamepad support, advanced array manipulation, and CSP solvers.

These extensions run **unsandboxed**, allowing direct communication with hardware via Bluetooth (BLE/Classic) and Serial/USB ports.

---

## 🛠️ Development & Building

**⚠️ Important:** This repository relies on a specific folder structure. To build this project, you must clone the `extensions` repository into a folder **adjacent** to this one.

### 1. Prerequisites

* **Node.js:** v18 or v20 LTS recommended.
* **Git:** Installed and configured.

### 2. Folder Structure

You **must** organize your folders like this:

```text
/Your-Project-Root/
├── extensions/             <-- Clone https://github.com/CrispStrobe/extensions here
└── scratch-desktop/        <-- This repository

```

### 3. Setup Instructions

**Step 1: Clone the Repositories**

```bash
# 1. Create a root folder
mkdir my-lego-scratch
cd my-lego-scratch

# 2. Clone the Extensions repo (MUST be named 'extensions')
git clone https://github.com/CrispStrobe/extensions.git extensions

# 3. Clone this Desktop repo
git clone https://github.com/CrispStrobe/turbowarp-desktop.git scratch-desktop

```

**Step 2: Install & Link Dependencies**
You need to link the local extensions folder so the builder can find it.

```bash
# A. Prepare the Extensions repo
cd extensions
npm install
npm link

# B. Prepare the Desktop repo
cd ../scratch-desktop
npm install
npm link scratch-gui
# Note: We manually patched 'scripts/prepare-extensions.mjs' to look for ../extensions directly

```

### 4. Build the App

**A. Fetch Resources**
This downloads the compiler, packager, and builds your local extensions.

```bash
npm run fetch

```

**B. Compile the Code**
This builds the Electron renderer process.

```bash
npm run webpack:prod

```

*(If you are on Windows and get a `cross-env` error, run this instead:)*

```bash
npx cross-env NODE_ENV=production npx webpack

```

**C. Package the Installer (.exe/.dmg)**
This generates the final executable files in the `dist/` folder.

```bash
# Windows
npx electron-builder --win

# macOS
npx electron-builder --mac

# Linux
npx electron-builder --linux

```

---

## 🚀 Releasing Updates

Since the final binaries (~150MB) are too large for Git, we use **GitHub Releases**:

1. Push your code changes to `master`.
2. Go to the **[Releases Page](https://www.google.com/search?q=https://github.com/CrispStrobe/turbowarp-desktop/releases)**.
3. Draft a new release (e.g., `v0.1.0`).
4. Upload the `.exe` (Windows) or `.dmg` (macOS) files from your `dist/` folder.

---

## 📜 License

Based on [TurboWarp/desktop](https://github.com/TurboWarp/desktop) and [LLK/scratch-desktop](https://github.com/LLK/scratch-desktop).

Licensed under the **GPLv3.0**. See `LICENSE` for more information.