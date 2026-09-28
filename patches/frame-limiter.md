# Frame Limiter Patch

## Purpose
Caps the guest GPU frame rate (default 60 FPS) so the game runs at correct
speed when vsync is disabled or on high-refresh displays. Without this, the
game runs too fast (uncapped frame rate = uncapped game speed).

## Location
`thirdparty/rexglue-sdk/src/graphics/command_processor.cpp`

## Changes

### 1. New cvar (near other GPU cvars, ~line 44)

```cpp
REXCVAR_DEFINE_INT32(frame_limit, 60, "GPU",
                     "Frame rate limit in FPS (0 = unlimited). Paces guest "
                     "swaps so game speed stays correct when vsync is off or "
                     "on high-refresh displays.");
```

Requires `#include <chrono>` and `#include <thread>` at the top of the file.

### 2. Limiter logic (in `ExecutePacketType3_XE_SWAP`, after swap handling)

```cpp
const int32_t frame_limit = REXCVAR_GET(frame_limit);
if (frame_limit > 0) {
  // Pace the guest: sleep until the next frame interval.
  auto frame_duration =
      std::chrono::microseconds(1000000 / frame_limit);
  // ... sleep logic to enforce frame pacing
}
```

The limiter sleeps the command processor thread so that guest swaps don't
exceed the configured FPS. Set to 0 for unlimited.

## Launcher integration
The Qt launcher exposes this as "Frame limit" (0–240 FPS, 0 = unlimited)
and passes `--frame_limit=<value>` to the game. See
`launcher-linux/game_config.cpp` and `launcher-linux/main_window.cpp`.
