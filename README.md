# mpv config

Personal mpv configuration for an i5-12450H + RTX 3050 6GB laptop on a 1440p 144Hz panel. Runs under the Flatpak `io.mpv.Mpv`.

## Features

- **NVIDIA shaders** — NVSharpen and NVScaler are selected by default
- **Conditional profiles** — mpv adjusts rendering for source resolution, HDR, frame rate, and display refresh rate
- **ModernZ OSC** — custom on-screen controller with seekbar thumbnails via thumbfast
- **Vulkan + nvdec** — hardware-accelerated decode with gpu-next renderer

## Profiles

`mpv.conf` includes `profiles.conf` and enables the `NVIDIA` profile. The automatic `SD` profile also applies `NVIDIA` when the source is at most 1280×720. The `8k-fix` profile clears the shader list for sources wider than 7000 pixels or taller than 3000 pixels.

## Keybindings

| Key | Action |
|-----|--------|
| `d` | Toggle deband |
| `a` / `A` | Cycle audio track |
| `x` / `X` | Audio delay −/+ 50ms |
| `Ctrl+0` | Clear all GLSL shaders |
| `Ctrl+1` | Toggle KrigBilateral |
| `Ctrl+2` / `Ctrl+3` | Toggle SSimSuperRes / SSimDownscaler |
| `Ctrl+4` | Toggle Adaptive Sharpen |
| `Ctrl+5` / `Ctrl+6` | Toggle FSRCNNX / FSRCNNX-distort |
| `Ctrl+7` | Toggle NNEDI3 nns32-4 |

## Installation

Run from the repository root to copy the config and its assets into Flatpak:

```bash
config_dir="$HOME/.var/app/io.mpv.Mpv/config/mpv"
mkdir -p "$config_dir"
cp -r mpv.conf profiles.conf input.conf scripts script-opts shaders fonts icc "$config_dir/"
```

To pull changes back from the live config:

```bash
config_dir="$HOME/.var/app/io.mpv.Mpv/config/mpv"
cp -r "$config_dir"/{mpv.conf,profiles.conf,input.conf,scripts,script-opts,shaders,fonts,icc} .
```

## Structure

```
mpv.conf                    # Main config and default profile selection
profiles.conf               # Named and automatic mpv profiles
input.conf                  # Custom keybindings
scripts/
  modernz.lua               # Custom OSC
  thumbfast.lua             # Seekbar thumbnail previews
script-opts/
  modernz.conf              # ModernZ UI options
  thumbfast.conf            # Seekbar thumbnail options
shaders/                    # NVIDIA, FSRCNNX, Anime4K, and other shaders
icc/                        # Display color profiles
fonts/
  Netflix Sans *.otf        # Subtitle font
```
