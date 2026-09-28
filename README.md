# Dante's Inferno - Xbox 360 to PC Port (ReXGlue)

<p align="center">
  <img src="assets/fan_artwork.png" alt="Dante's Inferno - Fan Artwork" width="256" />
</p>

<p align="center">
  <em>Fan artwork by <a href="https://www.deviantart.com/pooterman">POOTERMAN</a> (<a href="https://github.com/florinp93/hells-gate-recomp/issues/12">#12</a>)</em>
</p>

A static recompilation port of **Dante's Inferno** (Xbox 360) to native PC,
built with the [ReXGlue SDK](https://github.com/rexglue/rexglue-sdk).

ReXGlue converts Xbox 360 PowerPC XEX executables into portable C++ that runs
natively on Windows (D3D12) and Linux (Vulkan) - no emulation, no JIT at
runtime.

<p align="center">
  <a href="https://ko-fi.com/zerkiller">
    <img src="https://img.shields.io/badge/Ko--Fi-Buy%20me%20a%20coffee-FF5E5B?style=for-the-badge&logo=ko-fi&logoColor=white" alt="Ko-fi" />
  </a>
  <a href="https://discord.gg/mjGfv7ysG8">
    <img src="https://img.shields.io/badge/Discord-Join%20the%20server-5865F2?style=for-the-badge&logo=discord&logoColor=white" alt="Discord" />
  </a>
  <a href="https://github.com/florinp93">
    <img src="https://img.shields.io/badge/Other-Projects-0AB4F5?style=for-the-badge&logo=github&logoColor=white" alt="Other Projects" />
  </a>
</p>

## 🤖 AI Usage Disclosure

Transparency matters here, so let's be upfront: this fork's Linux AppImage
work (launcher improvements, bug fixes, packaging) was developed
collaboratively with an AI assistant (Muse, by Meta).

### What the AI did:
* **Launcher features:** Designed and implemented the FSR upscaling selector,
  frame limiter UI, and Vulkan argument handling in the Qt launcher
* **Bug diagnosis:** Tracked down the black-screen-at-launch (Vulkan sparse
  memory on AMD), the dead-mouse-input issue (dangling symlinks breaking the
  Qt Wayland plugin), and the build environment failures
* **Packaging:** Fixed the AppImage build script (symlink dereferencing,
  Wayland plugin bundling)
* **Documentation:** Wrote the patch docs and build notes in `patches/`

### What the human did:
* **Direction:** Every feature and fix was requested, prioritized, and
  approved by the maintainer
* **Testing:** All validation happened on real hardware (Kubuntu/Plasma
  Wayland, AMD RX 6600) — the AI can't test what it can't run
* **Decisions:** What to build, what to publish, and how to present it were
  all human calls

The upstream ReXGlue SDK and the original port are human-written. The AI
contributions here are real code, not just docs — and we're not going to
pretend otherwise.

## Linux build (Ubuntu)

The project can also be built natively on Linux using Vulkan.

The commands below use **Clang 22** and the same ReXGlue workflow used by the
Windows build, with the additional Linux GPU plugin and generated-code patch
step.

### 1. Install the required Linux dependencies

Make manifest backup and make the provided dependency installer executable and run it:

```bash
cp dantes_inferno_manifest.toml dantes_inferno_manifest.toml.back
chmod +x ./scripts/install_dantes_min.sh
./scripts/install_dantes_min.sh
```

This installs the compiler/toolchain and the development libraries needed by
ReXGlue on Ubuntu.

### 2. Set up the SDK

Run the existing project setup script with PowerShell:

```bash
pwsh ./setup.ps1
```

### 3. Configure Linux and build the ReXGlue CLI

```bash
cmake -B out/build/linux-release \
  -G Ninja \
  -DCMAKE_C_COMPILER=/usr/bin/clang-22 \
  -DCMAKE_CXX_COMPILER=/usr/bin/clang++-22 \
  -DCMAKE_CXX_FLAGS="-stdlib=libstdc++ -I$(pwd)/thirdparty/rexglue-sdk/thirdparty/imgui -mssse3 -mavx2" \
  -DREXSDK_DIR=thirdparty/rexglue-sdk

cmake --build out/build/linux-release --target rexglue
```

### 4. Build the Xenos GPU plugin

```bash
cmake --build out/build/linux-release --target rexgpu-xenos

cp thirdparty/rexglue-sdk/out/linux-amd64/lib*.so \
  ./out/build/linux-release/
```

### 5. Regenerate the SDK-managed project files

Make sure `game/default.xex` is present before running this command:

```bash
thirdparty/rexglue-sdk/out/linux-amd64/rexglue init \
  --force \
  --project-name dantes_inferno \
  --project-root . \
  --xex-path game/default.xex \
  --game-root game
```

### 6. Generate the recompiled C++ sources

Reconfigure the project:

```bash
cmake -B out/build/linux-release \
  -G Ninja \
  -DCMAKE_C_COMPILER=/usr/bin/clang-22 \
  -DCMAKE_CXX_COMPILER=/usr/bin/clang++-22 \
  -DCMAKE_CXX_FLAGS="-stdlib=libstdc++ -I$(pwd)/thirdparty/rexglue-sdk/thirdparty/imgui -mssse3 -mavx2" \
  -DREXSDK_DIR=thirdparty/rexglue-sdk
```

Restore the project manifest backup, then run codegen:

```bash
cp dantes_inferno_manifest.toml.back dantes_inferno_manifest.toml

cmake --build out/build/linux-release \
  --target dantes_inferno_codegen
```

### 7. Apply the generated-code patches

The generated sources need the project-specific patches before the final game
build:

```bash
python3 patches/generated/apply_generated_patches.py
```

### 8. Build the Linux executable

Reconfigure once more:

```bash
cmake -B out/build/linux-release \
  -G Ninja \
  -DCMAKE_C_COMPILER=/usr/bin/clang-22 \
  -DCMAKE_CXX_COMPILER=/usr/bin/clang++-22 \
  -DCMAKE_CXX_FLAGS="-stdlib=libstdc++ -I$(pwd)/thirdparty/rexglue-sdk/thirdparty/imgui -mssse3 -mavx2" \
  -DREXSDK_DIR=thirdparty/rexglue-sdk
```

Restore the manifest and build the port:

```bash
cp dantes_inferno_manifest.toml.back dantes_inferno_manifest.toml

cmake --build out/build/linux-release --target dantes_inferno
```

The resulting Linux executable is:

```text
out/build/linux-release/dantes_inferno
```

Remove manifest backup:

```bash
cp dantes_inferno_manifest.toml.back dantes_inferno_manifest.toml
rm -rf dantes_inferno_manifest.toml.back
```

### 9. Run on Linux

From the repository root:

```bash
./scripts/dantes_inferno_exe.sh
```

Code `dantes_inferno_exe.sh` is:

```bash
 LD_LIBRARY_PATH="$PWD/thirdparty/rexglue-sdk/out/linux-amd64:$LD_LIBRARY_PATH" 
 ./out/build/linux-release/dantes_inferno
```

## Progress Tracker

- [x] Game boots, runs, and is fully playable
  - ReXGlue SDK v0.10.0 codegen + native build
  - VMX/AltiVec PowerPC instructions supported
  - VP6/Bink FMV corruption fixed (upstream PR to be made)
  - Save system fixed (fiber/setjmp/longjmp + `XUserFindUsers` handler)

- [x] Graphics & input configured
  - Resolution scaling, anisotropic override, post-effect cvars
  - Aspect ratio control: 4:3 / 16:9 / 16:10 / 21:9 / 32:9
  - SDL input backend set as default
  - Mouse & keyboard keybind defaults configured

### In progress
- [x] DLC auto-install hook (`OnPostSetup` STFS package scan)
- [ ] 120 Hz / high-refresh timing polish (gameplay OK; menu/minigame timing under reverse engineering)
- [ ] Native DiligentCore/Vulkan renderer migration (working, not fully implemented)

## Roadmap

| Phase | Feature | Status |
|-------|---------|--------|
| 1 | Graphics quality cvars (resolution scale, AA, filtering) | Done |
| 2 | Input defaults + SDL backend | Done |
| 3 | DLC auto-install | In progress |
| 4 | Ultrawide / aspect-ratio support | Done |
| 5 | Button glyph replacement (input-device RE) | Planned |
| 6 | Native DiligentCore/Vulkan renderer migration | In progress (working, not final) |
