# quest-plat-converter-guide
Step-by-step guide to converting PC Beat Saber custom platforms (.plat) to Quest (.qplat) for the Void mod directly on Meta Quest using Termux and Python 3—no PC required.
# Converting Beat Saber Platforms (`.plat` → `.qplat`) Standalone on Meta Quest

How to run the `voidplat` Python converter (`plat2qplat.py`) directly on your Meta Quest headset using **Termux**—no PC required.

---

## Prerequisites

* **Beat Saber** patched via [ModsBeforeFriday (MBF)](https://bsquest.xyz) with `voidplat.qmod` installed
* **Termux** (install the APK from [F-Droid](https://f-droid.org/en/packages/com.termux/) or [Termux GitHub Releases](https://github.com/termux/termux-app ))
* **Hacker's Keyboard** (install the APK from [F-Droid](https://f-droid.org/en/packages/org.pocketworkstation.pckeyboard/) or [GitHub Releases](https://github.com/klausw/hackerskeyboard )) — required for typing and pressing **Enter** inside Termux on Horizon OS
* **A Quest file manager** (e.g., [ZArchiver](https://www.apkmirror.com/apk/zdevs/zarchiver/))
* **`void-converter-linux.zip`** from the `voidplat` GitHub Releases page

---

## 1. Enable Hacker's Keyboard on Quest

Horizon OS's stock virtual keyboard does not register keystrokes inside Termux's terminal view. Before opening Termux:

1. Sideload and open **Hacker's Keyboard**.
2. Tap **Enable Keyboard** and toggle **Hacker's Keyboard** ON in Quest's Android Language & Input settings.
3. Tap **Set Input Method** and switch your active keyboard to **Hacker's Keyboard**.
4. Inside Termux, if the keyboard doesn't pop up automatically, place your pointer on the **far-left edge** of the Termux window, hold the trigger, and **swipe right** to open the session drawer, then tap **KEYBOARD**.

---

## 2. Set Up Python & Dependencies in Termux

Because PyPI does not distribute pre-built ARM64 Android wheels for `numpy`, `scipy`, `Pillow`, or `pyfqmr`, install the heavy C/C++ libraries via Termux's `pkg` manager first and compile the remaining packages locally.

### 2.1 Grant Storage Access & Install Base Packages
Paste this block into Termux and press **Enter** (tap **Allow** when Horizon OS prompts for storage permissions):

```bash
termux-setup-storage
pkg update -y && pkg upgrade -y
pkg install -y python python-numpy python-scipy python-pillow git build-essential tur-repo
pkg install -y python-scipy
```

### 2.2 Install `UnityPy` & Build Tools
```bash
pip install UnityPy cython setuptools wheel
```
*(Compiling `UnityPy`'s native C++ sub-packages such as `astc-encoder-py` and `etcpak` will take 1–2 minutes.)*

### 2.3 Install `pyfqmr` (Fast Quadric Mesh Reduction)
1. In your Quest browser, open the [`Kramer84/pyfqmr-Fast-Quadric-Mesh-Reduction`](https://github.com) repository, select **Code → Download ZIP**, and wait for it to finish downloading to `/sdcard/Download`.
2. Build and install it from your Downloads folder without build isolation:
   ```bash
   pip install --no-build-isolation /sdcard/Download/*pyfqmr*.zip
   ```

---

## 3. Extract the Converter Script

1. Download **`void-converter-linux.zip`** from the `voidplat` GitHub Releases page.
2. Extract it using your file manager so `plat2qplat.py` is located at:
   ```text
   /sdcard/Download/void-converter/plat2qplat.py
   ```

---

## 4. Convert `.plat` Files to `.qplat`

1. Download any PC `.plat` file from [BeatMods](https://beatmods.com) or [ModelSaber](https://modelsaber.com) into `/sdcard/Download`.
2. Run the batch conversion command in Termux:
   ```bash
   cd /sdcard/Download && for f in *.plat; do python void-converter/plat2qplat.py "$f"; done
   ```

> [!NOTE]
> **Fixing `numpy.linalg.LinAlgError: Singular matrix`:**
> If a platform contains a zero-scaled mesh transform and crashes during static mesh merging, re-run the converter with `--no-merge`:
> ```bash
> python void-converter/plat2qplat.py "PlatformName.plat" --no-merge
> ```

---

## 5. Verify Output & Load Into Beat Saber

### 5.1 Check the Converter Summary
Before copying a `.qplat` into Beat Saber, inspect the summary printed in Termux:

| Indicator | Works Well on Quest | Likely Broken / Dark on Quest |
| :--- | :--- | :--- |
| **Output `.qplat` size** | `100 KB+` (up to several MB) | `10–15 KB` (geometry/shaders stripped) |
| **Lights & Rings** | `tube lights > 32` or `ring sets > 16` | `0 tube lights, 0 ring sets` |
| **Shaders** | `Unlit/Color`, `BeatSaber/Unlit Glow`, `BeatSaber/Lit Glow` | `Poiyomi Toon`, `CellShading`, `GlassRefraction`, `ProBuilder` |

*(Platforms built around Beat Saber tube lights and ring sets—such as **HiWave** or **SynthWave**—adapt dynamically to song lighting without relying on PC-only DirectX shaders.)*

### 5.2 Install the `.qplat` File
1. Copy the generated `.qplat` file into:
   ```text
   /sdcard/ModData/com.beatgames.beatsaber/Mods/voidplat/
   ```
   *(Alternatively, upload the `.qplat` file directly through **ModsBeforeFriday** in your Quest browser.)*
2. Launch Beat Saber, navigate to **Mods → Void** on the left panel, and select your converted platform.

## Troubleshooting

### 1. `pyfqmr` Fails to Install (`No matching distribution found` or `No module named 'setuptools'`)
PyPI does not have pre-built Android (`aarch64`) wheels or source tarballs for `pyfqmr`, so `pip install pyfqmr` will fail.
* **Fix:**
  1. Install `cython`, `setuptools`, and `wheel` first:
     ```bash
     pip install cython setuptools wheel
     ```
  2. Download the `pyfqmr-Fast-Quadric-Mesh-Reduction` source ZIP directly from GitHub in your browser (**Code → Download ZIP**) into `/sdcard/Download`.
  3. Install it locally without build isolation:
     ```bash
     pip install --no-build-isolation /sdcard/Download/*pyfqmr*.zip
     ```

### 2. `etcpak` (or `astc-encoder-py` / `texture2ddecoder`) Fails to Build During `pip install UnityPy`
`UnityPy` compiles several C/C++ texture decoders (`etcpak`, `astc-encoder-py`, `texture2ddecoder`, `brotli`, `fmod_toolkit`) from source on Android.
* **Fix:** Make sure Termux's C++ compiler toolchain is installed and force `clang` if needed:
  ```bash
  pkg install -y clang cmake ninja build-essential python
  CC=clang CXX=clang++ pip install UnityPy
  ```

### 3. `UnityPy` Fails on `Pillow` (`Failed building wheel for Pillow`)
Compiling `Pillow` from source via `pip` fails if JPEG/zlib headers are missing.
* **Fix:** Install Termux's pre-built `python-pillow` package instead, then re-run `pip install UnityPy`:
  ```bash
  pkg install -y python-pillow
  pip install UnityPy
  ```

### 4. `E: Unable to locate package cython` or `Cannot find command 'git'`
`cython` is a Python package (installed via `pip`), not a Termux `pkg` package—and if `pkg install` hits a missing package, it stops before installing the rest of the line.
* **Fix:** Install `git` with `pkg` and `cython` with `pip` separately:
  ```bash
  pkg install -y git
  pip install cython
  ```

### 5. `numpy.linalg.LinAlgError: Singular matrix` When Converting
Some `.plat` files contain a mesh object scaled to `0` on one axis, which crashes `merge_static` during conversion.
* **Fix:** Add `--no-merge` to the end of the conversion command:
  ```bash
  python void-converter/plat2qplat.py "PlatformName.plat" --no-merge
  ```

### 6. `no CustomPlatform component found: is this a platform?`
* **Cause A:** There are no `.plat` files in the folder you ran the loop on, so bash passed the literal text `*.plat` to Python. Make sure your `.plat` file is inside `/sdcard/Download` (or whichever folder you `cd` into).
* **Cause B:** The file is a `.bloq` saber, `.avatar`, or a platform built for an incompatible Unity version.

### 7. Platform Loads Pitch Black, Grayscale, or Has a Floating Shadow
* **Pitch black / grayscale:** The PC platform uses DirectX-only shaders (`Poiyomi Toon`, `CellShading`, `GlassRefraction`, `ProBuilder`) that Quest cannot render. Look for platforms using `Unlit/Color`, `BeatSaber/Unlit Glow`, `BeatSaber/Lit Glow`, and native `tube lights` or `ring sets` instead.
* **Floating dark shadow / shape:** On PC, some platforms include semi-transparent fog planes or camera-follow meshes that `plat2qplat.py` converts into solid dark unlit meshes.
* **Environment override:** Make sure **Override Default Environments** is turned **OFF** in Beat Saber's left-hand **Environments** menu.
