
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

**⚠️ Important:** This repository relies on a strict folder structure to function. You cannot build `turbowarp-desktop` in isolation; it requires the `extensions` and `scratch-gui` repositories to be present side-by-side.

### 1. Prerequisites

* **Node.js:** v18 or v20 LTS recommended.
* **Git:** Installed and configured.
* **Clean Environment:** Ensure you do not have a rogue `node_modules` folder in your root code directory (e.g. `~/code/node_modules`).

### 2. Folder Structure

You **must** organize your folders like this:

```text
/Your-Project-Root/
├── extensions/             <-- Clone https://github.com/CrispStrobe/extensions here
├── scratch-gui/            <-- Clone https://github.com/CrispStrobe/scratch-gui here
└── turbowarp-desktop/      <-- This repository

```

### 3. Setup Instructions

#### Step 1: Clone All Repositories

```bash
# 1. Create a root folder
mkdir my-lego-scratch
cd my-lego-scratch

# 2. Clone Extensions (Folder MUST be named 'extensions')
git clone https://github.com/CrispStrobe/extensions.git extensions

# 3. Clone GUI (Folder MUST be named 'scratch-gui')
git clone https://github.com/CrispStrobe/scratch-gui.git scratch-gui

# 4. Clone Desktop (Folder MUST be named 'turbowarp-desktop')
git clone https://github.com/CrispStrobe/turbowarp-desktop.git turbowarp-desktop

```

#### Step 2: Install & Build Dependencies

You must set up the repositories in this specific order.

**A. Setup Extensions**

```bash
cd extensions
npm install
npm link
cd ..

```

**B. Setup GUI**
This step builds the interface dist files required by the desktop app.

```bash
cd scratch-gui
# Ensure build script uses valid URL (if needed)
npm install
npm run build
npm link
cd ..

```

**C. Setup Desktop**
This connects everything. We use `file:` paths for stability.

```bash
cd turbowarp-desktop

# 1. Clean any old installs
rm -rf node_modules package-lock.json

# 2. Configure package.json to use local folders
# Ensure "scratch-gui" and "@turbowarp/extensions" point to "file:../[folder]"

# 3. Install dependencies
npm install

# 4. Force install React to prevent webpack conflicts
npm install react react-dom react-redux redux --save-dev

```

### 4. Build the App

**A. Fetch Resources**
Downloads the compiler, packager, and builds your local extensions.

```bash
npm run fetch

```

**B. Compile the Code**
This builds the Electron renderer process.

```bash
npm run webpack:prod

```

*(Windows Users: If you get a `cross-env` error, run `npx cross-env NODE_ENV=production npx webpack` instead)*

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

Since the final binaries (~150MB) are large, we use **GitHub Releases**:

1. Push your code changes to `master`.
2. Go to the **[Releases Page](https://www.google.com/search?q=https://github.com/CrispStrobe/turbowarp-desktop/releases)**.
3. Draft a new release (e.g., `v0.1.0`).
4. Upload the `.exe` (Windows) or `.dmg` (macOS) files from your `dist/` folder.

---

## 📜 License

Based on [TurboWarp/desktop](https://github.com/TurboWarp/desktop) and [LLK/scratch-desktop](https://github.com/LLK/scratch-desktop).

Licensed under the **GPLv3.0**. See `LICENSE` for more information.