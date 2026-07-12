<table border="0">
  <tr>
    <td>
      <!-- VERSION -->v1.00.00<br>
      <!-- DATE -->11-Jul-2026<br>
      macOS &nbsp;|&nbsp; Windows &nbsp;|&nbsp; Linux<br>
      <a href="https://landenlabs.com">Home</a>
    </td>
    <td>
      <a href="https://landenlabs.com">
        <img src="screens/landenlabs_400.webp" width="300" alt="LanDen Labs">
      </a>
    </td>
  </tr>
</table>

# Stitch Images

A command-line tool that stitches multiple images into a single composite —
scroll-screenshot / scan joins with automatic overlap removal, feature-based
panorama stitching via OpenCV, optional per-image cropping, and a two-pass
common-edge dedupe pass.

**By [LanDen Labs](https://github.com/landenlabs) (2026)**

---

## Features

- **Overlap join (default).** For `--orientation horizontal`/`vertical`, slides
  a band from the leading edge of each image across the previous one
  (`cv2.matchTemplate`) to find and remove the duplicated overlap before
  concatenating — ideal for scrolling screenshots and document scans.
- **Feature-based stitching.** `--orientation auto` (or `--method feature`)
  uses `cv2.Stitcher` in `scans` (flat, translation-only) or `panorama`
  (overlapping, rotated/perspective) mode, with automatic fallback to the
  other mode if the first fails.
- **Two-pass edge dedupe (default ON).** Detects pixel-identical top/bottom
  (vertical) or left/right (horizontal) blocks shared by every input and
  emits a single copy in the output instead of repeating them.
- **Cropping.** `--crop-height` / `--crop-width` trim each input before
  stitching, in pixels or percentages, with negative pixel values measured
  from the far edge (e.g. `10,-10`).
- **Wildcard input expansion.** `--input` accepts literal paths or shell glob
  patterns (quoted so this tool expands them, sorted alphabetically),
  repeatable and order-preserving.
- **Tunable matching.** `--match-band` / `--match-thresh` control the overlap
  detector; `--conf-thresh`, `--reg-resol`, `--features`, and
  `--no-wave-correction` tune `cv2.Stitcher` for tricky auto-stitch cases.
- **Verbose diagnostics.** `-v` reports per-input size, detected overlaps/edge
  blocks, and a naive-vs-deduped pixel savings summary.

---

## Requirements

- Python 3.9 or later
- opencv-python
- numpy

```bash
pip install -r requirements.txt
```

---

## Installation

### Run from source

```bash
git clone https://github.com/landenlabs/stitch-images.git
cd stitch-images
python stitch-images.py -i a.png -i b.png -o out.png
```

### Build a standalone binary

**macOS / Linux**

```bash
pyinstaller --onefile --name stitch-images stitch-images.py
```

**Windows**

```powershell
pyinstaller --onefile --name stitch-images stitch-images.py
```

Both commands use [PyInstaller](https://pyinstaller.org) to produce a self-contained executable.

Pushing a `v*` tag (e.g. `v1.0.0`) triggers `.github/workflows/build.yml`, which builds
macOS and Windows binaries and publishes them to a GitHub Release automatically.

---

## Usage

```
stitch-images.py --input image1.png -i image2.png -i image3.png -o final.png
```

### Orientation

- `auto` (default) — feature-based `cv2.Stitcher`; best for overlapping
  photographs needing rotation/perspective alignment.
- `horizontal` — join left-to-right; heights are scaled to match.
- `vertical` — join top-to-bottom; widths are scaled to match.

Any unique prefix is accepted (`a`, `h`, `v`, `hor`, `vert`, ...).

### Examples

```bash
# Wildcards expand to all matching files (sorted alphabetically):
stitch-images.py -i 'shots/shot*.png' -o combined.png

# Vertical join with the default overlap method (widths matched, duplicated
# scroll overlap between consecutive screenshots removed):
stitch-images.py --orientation vertical -i a.png -i b.png -i c.png -o stack.png

# Horizontal join; heights are matched automatically:
stitch-images.py --orientation horizontal -i a.png -i b.png -o row.png

# Tune overlap detection: bigger band = more distinctive; lower thresh if a
# real overlap is being missed:
stitch-images.py --ori vertical --match-band 200 --match-thresh 0.6 \
                 -v -i 'scroll*.png' -o page.png

# Feature-based join instead of overlap detection for horizontal/vertical:
stitch-images.py --orientation vertical --method feature -i 'frame*' -o m.png

# Crop each input before stitching (trim 10px borders top/bottom):
stitch-images.py --crop-height 10,-10 -i 'shot*.png' -o trimmed.png

# Crop using percentages (keep the middle 80% vertically):
stitch-images.py --crop-height 10%,90% -i a.png -i b.png -o middle.png

# Two-pass edge dedupe is on by default; disable it if unwanted:
stitch-images.py --no-dedupe-edges --orientation vertical -i 'shot*.png' -o stack.png

# Auto-stitch troubleshooting when images fail to merge despite overlap:
stitch-images.py --ori auto --conf-thresh 0.3 --reg-resol -1 --features sift \
                 --no-wave-correction -v -i 'frame*' -o m.png
```

Run `stitch-images.py --help` for the full option reference and more examples.

---

## Project structure

```
stitch-images/
├── stitch-images.py            # Main script (single-file CLI)
├── version.py                  # Version string (__version__)
├── VERSION                     # Bare X.Y.Z, mirrors version.py
├── set-version.bash            # Bump version, commit, tag, push (macOS/Linux)
├── set-version.ps1             # Bump version, commit, tag, push (Windows)
├── requirements.txt
├── README.md
├── LICENSE
├── screens/                    # Images used in this README
└── .github/workflows/build.yml # Tag-triggered build + GitHub Release
```

---

## Releasing

Versions are bumped with `set-version.bash` (or `set-version.ps1` on Windows), run from
the repo root:

```bash
./set-version.bash -version 1.0.1 -message "Fix overlap-band scoring"
```

This updates `VERSION`, `version.py` (`__version__`), and the `<!-- VERSION -->`/
`<!-- DATE -->` markers in this README, then commits, tags, and pushes. Pushing the
`vX.Y.Z` tag triggers the release build above.

---

## License

Apache 2.0 © [LanDen Labs](https://github.com/landenlabs) 2026
