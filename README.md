
# TurboWarp Desktop (Scratch Mod) with Lego Bricks Extension

**A specialized build of TurboWarp Desktop featuring advanced, unsandboxed extensions for LEGO® hardware.**

This version includes custom extensions:

* **LEGO® EV3 (Mindstorms):** Direct control via Bluetooth/USB and NXC transpilation.
* **LEGO® NXT:** Full support for motors, sensors, and screen drawing.
* **LEGO® Spike Prime / Robot Inventor:** Advanced control via BLE.
* **LEGO® WeDo 2.0 & Boost:** Unified support.
* **Utilities:** Gamepad support, advanced array manipulation, and CSP solvers.

**[⬇️ Download Latest Release](https://github.com/CrispStrobe/turbowarp-desktop/releases)**

---

## ⚠️ Disclaimer

**This is an unofficial, community-created modification.**

LEGO®, MINDSTORMS®, EV3, NXT, SPIKE™ Prime, and WeDo™ are trademarks of the LEGO Group. The LEGO Group does not sponsor, authorize, or endorse this software. This project is built upon [TurboWarp](https://turbowarp.org/) and [Scratch](https://scratch.mit.edu/), which are developed by their respective independent groups.

---

## 📂 Project Structure (Critical)

**⚠️ Important:** This project relies on a specific "side-by-side" folder structure. You cannot build this repo in isolation.

You **must** organize your folders exactly like this:

```text
/Your-Project-Root/
├── extensions/          # Clone: https://github.com/CrispStrobe/extensions (branch: main)
├── scratch-gui/         # Clone: https://github.com/CrispStrobe/scratch-gui (branch: develop)
└── turbowarp-desktop/   # Clone: https://github.com/CrispStrobe/turbowarp-desktop (This repo)

```

---

## 🛠️ Build Instructions: macOS

> **Note:** The macOS build process is non-standard due to React version conflicts (v16 vs v19) and upstream URL issues. You must follow the **"Brain Transplant"** method below.

### 1. Build the GUI Library

The Desktop app requires a pre-built UMD library from `scratch-gui`.

```bash
cd ../scratch-gui

# Install dependencies
npm install

# Build the library (BUILD_MODE=dist forces the correct UMD output)
BUILD_MODE=dist npm run build

# Fix Output Path Bug: Webpack outputs to 'dist/js/', but Desktop expects 'dist/'
mv dist/js/* dist/
rmdir dist/js

```

*Verify:* Ensure `dist/scratch-gui.js` exists before proceeding.

### 2. Prepare Desktop Dependencies

We perform a clean install while bypassing broken upstream scripts.

```bash
cd ../turbowarp-desktop

# Clean previous artifacts
rm -rf node_modules package-lock.json dist dist-renderer-webpack

# Install dependencies IGNORING scripts
# (This prevents an 'ECONNRESET' error from a broken upstream prepublish script)
npm install --ignore-scripts

```

### 3. The "Brain Transplant" (Manual Linking)

We must manually overwrite the standard libraries with custom-built local versions.

```bash
# Remove standard packages downloaded by npm
rm -rf node_modules/scratch-gui
rm -rf node_modules/@turbowarp/extensions

# Inject your local custom builds
cp -R ../scratch-gui node_modules/
mkdir -p node_modules/@turbowarp
cp -R ../extensions node_modules/@turbowarp/

```

### 4. Resolve Dependency Conflicts (Hoisting)

We must fix a React version conflict (v16 vs v19) and hoist critical libraries so Electron can find them.

```bash
# 1. Remove nested React/Redux from GUI to force usage of Desktop's root versions
# (This fixes the "store is null" and "render is not a function" crashes)
rm -rf node_modules/scratch-gui/node_modules/react
rm -rf node_modules/scratch-gui/node_modules/react-dom
rm -rf node_modules/scratch-gui/node_modules/redux
rm -rf node_modules/scratch-gui/node_modules/react-redux

# 2. Hoist core engines up to root (Fixes "media glob" and extension worker errors)
# We delete the root versions first to ensure a clean move
rm -rf node_modules/scratch-blocks node_modules/scratch-vm
mv node_modules/scratch-gui/node_modules/scratch-blocks node_modules/
mv node_modules/scratch-gui/node_modules/scratch-vm node_modules/

# 3. Hoist TurboWarp utilities (Fixes l10n and SVG renderer errors)
cp -Rn node_modules/scratch-gui/node_modules/@turbowarp/* node_modules/@turbowarp/

# 4. Hoist remaining loaders
cp -Rn node_modules/scratch-gui/node_modules/* node_modules/

```

### 5. Compile and Package

```bash
# Download required assets (Skipped during npm install)
npm run fetch

# Compile application
npm run webpack:prod

# Package for macOS
npx electron-builder --mac

```

The installer will be located at `dist/TurboWarp-Setup-[version].dmg`.

---

## 🛠️ Build Instructions: Windows

The Windows build process is simpler and uses standard `npm link`.

### 1. Setup Extensions

```bash
cd ../extensions
npm install
npm link

```

### 2. Setup Desktop

```bash
cd ../turbowarp-desktop
npm install

# Link the local extensions folder
npm link scratch-gui
# Note: Ensure 'scripts/prepare-extensions.mjs' is patched to look for ../extensions

```

### 3. Build

```bash
# Fetch resources
npm run fetch

# Compile Code (Use cross-env for Windows compatibility)
npx cross-env NODE_ENV=production npx webpack

# Package Installer
npx electron-builder --win

```

---

## 📜 License

Based on [TurboWarp/desktop](https://github.com/TurboWarp/desktop) and [LLK/scratch-desktop](https://github.com/LLK/scratch-desktop).

Licensed under the **GPLv3.0**. See `LICENSE` for more information.