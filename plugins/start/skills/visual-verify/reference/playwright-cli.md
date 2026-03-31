# Playwright CLI Reference

Exact CLI commands and flags for visual verification. This skill uses the Playwright CLI directly — no test scripts.

---

## Core Screenshot Command

```bash
npx playwright screenshot [options] <url> <output-path>
```

### Essential Flags

| Flag | Purpose | Example |
|------|---------|---------|
| `--browser` | Browser engine | `--browser chromium` (default) |
| `--viewport-size` | Width x Height | `--viewport-size 1280,720` |
| `--device` | Emulate named device | `--device "iPhone 14"` |
| `--full-page` | Capture full scrollable page | `--full-page` |
| `--wait-for-timeout` | Wait ms after load | `--wait-for-timeout 2000` |
| `--timeout` | Navigation timeout ms | `--timeout 30000` |
| `--color-scheme` | Force color scheme | `--color-scheme dark` |

### Viewport Presets

Use these as standard viewports for consistent captures:

| Name | Dimensions | Flag |
|------|-----------|------|
| Desktop | 1280x720 | `--viewport-size 1280,720` |
| Desktop HD | 1920x1080 | `--viewport-size 1920,1080` |
| Tablet | 768x1024 | `--viewport-size 768,1024` |
| Mobile | 390x844 | `--viewport-size 390,844` |

### Device Emulation

Named devices include viewport, user agent, and device scale factor:

```bash
npx playwright screenshot --device "iPhone 14" http://localhost:3000 mobile.png
npx playwright screenshot --device "iPad Pro 11" http://localhost:3000 tablet.png
npx playwright screenshot --device "Pixel 7" http://localhost:3000 android.png
```

Common devices: `iPhone 14`, `iPhone 14 Pro Max`, `iPad Pro 11`, `Pixel 7`, `Galaxy S23`.

List all available devices:
```bash
npx playwright devices
```

---

## Command Patterns

### Basic Screenshot

```bash
npx playwright screenshot \
  --browser chromium \
  --viewport-size 1280,720 \
  --full-page \
  --wait-for-timeout 1000 \
  http://localhost:3000 \
  .visual-baselines/home_desktop.png
```

### Multiple Viewports (run sequentially)

```bash
# Desktop
npx playwright screenshot --viewport-size 1280,720 --full-page \
  http://localhost:3000 .visual-baselines/home_desktop.png

# Tablet
npx playwright screenshot --viewport-size 768,1024 --full-page \
  http://localhost:3000 .visual-baselines/home_tablet.png

# Mobile
npx playwright screenshot --viewport-size 390,844 --full-page \
  http://localhost:3000 .visual-baselines/home_mobile.png
```

### Dark Mode Variant

```bash
npx playwright screenshot \
  --viewport-size 1280,720 \
  --color-scheme dark \
  --full-page \
  http://localhost:3000 \
  .visual-baselines/home_desktop_dark.png
```

### With Authentication State

If the project has a Playwright config with `storageState`:

```bash
# First check for existing auth state
cat playwright/.auth/user.json 2>/dev/null

# Use storage state if available
npx playwright screenshot \
  --browser chromium \
  --viewport-size 1280,720 \
  http://localhost:3000/dashboard \
  .visual-baselines/dashboard_desktop.png
```

Note: For authenticated routes, the user may need to provide a storage state file or manually log in first. Ask rather than assume.

---

## Wait Strategies

Screenshots may capture incomplete renders. Use wait strategies:

| Scenario | Strategy |
|----------|----------|
| Static page | `--wait-for-timeout 500` (default safe) |
| SPA with hydration | `--wait-for-timeout 2000` |
| Heavy data loading | `--wait-for-timeout 3000-5000` |
| Animations present | `--wait-for-timeout 1000` + capture after settle |

Prefer shorter waits and increase only if captures show incomplete renders.

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `browserType.launch: Executable doesn't exist` | Run `npx playwright install chromium` |
| Blank screenshot | Increase `--wait-for-timeout`, check URL is correct |
| Missing fonts / wrong rendering | Run `npx playwright install-deps` for system dependencies |
| SSL errors on localhost | URL should be `http://` not `https://` for local dev |
| Screenshot too large (> 10MB) | Remove `--full-page` or capture specific viewport only |

---

## Directory Structure

```
.visual-baselines/
├── home_desktop.png
├── home_tablet.png
├── home_mobile.png
├── about_desktop.png
├── about_tablet.png
├── about_mobile.png
├── .current/                  # Temporary captures for comparison
│   ├── home_desktop.png
│   └── ...
└── manifest.json              # Route + viewport + timestamp index
```

### Manifest Schema

```json
{
  "version": 1,
  "capturedAt": "2026-03-31T12:00:00Z",
  "baseUrl": "http://localhost:3000",
  "routes": [
    {
      "path": "/",
      "slug": "home",
      "viewports": ["desktop", "tablet", "mobile"],
      "capturedAt": "2026-03-31T12:00:00Z"
    }
  ]
}
```

### Route Slug Generation

Convert route paths to safe filenames:
- `/` → `home`
- `/about` → `about`
- `/blog/post-1` → `blog--post-1`
- `/dashboard/settings` → `dashboard--settings`
- `/api/v2/users` → `api--v2--users`

Rule: replace `/` with `--`, strip leading/trailing slashes, use `home` for root.
