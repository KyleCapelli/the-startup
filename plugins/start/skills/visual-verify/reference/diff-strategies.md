# Diff Strategies

How to compare screenshots and classify visual changes. Prioritize approaches by availability.

---

## Comparison Approach Priority

Try in order — use the first available:

### 1. ImageMagick `compare` (Preferred)

Most commonly available on dev machines. Produces both a diff metric and a visual diff image.

```bash
# Check availability
compare --version 2>/dev/null && echo "available" || echo "not found"

# Pixel comparison — outputs diff metric and diff image
compare -metric AE \
  .visual-baselines/home_desktop.png \
  .visual-baselines/.current/home_desktop.png \
  .visual-baselines/.diffs/home_desktop_diff.png 2>&1
```

The `AE` (Absolute Error) metric returns the count of different pixels. Calculate percentage:

```bash
# Get total pixel count
identify -format "%[fx:w*h]" .visual-baselines/home_desktop.png

# diff_percent = (diff_pixels / total_pixels) * 100
```

### 2. Playwright Built-in Comparison

If ImageMagick is not available, use Playwright's `toHaveScreenshot` approach via a minimal inline script:

```bash
# Generate a diff using Playwright's comparator
node -e "
const { comparePNGs } = require('playwright-core/lib/utils');
const fs = require('fs');
const baseline = fs.readFileSync('.visual-baselines/home_desktop.png');
const current = fs.readFileSync('.visual-baselines/.current/home_desktop.png');
const result = comparePNGs(baseline, current, { threshold: 0.2 });
console.log(JSON.stringify({ diffPixels: result?.diff?.length || 0, match: !result }));
"
```

Note: This API is internal and may change between Playwright versions. Prefer ImageMagick.

### 3. Manual / Fallback

If neither tool is available, report that visual diffs can't be automated and suggest:
- Side-by-side viewing in an image viewer
- Installing ImageMagick: `brew install imagemagick` / `apt install imagemagick`
- The user inspects screenshots manually

---

## Threshold Tuning

| Diff Percentage | Classification | Action |
|----------------|----------------|--------|
| < 0.1% | MATCH | Noise — anti-aliasing, subpixel rendering, font hinting |
| 0.1% — 1.0% | DRIFT (minor) | Review recommended — may be intentional styling tweak |
| 1.0% — 5.0% | DRIFT (moderate) | Review required — likely a real visual change |
| > 5.0% | DRIFT (significant) | Likely structural change — layout shift, new/removed elements |
| 100% | NEW or MISSING | No baseline exists, or dimensions changed entirely |

### Adjusting Thresholds

Some projects need tighter or looser thresholds:

| Context | Recommended Noise Threshold |
|---------|---------------------------|
| Marketing / landing pages | 0.05% (tight — pixel-perfect matters) |
| Internal dashboards | 0.5% (loose — data changes expected) |
| Dynamic content (dates, avatars) | 1.0% (very loose — mask dynamic regions) |

---

## Region Detection

When a diff is found, describe WHERE the change occurred in human-readable terms. Use the diff image pixel coordinates mapped to page regions:

### Heuristic Region Mapping

Divide the viewport into a 3x3 grid:

```
┌──────────┬──────────┬──────────┐
│ top-left │ top-mid  │ top-right│  ← Header / Nav region
├──────────┼──────────┼──────────┤
│ mid-left │  center  │mid-right │  ← Main content region
├──────────┼──────────┼──────────┤
│ bot-left │ bot-mid  │bot-right │  ← Footer region
└──────────┴──────────┴──────────┘
```

Map diff pixel clusters to grid positions:
- Top row (0–33% height) → "header area", "navigation"
- Middle row (33–66% height) → "main content", "hero section", "sidebar"
- Bottom row (66–100% height) → "footer area", "bottom CTA"

Combine with horizontal position for specificity:
- "top-right navigation" → likely a menu or auth button change
- "center main content" → layout or content change
- "bottom-left footer" → footer link or copyright update

### Advanced Region Detection (when ImageMagick available)

```bash
# Generate diff image with fuzz tolerance
compare -fuzz 5% -highlight-color red -lowlight-color none \
  baseline.png current.png diff.png

# Get bounding box of changed regions
convert diff.png -threshold 50% -trim -format "%wx%h+%X+%Y" info:
```

---

## False Positive Handling

Common sources of noise and how to handle them:

| Source | Symptom | Mitigation |
|--------|---------|------------|
| Font rendering | Slight text shifts between runs | Use 0.1% threshold |
| Animation frame | Captures mid-animation | Increase `--wait-for-timeout` |
| Cursor blink | Input fields show/hide cursor | Click away from inputs before capture |
| Dynamic content | Timestamps, avatars, live data | Note in report, suggest masking |
| Scrollbar visibility | OS shows/hides scrollbar | Set viewport slightly wider |
| Retina vs standard | DPI differences between captures | Always use same `--device` or viewport |

### When to Recommend Masking

If a route consistently produces false positives due to dynamic content, recommend the user add it to a mask list in the manifest:

```json
{
  "path": "/dashboard",
  "masks": [
    { "region": "top-right", "reason": "user avatar and timestamp" },
    { "region": "center", "reason": "live data chart" }
  ]
}
```

Masked regions are excluded from diff percentage calculations in future comparisons.
