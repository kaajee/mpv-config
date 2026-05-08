# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Hardware Target

i5-12450H + RTX 3050 6GB Laptop, 1440p 144Hz panel. mpv runs as a Flatpak (`io.mpv.Mpv`).

## Deploying Changes

This directory is a working copy. To apply changes to the running Flatpak install:

```bash
cp -r /home/kevin/playground/mpv-config/. /home/kevin/.var/app/io.mpv.Mpv/config/mpv/
```

To pull latest from Flatpak into this repo:

```bash
cp -r /home/kevin/.var/app/io.mpv.Mpv/config/mpv/. /home/kevin/playground/mpv-config/
```

## Architecture

### Quality Bucket System

The core feature. `mpv.conf` has `load-auto-profiles=no` — the built-in auto-profiles is disabled. Instead:

- `scripts/auto-profiles.lua` — the engine: watches properties, re-evaluates `profile-cond` expressions on idle, applies/restores profiles.
- `lua-modules/auto-profiles-functions.lua` — exports `is_high()`, `is_mid()`, `is_low()` used by profile conditions in `mpv.conf`.

**Decision matrix** (from `determine_level()` in `auto-profiles-functions.lua`):

| Power | FPS / Resolution | Bucket |
|-------|-----------------|--------|
| AC | fps ≤ 61 and width ≤ 3841 | HIGH |
| AC | fps > 61 or (UHD + fps > 61) | MID |
| Battery | fps ≤ 61 | MID |
| Battery | fps > 61 or (UHD + fps > 31) | LOW |

Battery state is read from `/sys/class/power_supply/AC*/online` (Linux) on every file load.

### Profiles in `mpv.conf`

Three quality buckets set scalers and deband:
- `[high-quality]` — ewa_lanczossharp, deband 2-pass
- `[mid-quality]` — spline36, deband 1-pass
- `[low-quality]` — bicubic, no deband

Three resolution-gated upscaler profiles (mutually exclusive, skip if LOW):
- `[upscale-sub-hd]` — width < 1280 → `FSRCNN_x2_r2_8-0-2.glsl`
- `[upscale-hd]` — 1280 ≤ width ≤ 1920 → `FSRCNNX_x2_8-0-4-1.glsl`
- `[upscale-none]` — width > 1920 or LOW → clears glsl-shaders

### Video Stack

`vo=gpu-next`, `gpu-api=vulkan`, `hwdec=nvdec`. If nvdec fails inside Flatpak, fall back to `hwdec=auto-copy-safe`.

### OSC & Scripts

- `scripts/modernz.lua` — custom OSC (replaces built-in; `osc=no` in `mpv.conf`)
- `scripts/thumbfast.lua` — seekbar thumbnail previews
- `script-opts/modernz.conf` — all ModernZ UI options (colors, button layout, behaviour)

### Shaders (`shaders/`)

- **Anime4K** — full preset library; activated via `Ctrl+1–6` in `input.conf`
- **FSRCNNX_x2_8-0-4-1.glsl** — heavy NN upscaler for 720p–1080p sources
- **FSRCNN_x2_r2_8-0-2.glsl** — lighter single-pass NN for sub-720p sources
- `shaders/auto-profiles-functions.lua` is a stale copy; the live version is `lua-modules/auto-profiles-functions.lua`

### Key Bindings (`input.conf`)

| Key | Action |
|-----|--------|
| `b` | Toggle deband |
| `a` / `A` | Cycle audio track forward/back |
| `Ctrl+1–6` | Anime4K presets (A, B, C, A+A, B+B, C+A) |
| `Ctrl+9` | Restore FSRCNNX shaders |
| `Ctrl+0` | Clear all GLSL shaders |
| `x` / `X` | Audio delay −/+ 50ms |

### Fonts (`fonts/`)

Netflix Sans (Bold/Light/Medium) — used as `sub-font` in `mpv.conf`. ModernZ icon fonts also bundled here.
