# quest-plat-converter-guide
Step-by-step guide to converting PC Beat Saber custom platforms (.plat) to Quest (.qplat) for the Void mod directly on Meta Quest using Termux and Python 3—no PC required.
# Converting Beat Saber Platforms (`.plat` → `.qplat`) Standalone on Meta Quest

Run the `voidplat` Python converter (`plat2qplat.py`) directly on your Meta Quest headset using **Termux**—no PC required.

---

## Prerequisites

* **Beat Saber** patched via [ModsBeforeFriday (MBF)](https://bsquest.xyz) with `voidplat.qmod` installed
* **Termux** (install the APK from [F-Droid](https://f-droid.org) or [Termux GitHub Releases](https://github.com))
* **Hacker's Keyboard** (install the APK from [F-Droid](https://f-droid.org) or [GitHub Releases](https://github.com)) — required for typing and pressing **Enter** inside Termux on Horizon OS
* **A Quest file manager** (e.g., [ZArchiver](https://uptodown.com))
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
