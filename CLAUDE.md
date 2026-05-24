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

The config is split into two files, loaded via `include="~~/profiles.conf"` from `mpv.conf`:

- **`mpv.conf`** — global options (video output, audio, subs, OSD) and the list of unconditional `profile=<name>` activations.
- **`profiles.conf`** — all profile definitions, both *general* (activated explicitly via `profile=` in `mpv.conf`) and *conditional* (auto-triggered by `profile-cond` when `load-auto-profiles=yes`).

There is no Lua-driven quality bucket engine. Profile selection is entirely declarative: mpv's built-in `load-auto-profiles` re-evaluates `profile-cond` expressions on property changes and applies / restores via `profile-restore`.

### General profiles (activated in `mpv.conf`)

Listed at the top of `mpv.conf` under "Profiles enable parameters". Currently active by default: `Target` (target-prim colour management), `Dither` (fruit dither), `Tscale` (display-resample + oversample interpolation), `HQ` (ewa_lanczossharp + bilinear cscale + lanczos dscale), `DeBand-low` (deband params; `deband` itself stays off until `[Deband]` fires).

Commented-out alternatives: `ICC`, `ICC+`, `Dither+`, `Tscale+`, `DeBand-medium/high`, `SDR2HDR`, `HDR-PASS`, `SWscaler`, plus a row of shader profiles (`NNEDI3`, `NNEDI3+`, `ravu-zoom`, `FSRCNNX`, `FSRCNNX+`, `Ani4K`, `AniSD`, `Anime4K`, `SSIM`). Uncomment one to set `glsl-shaders=` permanently.

### Conditional profiles (`profiles.conf` bottom half)

| Profile | Trigger | Effect |
|---------|---------|--------|
| `[SD]` | `height ≤ 720` | apply `FSRCNNX+` (FSRCNNX_x1_distort + SSimDownscaler) |
| `[Deband]` | source bpp < 24, not 8K | `deband=yes` |
| `[HDR2SDR]` | HDR source on SDR display | tone-mapping params (peak detect, contrast recovery) |
| `[peak-percentile]` | low avg PQ luma, gpu-next | tighter HDR peak percentile |
| `[SDR-gamut]` | non-BT.709 SDR | `gamut-mapping-mode=clip` |
| `[SDR-target]` | BT.1886 + no inverse tone map | drop colorspace hint |
| `[HDR]` | `max-luma > 203` | HDR display targeting, deband off, clear vf |
| `[HDR-PASS]` | HDR src + HDR display | full passthrough |
| `[video-sync]` / `[fps-fix]` | speed-corrected audio / >47 fps src or >120Hz display | switch video-sync mode |
| `[8k-fix]` | width > 7000 or height > 3000 | drop shaders/deband for 8K |
| `[pgs-fix]` | aspect > 16:9 | stretch image subs |
| `[pause]` / `[maximized]` / `[minimized]` / `[end]` / `[media-title]` | window/playback state | UX tweaks |

### Video Stack

`vo=gpu-next`, `gpu-api=vulkan`, `hwdec=nvdec`. If nvdec fails inside Flatpak, fall back to `hwdec=auto-copy-safe`.

### OSC & Scripts

- `scripts/modernz.lua` — custom OSC (replaces built-in; `osc=no` in `mpv.conf`)
- `scripts/thumbfast.lua` — seekbar thumbnail previews
- `script-opts/modernz.conf` — all ModernZ UI options (colors, button layout, behaviour)
### Shaders (`shaders/`)

Bundles ACNet, AMD/Nvidia, Ani4Kv2 + AniSD, Anime4K, AnimeJaNai, ArtCNN, Chroma, CuNNy, FSRCNNX, NNEDI3, RAVU, SSim, plus standalone `adaptive-sharpen.glsl`, `hdeband.glsl`, `nlmeans.glsl`. Profile activation paths live in `profiles.conf`.

### Key Bindings (`input.conf`)

| Key | Action |
|-----|--------|
| `b` | Toggle deband |
| `a` / `A` | Cycle audio track forward/back |
| `Ctrl+1–6` | Anime4K presets (A, B, C, A+A, B+B, C+A) |
| `Ctrl+9` | Restore default shaders |
| `Ctrl+0` | Clear all GLSL shaders |
| `x` / `X` | Audio delay −/+ 50ms |

### Fonts (`fonts/`)

Netflix Sans (Bold/Light/Medium) — used as `sub-font` in `mpv.conf`. ModernZ icon fonts also bundled here.
