# Example Visual Verify Output

## Baseline Captured

🔍 Visual Verification — Baseline

Target: http://localhost:3000
Viewports: desktop (1280x720), tablet (768x1024), mobile (390x844)
Routes: 4 captured
Timestamp: 2026-03-31T14:22:00Z

📸 Baselines Captured

Route: /
  ✅ desktop (1280x720) — .visual-baselines/home_desktop.png
  ✅ tablet (768x1024) — .visual-baselines/home_tablet.png
  ✅ mobile (390x844) — .visual-baselines/home_mobile.png

Route: /pricing
  ✅ desktop (1280x720) — .visual-baselines/pricing_desktop.png
  ✅ tablet (768x1024) — .visual-baselines/pricing_tablet.png
  ✅ mobile (390x844) — .visual-baselines/pricing_mobile.png

Route: /about
  ✅ desktop (1280x720) — .visual-baselines/about_desktop.png
  ✅ tablet (768x1024) — .visual-baselines/about_tablet.png
  ✅ mobile (390x844) — .visual-baselines/about_mobile.png

Route: /contact
  ✅ desktop (1280x720) — .visual-baselines/contact_desktop.png
  ✅ tablet (768x1024) — .visual-baselines/contact_tablet.png
  ✅ mobile (390x844) — .visual-baselines/contact_mobile.png

Total: 12 screenshots across 4 routes
Stored in: .visual-baselines/

Visual Status: ✅ BASELINES CAPTURED

---

## Comparison Report

🔍 Visual Verification — Compare

Target: http://localhost:3000
Viewports: desktop (1280x720), tablet (768x1024), mobile (390x844)
Routes: 4 compared
Timestamp: 2026-03-31T16:45:00Z

📊 Visual Comparison Results

🔴 Significant Changes (1)
  /pricing — mobile (390x844)
    Diff: 14.2% | Regions: center main content, bottom CTA
    Baseline: .visual-baselines/pricing_mobile.png
    Current:  .visual-baselines/.current/pricing_mobile.png
    Detail: Pricing cards collapsed to single column but CTA button
            overlaps the last card. Likely a CSS grid breakpoint issue.

🟡 Minor Changes (2)
  / — desktop (1280x720)
    Diff: 0.4% | Regions: top-right header
    Detail: Navigation link order changed (likely new page added).

  /about — tablet (768x1024)
    Diff: 0.2% | Regions: footer area
    Detail: Footer copyright year updated. Expected change.

✅ Matching (9)
  / (tablet, mobile)
  /pricing (desktop, tablet)
  /about (desktop, mobile)
  /contact (all viewports)

Summary: 1 significant | 2 minor | 9 matching

Visual Status: 🔴 CHANGES DETECTED

---

## Exploration Report

🔍 Visual Verification — Explore

Target: http://localhost:3000
Viewports: desktop (1280x720), tablet (768x1024), mobile (390x844)
Routes: 6 discovered from Next.js App Router
Timestamp: 2026-03-31T18:10:00Z

📋 Route Discovery

Framework: Next.js (App Router)
Source: app/**/page.tsx
Routes found:
  / — app/page.tsx
  /pricing — app/pricing/page.tsx
  /about — app/about/page.tsx
  /contact — app/contact/page.tsx
  /blog — app/blog/page.tsx
  /dashboard — app/dashboard/page.tsx (requires auth — skipped)

🔍 Exploration Results

Findings:
  🔴 /pricing — mobile (390x844)
    Observation: Pricing cards overflow container horizontally,
                 third card partially off-screen. Horizontal
                 scroll appears on page.
    Recommendation: Check CSS grid/flex breakpoint at 390px width.
                    Cards likely need `grid-template-columns: 1fr`
                    below tablet breakpoint.

  🟠 /blog — desktop (1280x720)
    Observation: Hero image causes 250px layout shift during load.
                 Below-the-fold content jumps visibly.
    Recommendation: Add explicit width/height or aspect-ratio to
                    hero image container. Check Next.js Image
                    component usage.

  🟡 /contact — tablet (768x1024)
    Observation: Form labels misaligned with inputs at tablet width.
                 Labels float left while inputs are full-width.
    Recommendation: Check form layout CSS between 768px-1024px
                    breakpoint range.

  ✅ / — all viewports
    Clean render across all viewports, no anomalies.

  ✅ /about — all viewports
    Clean render across all viewports, no anomalies.

  ⚠️ /dashboard — skipped
    Reason: Returns 302 redirect to /login (auth required).
    Recommendation: Provide auth state file to capture
                    authenticated routes.

Screenshots saved to: .visual-baselines/.current/
Routes captured: 5 of 6 (1 skipped — auth required)

Visual Status: 🟠 FINDINGS DETECTED — 3 issues across 2 routes

---

## Verification Report

🔍 Visual Verification — Verify

Spec: .start/specs/003-checkout-flow/
Feature: Multi-step checkout with order summary
Routes verified: /checkout, /checkout/payment, /checkout/confirm
Viewports: desktop (1280x720), tablet (768x1024), mobile (390x844)
Timestamp: 2026-03-31T20:30:00Z

📋 Verification Checklist

✅ PASS — Step indicator shows current step highlighted (PRD 1.1)
  /checkout:          desktop: ✅  tablet: ✅  mobile: ✅
  /checkout/payment:  desktop: ✅  tablet: ✅  mobile: ✅
  /checkout/confirm:  desktop: ✅  tablet: ✅  mobile: ✅

✅ PASS — Order summary panel visible on all steps (PRD 1.3)
  desktop: ✅ (right sidebar)  tablet: ✅ (collapsible)  mobile: ✅ (bottom drawer)

🔴 FAIL — Payment form fields validate inline with error messages (PRD 2.2)
  /checkout/payment:
    desktop: 🔴  tablet: 🔴  mobile: 🔴
  Observation: Error messages render below the form as a batch, not inline
               next to the invalid field. Card number field shows no
               visual error state (no red border).
  Expected: Each invalid field shows red border + inline error message
            directly below the field per SDD section 4.1
  Recommendation: Add field-level validation display. Check form component
                  props — likely needs `showInlineErrors={true}`.

🔴 FAIL — Confirm page shows order total with tax breakdown (PRD 3.1)
  /checkout/confirm:
    desktop: 🔴  tablet: 🔴  mobile: 🔴
  Observation: Order total shows single "Total: $XX.XX" line.
               No subtotal, tax, or shipping breakdown visible.
  Expected: Itemized breakdown — Subtotal, Tax, Shipping, Total — per
            PRD section 3.1 acceptance criteria
  Recommendation: Check OrderSummary component — may be missing
                  `showBreakdown` prop or the breakdown data isn't
                  being passed from the checkout state.

🟡 FLAG — Mobile checkout fits in single viewport without horizontal scroll (SDD 5.2)
  /checkout:
    mobile: 🟡
  Observation: Page fits vertically but quantity selector buttons are
               very close to screen edge (4px margin). Technically
               no overflow, but touch target is tight.
  Expected: Min 8px margin from screen edge per SDD responsive guidelines
  Recommendation: Minor — increase horizontal padding on quantity controls.

✅ PASS — Back/Continue buttons present on each step (PRD 1.2)
  All routes, all viewports: ✅

✅ PASS — Responsive layout: sidebar on desktop, drawer on mobile (SDD 5.1)
  desktop: ✅ sidebar  tablet: ✅ collapsible  mobile: ✅ bottom drawer

Summary: 3 passing | 2 failing | 1 flagged
Screenshots: .visual-baselines/.current/checkout_*.png

Verification Status: 🔴 FAILURES FOUND — 2 spec requirements not met
