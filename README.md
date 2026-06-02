# ColorSee

Real-time color identification from your camera or uploaded images. Names what you see — Crimson, Tan, Steel Blue, Sea Green — in pill-shaped labels overlaid on the image, with each pill filled in the color it identifies so you see the match instantly.

Single HTML file. No build step. No external CDN. Drop it on GitHub Pages and it works.

## Deploy to GitHub Pages

1. Create a new public repo (e.g. `colorsee`).
2. Add `index.html` to the repo root and push.
3. In the repo, open **Settings → Pages**.
4. Under **Source**, choose `Deploy from a branch`, then pick `main` / `(root)` and **Save**.
5. After a minute, visit `https://<your-username>.github.io/<repo-name>/`.

That's it. The camera will work because GitHub Pages serves over HTTPS, which `getUserMedia` requires.

### Local testing

The camera needs HTTPS or `localhost` — opening the file directly with `file://` won't work. Quickest local server:

```bash
# Python 3
python3 -m http.server 8000
# then visit http://localhost:8000
```

Image upload works from `file://` either way.

## What changed from the previous version

Three substantive issues fixed:

**1. Labels were nearly invisible on phones.** The canvas's internal resolution was matched to the video (e.g. 1920×1080), then CSS scaled it down to phone width (~400px). A 14px font drawn at 1920-wide ends up rendered at roughly 3 CSS pixels on screen. Now the canvas is sized to `viewport × devicePixelRatio` and all drawing happens in CSS units, so labels are crisp at native pixel density regardless of camera resolution.

**2. Color matching used RGB Euclidean distance.** That treats all three channels equally, but human vision is roughly 3× more sensitive to green than blue, so RGB distance regularly picks perceptually-wrong neighbors. Switched to CIE Lab with squared-Δ distance — Lab is designed so Euclidean distance approximates perceptual difference. Palette Lab coordinates are precomputed once at startup.

**3. Single-pixel sampling caused flicker.** One noisy camera pixel would flip a cell between adjacent palette names every frame. Now each grid cell is sampled from a downsampled offscreen canvas (one pixel per cell), which gives free spatial averaging via the browser's downsampler. Camera mode adds temporal EMA smoothing on top, killing the per-frame flicker.

Other notable changes:
- Pill-shaped labels filled with the detected color, text auto-flipping black/white based on WCAG luminance, so the swatch itself is the primary signal
- Label density picker (Low / Med / High)
- `object-fit: cover` cropping is correctly mirrored in the sampling step so labels align with what you see
- Camera pauses when the tab is hidden
- Dropped the Tailwind CDN — all styling is inline CSS now, one less external dependency

## Support

If this is useful: [`$unixarcade`](https://cash.app/$unixarcade) on Cash App.
