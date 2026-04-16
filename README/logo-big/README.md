# logo-big.svg — Generation Process

`logo-big.svg` is a clean vector version of `logo-big.png`, generated through four steps.

```
logo-big.png
    │  Step 1: binary thresholding
    ▼
logo-big-binary.png
    │  Step 2: vectorize with potrace
    ▼
logo-big-vector.svg   ◄── revise-mcp/README/logo-big.svg  (pen/icon paths)
    │  Step 3: transplant icon paths
    ▼
logo-big-vector.svg   ◄── Modica-Bold.otf
    │  Step 4: font path substitution
    ▼
logo-big.svg
```

Scripts for steps 1, 3, and 4 live in the revise-mcp project at
`/Users/davidf/Projects/revise-mcp/README/logo-big/`.

All commands below are run from the `README/` directory of this project
(`/Users/davidf/.claude/skills/revise/README/`). Steps 1–2 produce intermediate
files (`logo-big-binary.png`, `logo-big-binary.bmp`, `logo-big-vector.svg`) in
`README/` that can be deleted after the pipeline completes.

---

## Step 1: Binary Thresholding

**`logo-big.png` → `logo-big-binary.png`**

Script: `revise-mcp/README/logo-big/make_bg_transparent.py`

```bash
python /Users/davidf/Projects/revise-mcp/README/logo-big/make_bg_transparent.py \
    --binary logo-big.png
```

In `--binary` mode the script:

1. Detects the background color by taking the median of 10×10-pixel samples from each image corner.
2. Classifies each pixel by its max-channel distance from the background:
   - distance < 20 → fully transparent (background)
   - distance ≥ 20 → fully opaque, snapped to a single detected foreground color
3. The foreground color is the median RGB of pixels whose distance from the background exceeds 100.

Output is a PNG with a transparent background and all foreground pixels set to the detected ink color. Snapping all opaque pixels to one color eliminates the anti-aliased halo that would otherwise appear after vectorization.

Key constants in the script:

| Constant | Value | Meaning |
|---|---|---|
| `BINARY_THRESHOLD` | 20.0 | Distance cutoff: foreground vs. background |
| `BINARY_FG_SAMPLE_DIST` | 100.0 | Min distance for "deep foreground" color sampling |
| `CORNER_SAMPLE` | 10 | Size of corner region (px) used for bg color detection |

---

## Step 2: Vectorize

**`logo-big-binary.png` → `logo-big-vector.svg`**

Tool: [Potrace](https://potrace.sourceforge.net/) (default settings)

Potrace requires BMP input. The binary PNG is first converted with the transparent background
composited over white, so stroke pixels become dark and the background becomes white —
the polarity potrace expects:

```bash
python3 -c "
from PIL import Image
img = Image.open('logo-big-binary.png').convert('RGBA')
bg  = Image.new('RGB', img.size, (255, 255, 255))
bg.paste(img, mask=img.split()[3])
bg.save('logo-big-binary.bmp')
"

potrace --svg logo-big-binary.bmp -o logo-big-vector.svg
```

Potrace traces each connected dark region into a filled `<path>` element. The output SVG uses
the coordinate transform `translate(0,560) scale(0.1,-0.1)`, placing path coordinates in a
y-up space where 1 path unit = 0.1 pt on screen.

---

## Step 3: Transplant Icon Paths

**`logo-big-vector.svg` ← `revise-mcp/README/logo-big.svg` → `logo-big-vector.svg`**

Script: `revise-mcp/README/logo-big/transplant_pen_paths.py`

```bash
python /Users/davidf/Projects/revise-mcp/README/logo-big/transplant_pen_paths.py .
```

The blue-source potrace output has inferior pen/box/checkmark detail compared to the
red-source (revise-mcp) version. This step replaces the icon paths in the blue
`logo-big-vector.svg` with the cleaner red-source ones from
`revise-mcp/README/logo-big.svg`, while leaving the text paths untouched (they will
be replaced by font paths in step 4).

The icon region is identified by path y-start ≥ 1900 in the SVG coordinate space.
After this step `logo-big-vector.svg` contains:

| Region | Source | Paths |
|---|---|---|
| Icon (pen, box, checkmark) | revise-mcp/README/logo-big.svg | 5 |
| Text (REVISE + SKILL, noisy) | blue potrace output | varies |

---

## Step 4: Font Path Substitution

**`logo-big-vector.svg` + `Modica-Bold.otf` → `logo-big.svg`**

Script: `revise-mcp/README/logo-big/substitute_skill_text_paths.py`

```bash
python /Users/davidf/Projects/revise-mcp/README/logo-big/substitute_skill_text_paths.py .
```

Requires: `pip install fonttools`

The vectorized text has jagged edges from raster tracing. This step replaces all text paths
(y-start < 1900) with geometrically clean bezier paths rendered directly from Modica Bold,
while keeping the 5 transplanted icon paths unchanged.

**Font:** Modica Bold by [Studio Lobo](https://www.studiolobo.co/) — a geometric bold sans-serif
with uniform stroke weight, matching the original logo's letterforms. (Commercial license required;
font file not included in the repository.)

**Scale and positioning:**

- Scale: 1.0 (1 font unit = 1 SVG path unit). Modica Bold's cap height is 700/1000 UPM, which
  maps to 70 pt on screen — matching the original logo's letter height.
- REVISE baseline: y = 1100 → letters occupy screen y 380–450 pt
- SKILL baseline: y = 270 → letters occupy screen y 463–533 pt
- Both words are centered horizontally within the 560 pt canvas.
