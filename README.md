
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
Here is a professional, concise, and technically accurate `README.md` for your project.

---

# Buildung

## Prerequisites

* **Node.js**: v16+ (v20+ recommended)
* **Operating System**: macOS (Required for .dmg generation)
* **Git**: Required for repository cloning

## Project Structure

Ensure your directory structure matches the following layout before proceeding:

```text
/code/turbowarp/
├── extensions/          # Fork: CrispStrobe/extensions (branch: main)
├── scratch-gui/         # Fork: CrispStrobe/scratch-gui (branch: develop)
└── turbowarp-desktop/   # Fork: CrispStrobe/turbowarp-desktop

```

## Build Instructions (macOS)

### 1. Build the GUI Library

The Desktop application requires a pre-compiled UMD library from `scratch-gui`.

```bash
cd scratch-gui

# Install dependencies
npm install

# Build library (BUILD_MODE=dist forces UMD output)
BUILD_MODE=dist npm run build

# Correct build output path
# Webpack outputs to dist/js/, but package.json expects dist/
mv dist/js/* dist/
rmdir dist/js

```

**Verification:** Ensure `dist/scratch-gui.js` exists.

### 2. Configure and Install Desktop Dependencies

Navigate to the desktop repository and perform a clean install.

```bash
cd ../turbowarp-desktop

# Clean previous artifacts
rm -rf node_modules package-lock.json dist dist-renderer-webpack

# Install dependencies ignoring scripts
# This bypasses the upstream 'prepublish' script which fails due to broken URLs
npm install --ignore-scripts

```

### 3. Replace Dependencies (Manual Linking)

Overwrite the standard installed packages with your local, custom-built versions.

```bash
# Remove standard packages
rm -rf node_modules/scratch-gui
rm -rf node_modules/@turbowarp/extensions

# Copy local builds
cp -R ../scratch-gui node_modules/
mkdir -p node_modules/@turbowarp
cp -R ../extensions node_modules/@turbowarp/

```

### 4. Resolve Dependency Conflicts (Hoisting)

Manually resolve React version conflicts and hoist critical libraries to the top-level `node_modules` to ensure Electron can locate them.

```bash
# Remove nested React/Redux to force usage of root versions (Fixes "store is null" crash)
rm -rf node_modules/scratch-gui/node_modules/react
rm -rf node_modules/scratch-gui/node_modules/react-dom
rm -rf node_modules/scratch-gui/node_modules/redux
rm -rf node_modules/scratch-gui/node_modules/react-redux

# Hoist core engines (Fixes "media glob" and extension worker errors)
mv node_modules/scratch-gui/node_modules/scratch-blocks node_modules/
mv node_modules/scratch-gui/node_modules/scratch-vm node_modules/

# Hoist TurboWarp utilities (Fixes l10n and SVG renderer errors)
cp -Rn node_modules/scratch-gui/node_modules/@turbowarp/* node_modules/@turbowarp/

# Hoist remaining loaders
cp -Rn node_modules/scratch-gui/node_modules/* node_modules/

```

### 5. Compile and Package

Run the final build steps.

```bash
# Download required assets (Skipped during npm install)
npm run fetch

# Compile application
npm run webpack:prod

# Package for macOS
npx electron-builder --mac

```

The installer will be located at `dist/TurboWarp-Setup-[version].dmg`.

## Troubleshooting

* **`ECONNRESET` during install:** Ensure you use `--ignore-scripts` in Step 2.
* **White Screen on Launch:** Ensure nested React dependencies were removed in Step 4.
* **Missing Extensions:** Ensure `scratch-vm` was correctly hoisted in Step 4.
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