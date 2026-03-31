# Output Format Reference

Guidelines for visual verification output. See `examples/output-example.md` for concrete rendered examples of all three report types.

---

## Report Types

Four report types used during visual verification:

1. **Baseline Captured** — after initial screenshot capture
2. **Comparison Report** — after diffing current vs baselines
3. **Exploration Report** — after discovering and analyzing routes in explore mode
4. **Verification Report** — after checking implemented feature against spec requirements

## Diff Status Icons

| Status | Icon | Meaning |
|--------|------|---------|
| MATCH | `✅` | Visual match within noise threshold |
| DRIFT (minor) | `🟡` | < 1% diff — review recommended |
| DRIFT (moderate) | `🟠` | 1–5% diff — review required |
| DRIFT (significant) | `🔴` | > 5% diff — structural change likely |
| NEW | `🆕` | No baseline exists for this route/viewport |
| MISSING | `⚠️` | Baseline exists but route not captured (removed?) |

## Report Structure

### Header (all reports)

```
🔍 Visual Verification — {Mode}

Target: {URL or route pattern}
Viewports: {list of viewports used}
Routes: {count} captured
Timestamp: {ISO 8601}
```

### Baseline Report Body

```
📸 Baselines Captured

Route: /
  ✅ desktop (1280x720) — .visual-baselines/home_desktop.png
  ✅ tablet (768x1024) — .visual-baselines/home_tablet.png
  ✅ mobile (390x844) — .visual-baselines/home_mobile.png

Route: /about
  ✅ desktop (1280x720) — .visual-baselines/about_desktop.png
  ...

Total: {N} screenshots across {M} routes
Stored in: .visual-baselines/
```

### Comparison Report Body

Group by status, most severe first:

```
📊 Visual Comparison Results

🔴 Significant Changes (2)
  /dashboard — desktop (1280x720)
    Diff: 12.4% | Regions: header nav, main content area
    Baseline: .visual-baselines/dashboard_desktop.png
    Current:  .visual-baselines/.current/dashboard_desktop.png

  /dashboard — mobile (390x844)
    Diff: 8.7% | Regions: navigation menu, card layout
    ...

🟡 Minor Changes (1)
  /about — desktop (1280x720)
    Diff: 0.3% | Regions: footer area
    ...

✅ Matching (5)
  /, /about (tablet, mobile), /contact (all viewports)

Summary: 2 significant | 1 minor | 5 matching
```

### Exploration Report Body

```
🔍 Exploration Results

Routes Discovered: {N} from {framework} routing
Viewports: desktop, tablet, mobile

Findings:
  🟠 /pricing — mobile (390x844)
    Observation: pricing cards overflow horizontally, text truncated
    Recommendation: Check responsive breakpoint for pricing grid

  🟡 /blog — desktop (1280x720)
    Observation: large layout shift visible in hero section
    Recommendation: Check image dimensions and loading strategy

  ✅ / — all viewports
    Clean render, no anomalies detected

Screenshots saved to: .visual-baselines/.current/
```

### Verification Report Body

Group by spec requirement, show pass/fail per viewport:

```
✅ Feature Verification — Spec {NNN}

Spec: .start/specs/001-pricing-page/
Feature: Pricing page with tier comparison
Routes verified: /pricing
Viewports: desktop, tablet, mobile

📋 Verification Checklist

✅ PASS — Three pricing tiers displayed (Basic, Pro, Enterprise)
  desktop: ✅  tablet: ✅  mobile: ✅

✅ PASS — CTA button visible for each tier
  desktop: ✅  tablet: ✅  mobile: ✅

🔴 FAIL — Cards arranged horizontally on desktop, stacked on mobile
  desktop: ✅  tablet: ✅  mobile: 🔴
  Observation: Cards remain in 3-column grid on mobile, causing horizontal overflow
  Expected: Cards stack vertically below 768px per SDD section 3.2
  Recommendation: Add responsive breakpoint — grid-template-columns: 1fr below 768px

🟡 FLAG — Enterprise tier has "Contact Sales" instead of price
  desktop: 🟡  tablet: 🟡  mobile: 🟡
  Observation: Shows "Contact Sales" button, spec says "Custom pricing - Contact Sales"
  Expected: "Custom pricing" label above the button per PRD section 2.1
  Recommendation: Add pricing label text above CTA button

Summary: 2 passing | 1 failing | 1 flagged
```

## Summary Line Format

Always end with a single summary line:

```
Visual Status: {✅ CLEAN | 🟡 MINOR CHANGES | 🔴 CHANGES DETECTED}
```

For Verify mode:
```
Verification Status: {✅ ALL PASS | 🟡 PASSED WITH FLAGS | 🔴 FAILURES FOUND}
```
