# SDK Patches

This directory documents the changes made to `thirdparty/rexglue-sdk` (which
is git-ignored and fetched separately) to produce the AppImage releases.

## Patches

### fsr-linux-headers-only.patch
Makes the FidelityFX SDK integration Linux-friendly: on Linux, only the
FidelityFX headers are used (enabling the precompiled FSR 1.0 EASU/RCAS
shaders). The full FSR2/FSR3 runtime is linked only on Windows because it
requires `FidelityFX_SC.exe` (a Windows-only shader compiler) to build.

Apply with: `git apply fsr-linux-headers-only.patch` from the SDK root.

### frame-limiter.md
Documents the `frame_limit` cvar and frame-pacing logic added to
`src/graphics/command_processor.cpp`. Kept as documentation (not a git patch)
because the change is interleaved with other local SDK modifications.

## Building with these patches

1. Clone the ReXGlue SDK to `thirdparty/rexglue-sdk`
2. Apply `fsr-linux-headers-only.patch`
3. Manually apply the frame limiter changes per `frame-limiter.md`
4. Configure with `-DREXGLUE_ENABLE_FIDELITYFX=ON`
5. Build the main project per the top-level README
