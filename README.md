# turbowarp-desktop (LEGO fork)

An Electron build of [TurboWarp Desktop](https://github.com/TurboWarp/desktop)
that bundles custom **unsandboxed** LEGO hardware extensions:

- **EV3 (Mindstorms):** direct control over Bluetooth/USB; transpile to NXC or
  lmsasm and compile via [`legacy-lego-compiler`](https://github.com/CrispStrobe/legacy-lego-compiler).
- **EV3 (ev3dev firmware):** stream commands or transpile to Python via the
  on-device bridge (`ev3dev_ondevice.py`).
- **NXT:** motors, sensors, screen drawing, NXC transpile.
- **Spike Prime / Robot Inventor:** BLE (FW 3.x) and BTC (FW 2.x).
- **WeDo 2.0, Boost, Powered UP, Technic Hub:** unified BLE.
- **Utilities:** gamepad input, array/tensor blocks, CSP solver.

**[Download latest release](https://github.com/CrispStrobe/turbowarp-desktop/releases)**

## What's different from upstream TurboWarp Desktop

This fork swaps the upstream extension gallery and the bundled `scratch-gui`
build for local custom builds, then overrides Electron's security policy to
allow `navigator.bluetooth` / `navigator.serial` from inside the LEGO
extensions. Concretely:

- `node_modules/scratch-gui` is replaced by [`CrispStrobe/brickwright`](https://github.com/CrispStrobe/brickwright) (`develop` branch, library mode).
- `node_modules/@turbowarp/extensions` is replaced by [`CrispStrobe/extensions`](https://github.com/CrispStrobe/extensions) (`main`).
- React/Redux are hoisted up to the Electron root to resolve a v16/v19 version conflict that breaks the renderer.

The build steps below codify the "brain transplant" needed to make those
swaps work.

## Disclaimer

Unofficial, community-created modification. LEGO®, MINDSTORMS®, EV3, NXT,
SPIKE™ Prime, and WeDo™ are trademarks of the LEGO Group, which does not
sponsor, authorize, or endorse this software. Built on
[TurboWarp](https://turbowarp.org/) and [Scratch](https://scratch.mit.edu/).

## Related repos

| Repo | Role |
|------|------|
| **`turbowarp-desktop` (this)** | Electron desktop build. |
| [`CrispStrobe/brickwright`](https://github.com/CrispStrobe/brickwright) | The editor UI (built in library mode for desktop). |
| [`CrispStrobe/extensions`](https://github.com/CrispStrobe/extensions) | Extension gallery consumed by the editor. |
| [`CrispStrobe/turbowarp-lego`](https://github.com/CrispStrobe/turbowarp-lego) | Working sandbox + Python bridges. |
| [`CrispStrobe/legacy-lego-compiler`](https://github.com/CrispStrobe/legacy-lego-compiler) | NXC / lmsasm REST API. |
| [`CrispStrobe/turbowarp-android`](https://github.com/CrispStrobe/turbowarp-android) | Android wrapper variant. |
| [`CrispStrobe/turbowarp-ios`](https://github.com/CrispStrobe/turbowarp-ios) | iOS wrapper variant. |

## Project structure (required)

This repo cannot be built in isolation — the build copies sources from sibling
clones. Lay them out exactly like this:

```text
/Your-Project-Root/
├── extensions/          # clone: https://github.com/CrispStrobe/extensions  (branch: main)
├── scratch-gui/         # clone: https://github.com/CrispStrobe/brickwright (branch: develop)
└── turbowarp-desktop/   # clone: https://github.com/CrispStrobe/turbowarp-desktop  (this repo)
```

---

## Build: macOS

The macOS build is non-standard because of the v16/v19 React conflict and a
broken upstream prepublish URL. Follow the steps below in order.

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

## Build: Windows

The Windows build is simpler and uses standard `npm link`.

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

## Troubleshooting

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `ECONNRESET` during `npm install` | Upstream prepublish hits a flaky URL | use `npm install --ignore-scripts` (the brain-transplant steps above already do this) |
| `store is null` / `render is not a function` at startup | Duplicate React (v16 inside `scratch-gui`, v19 at root) | re-run step 4 — remove `node_modules/scratch-gui/node_modules/{react,react-dom,redux,react-redux}` |
| `Cannot find module 'scratch-blocks'` | scratch-blocks not hoisted | re-run step 4 — `mv node_modules/scratch-gui/node_modules/scratch-blocks node_modules/` |
| `media glob failed` / extension worker crashes | scratch-vm not hoisted | same as above for `scratch-vm` |
| Bluetooth device picker doesn't appear | Permission policy not loosened in Electron | check `src/main.js` (web preferences) and `tw-security-manager.jsx` allowlist in `scratch-gui` |

## License

Based on [TurboWarp/desktop](https://github.com/TurboWarp/desktop) and
[LLK/scratch-desktop](https://github.com/LLK/scratch-desktop). Licensed under
**GPL-3.0**. See [`LICENSE`](LICENSE).