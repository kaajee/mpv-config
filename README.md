# mpv config

Personal mpv configuration for an i5-12450H + RTX 3050 6GB laptop on a 1440p 144Hz panel. Runs under the Flatpak `io.mpv.Mpv`.

## Features

- **Adaptive quality profiles** — automatically switches between High/Mid/Low rendering based on power source, video FPS, and resolution
- **Neural upscaling** — FSRCNNX for 720p–1080p sources, FSRCNN for sub-720p; skipped on Low profile
- **Anime4K presets** — six Ctrl+1–6 hotkeys for on-the-fly anime upscaling modes
- **ModernZ OSC** — custom on-screen controller with seekbar thumbnails via thumbfast
- **Vulkan + nvdec** — hardware-accelerated decode with gpu-next renderer

## Quality Profile Logic

| Power | Video | Profile |
|-------|-------|---------|
| AC | ≤ 61 fps, ≤ 4K | **High** — ewa_lanczossharp, deband 2-pass |
| AC | > 61 fps or (4K+ & > 61 fps) | **Mid** — spline36, deband 1-pass |
| Battery | ≤ 61 fps | **Mid** |
| Battery | > 61 fps or (4K+ & > 31 fps) | **Low** — bicubic, no deband, no NN shader |

Profiles are evaluated on every file load via `scripts/auto-profiles.lua`.

## Keybindings

| Key | Action |
|-----|--------|
| `b` | Toggle deband |
| `a` / `A` | Cycle audio track |
| `x` / `X` | Audio delay −/+ 50ms |
| `Ctrl+1` | Anime4K Mode A (HQ) |
| `Ctrl+2` | Anime4K Mode B (HQ) |
| `Ctrl+3` | Anime4K Mode C (HQ) |
| `Ctrl+4` | Anime4K Mode A+A (HQ) |
| `Ctrl+5` | Anime4K Mode B+B (HQ) |
| `Ctrl+6` | Anime4K Mode C+A (HQ) |
| `Ctrl+9` | FSRCNNX (restore) |
| `Ctrl+0` | Clear all GLSL shaders |

## Installation

Copy to the Flatpak config directory:

```bash
cp -r . /home/kevin/.var/app/io.mpv.Mpv/config/mpv/
```

To pull changes back from the live config:

```bash
cp -r /home/kevin/.var/app/io.mpv.Mpv/config/mpv/. .
```

## Structure

```
mpv.conf                    # Main config — video stack, subtitles, quality profiles
input.conf                  # Custom keybindings
scripts/
  auto-profiles.lua         # Profile condition engine
  modernz.lua               # Custom OSC
  thumbfast.lua             # Seekbar thumbnail previews
script-opts/
  modernz.conf              # ModernZ UI options
  thumbfast.conf            # Seekbar thumbnail options
lua-modules/
  auto-profiles-functions.lua  # is_high/is_mid/is_low helpers + battery detection
shaders/
  FSRCNNX_x2_8-0-4-1.glsl  # Heavy NN upscaler (720p–1080p)
  FSRCNN_x2_r2_8-0-2.glsl  # Light NN upscaler (sub-720p)
  Anime4K_*.glsl           # Anime upscaling/restoration shaders
fonts/
  Netflix Sans *.otf        # Subtitle font
```
