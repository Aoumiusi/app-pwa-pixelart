# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## App Overview

**PIXEL FORGE** is a pixel art converter (ドット絵変換ツール) that transforms photos/illustrations into retro pixel art. All processing happens client-side using the Canvas API — no server, no build step, no dependencies.

## Development

No build process. Open `app/index.html` directly in a browser to run the app. There is no package.json, no test framework, and no linting configuration.

To test changes, open `app/index.html` in a browser. The entire app is a single file.

## Architecture

The entire application lives in `app/index.html` (~1,200 lines). Structure within the file:

- **`<style>` (lines 7–465)** — CSS with custom properties (`--bg`, `--accent`, etc.), dark theme, responsive 2-column grid layout
- **`<body>` (lines 466–613)** — HTML: header, `.drop-zone`, `.settings` aside, `.preview-area`
- **`<script>` (lines 614–1243)** — All JavaScript: state, event handlers, image processing functions

### Conversion Pipeline

The convert button runs a 5-step async pipeline:

1. **Illustrate** (optional) — full-resolution artistic processing: saturation → posterization → Sobel edge detection → edge compositing
2. **Downscale** — nearest-neighbor scaling to target size (16–256 px square)
3. **Color reduction** — preset palette (nearest-color mapping) or median cut (auto)
4. **Dithering** (optional) — Floyd-Steinberg error diffusion
5. **Display upscale** — nearest-neighbor zoom for preview only (not saved to file)

### Key Functions in `<script>`

| Function | Purpose |
|---|---|
| `PRESETS` | Palette definitions (Game Boy, Famicom, PICO-8, etc.) |
| `buildPresets()` | Builds palette preset UI with color swatches |
| `loadFile()` | Reads dropped/selected image into canvas |
| `btnConvert` handler | Orchestrates the 5-step pipeline (async) |
| `illustrateImage()` | Anime/flat/sketch style processing |
| `gaussianBlurData()` | 3×3 box blur approximation |
| `sobelEdge()` | Gradient-based edge detection |
| `reduceColors()` / `medianCut()` | Automatic palette generation |
| `nearestColor()` | Maps pixels to nearest palette entry (Euclidean RGB distance) |
| `floydSteinberg()` | Error diffusion dithering |

### State

Global variables hold app state: `originalImage`, `outputCanvas`, `selectedSize` (default 32), `activePalette` (default `null`), `activeIllustrateStyle`.

### Output

Downloads as `pixelart_{size}x{size}.png` at the actual pixel size (zoom not applied to download).

## Key Constraints

- Output is always square — non-square inputs are force-resized, changing aspect ratio
- Illustration processing on images ≥4000 px can take several seconds
- The 3×3 box blur used in `gaussianBlurData()` is an approximation of Gaussian blur
- `PRESETS.famicom` has 54 colors (full PPU table) but real hardware only allows 25 simultaneously
