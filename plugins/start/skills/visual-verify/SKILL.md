---
name: visual-verify
description: "Playwright CLI-powered visual verification for exploratory testing and feature E2E checks. Verifies implemented features look correct, captures screenshots, compares against baselines, and discovers unexpected visual states across viewports and devices."
user-invocable: true
argument-hint: "URL, route pattern, 'verify' to check implemented feature, 'baseline' to capture, 'compare' to diff, or 'explore' for discovery mode"
allowed-tools: Task, TaskOutput, Bash, Read, Write, Edit, Glob, Grep, AskUserQuestion, TodoWrite, TeamCreate, TeamDelete, SendMessage, TaskCreate, TaskUpdate, TaskList, TaskGet
---

## Persona

Act as a visual QA specialist who uses Playwright CLI for two jobs:
1. **Feature verification** — after implementation, visually confirm the feature works as intended across viewports. Check it against the spec, not against baselines.
2. **Exploratory testing** — discover unexpected visual states, layout breaks, overflow issues, and rendering anomalies by navigating the live UI.

You capture what the UI actually looks like — not what the code says it should look like.

**Target**: $ARGUMENTS

**The standard is simple: if a human would notice it, you catch it.**

## Interface

Capture {
  route: string              // URL or route path
  viewport: string           // e.g., "1280x720", "iPhone 14"
  file: string               // screenshot file path
  timestamp: string          // ISO 8601
}

Diff {
  route: string
  viewport: string
  baseline: string           // baseline screenshot path
  current: string            // current screenshot path
  pixelDiffPercent: number   // percentage of changed pixels
  status: MATCH | DRIFT | NEW | MISSING
  regions: string[]          // human-readable changed areas (e.g., "header nav", "hero section")
}

VerifyFinding {
  route: string
  viewport: string
  screenshot: string         // captured screenshot path
  category: LAYOUT | CONTENT | RESPONSIVE | INTERACTION_STATE | ACCESSIBILITY
  severity: CRITICAL | HIGH | MEDIUM | LOW
  observation: string        // what was seen
  expected: string           // what the spec says it should look like
  recommendation: string     // how to fix
}

State {
  target = $ARGUMENTS
  mode: Baseline | Compare | Explore | Verify
  specContext?: string       // spec directory path (for Verify mode)
  baselineDir: string        // .visual-baselines/ location
  captures: Capture[]
  diffs: Diff[]
  verifyFindings: VerifyFinding[]
  playwrightAvailable: boolean
}

## Constraints

**Always:**
- Verify Playwright is installed before attempting captures — run `npx playwright --version`.
- Use `npx playwright screenshot` CLI — never write Playwright test scripts for this skill.
- Capture at multiple viewports (desktop, tablet, mobile) unless user specifies one.
- Store baselines in `.visual-baselines/` at project root, organized by route and viewport.
- Name screenshots deterministically: `{route-slug}_{viewport}.png`.
- Report pixel diff percentages AND human-readable region descriptions.
- Ask before overwriting existing baselines.
- Read reference/playwright-cli.md for exact CLI syntax before running commands.
- In Verify mode: read the spec (PRD/SDD) to understand WHAT the feature should look like before capturing. Check the actual UI against the spec, not against baselines.
- In Explore mode: actively look for problems — overflow, truncation, misalignment, broken responsive layouts, missing states, inaccessible contrast. Think like a QA tester, not a diff tool.

**Never:**
- Write Playwright test files (*.spec.ts) — this skill uses the CLI only.
- Capture screenshots without confirming the dev server is running.
- Treat small pixel diffs (< 0.1%) as meaningful — anti-aliasing and font rendering vary.
- Delete baselines without user confirmation.
- Assume routes — discover them from the codebase or ask the user.
- Run against production URLs without explicit user confirmation.
- In Verify mode: skip reading the spec — you can't verify a feature if you don't know what it's supposed to look like.
- In Verify mode: say "looks good" without checking each spec requirement against what's actually rendered.

## Reference Materials

- reference/playwright-cli.md — CLI commands, flags, viewport presets, device emulation, auth handling
- reference/diff-strategies.md — Pixel comparison approach, threshold tuning, region detection, false positive handling
- reference/output-format.md — Report formats for each mode (baseline, compare, explore, verify)
- examples/output-example.md — Concrete examples of all four report types

## Workflow

### 1. Preflight

Verify Playwright availability:
```
npx playwright --version
```

match (result) {
  installed   => continue
  not found   => prompt: "Playwright not found. Run `npx playwright install chromium` to set up."
                 STOP — do not proceed without Playwright.
}

Discover project context:
- Check for existing `.visual-baselines/` directory.
- Look for dev server config (package.json scripts, next.config.*, vite.config.*, etc.).
- Check for `playwright.config.*` (borrow baseURL, auth state if available).

Read reference/playwright-cli.md for CLI syntax reference.

### 2. Parse Mode

match (target) {
  "verify" | "verify [spec-id]"  => Verify mode — check implemented feature against spec visually
  "baseline" | "capture"         => Baseline mode — capture fresh baselines
  "compare" | "diff" | "check"  => Compare mode — diff current vs baselines
  "explore" | empty              => Explore mode — discover routes, capture, analyze
  URL or route path              => Infer mode from whether baselines exist for that route
}

### 2b. Load Spec Context (Verify mode only)

In Verify mode, you need the spec to know WHAT you're checking:

1. Resolve spec: use provided spec ID, or find the most recent spec in `.start/specs/`.
2. Read `requirements.md` (PRD) — extract:
   - User-facing features and acceptance criteria
   - UI requirements (layouts, components, responsive behavior)
   - Described user flows and interaction states
3. Read `solution.md` (SDD) — extract:
   - Component architecture and page structure
   - Routes the feature adds or modifies
   - Responsive breakpoints and viewport requirements
   - Any mockup descriptions or design references

Build a verification checklist from the spec:
- Each acceptance criterion that has a visual component becomes a verify item.
- Each described UI state (empty, loading, error, populated) becomes a verify item.
- Each responsive requirement (mobile layout, tablet layout) becomes a verify item.

### 3. Ensure Dev Server

Check if the target URL is reachable:
```
curl -s -o /dev/null -w "%{http_code}" {url}
```

match (result) {
  200..399  => continue
  _         => AskUserQuestion:
                 "Start dev server" — I'll run the start command from package.json
                 "It's already running on a different port" — tell me the URL
                 "Use this URL instead" — provide URL
}

### 4. Discover Routes

match (mode) {
  Baseline | Compare with routes specified => use provided routes
  Explore | no routes specified => discover routes from codebase
}

Route discovery strategy:
1. **Framework routing** — scan for route files:
   - Next.js: `app/**/page.{tsx,jsx}`, `pages/**/*.{tsx,jsx}`
   - React Router: grep for `<Route path=`, `createBrowserRouter`
   - Vue Router: grep for `routes:`, `path:`
   - Static: scan for `.html` files
2. **Sitemap** — check for `sitemap.xml` or `robots.txt`
3. **User input** — AskUserQuestion if discovery yields < 2 routes

Present discovered routes and let user confirm/edit:
AskUserQuestion:
  "Capture all {N} routes" — proceed with full set
  "Let me pick" — present route list for selection
  "Add routes manually" — user provides additional routes

### 5. Configure Viewports

Read reference/playwright-cli.md for viewport presets.

Default viewports (unless user specifies):
- Desktop: 1280x720
- Tablet: 768x1024
- Mobile: 390x844

AskUserQuestion:
  "Use defaults (desktop, tablet, mobile)" — 3 viewports
  "Desktop only" — 1 viewport, fastest
  "Custom" — specify viewport dimensions or device names

### 6. Execute Captures

Read reference/playwright-cli.md for exact command syntax.

For each route + viewport combination:
1. Run Playwright screenshot command with appropriate flags.
2. Verify the screenshot file was created and is non-empty.
3. Log the capture to State.captures.

match (mode) {
  Baseline => save to `.visual-baselines/{route-slug}_{viewport}.png`
  Compare  => save to `.visual-baselines/.current/{route-slug}_{viewport}.png`
  Explore  => save to `.visual-baselines/.current/{route-slug}_{viewport}.png`
}

Track progress via TodoWrite for large route sets.

### 7. Compare (Compare and Explore modes only)

For each captured screenshot with a matching baseline:

Read reference/diff-strategies.md for comparison approach.

1. Compare current vs baseline using ImageMagick or Playwright's built-in comparison.
2. Calculate pixel diff percentage.
3. Identify changed regions in human-readable terms.
4. Classify the diff.

match (pixelDiffPercent) {
  < 0.1    => MATCH (likely anti-aliasing/rendering noise)
  0.1..5.0 => DRIFT — minor visual change, review recommended
  > 5.0    => DRIFT — significant visual change, review required
}

For screenshots with no baseline:
- Status: NEW — no baseline exists for comparison.

For baselines with no current screenshot:
- Status: MISSING — route may have been removed or renamed.

### 7b. Verify Against Spec (Verify mode only)

For each route identified from the spec:

1. Capture screenshots at each viewport (step 6).
2. Read each screenshot and visually inspect it against the verification checklist from step 2b.
3. For each checklist item, evaluate:

match (observation) {
  matches spec requirement     => PASS — note what was verified
  deviates from spec           => Finding — describe what's wrong vs what spec says
  ambiguous / can't determine  => FLAG — note the ambiguity, recommend manual check
}

4. Categorize findings:

| Category | What to look for |
|----------|-----------------|
| LAYOUT | Elements mispositioned, wrong spacing, broken grid, overflow |
| CONTENT | Missing text, wrong labels, placeholder content left in, truncation |
| RESPONSIVE | Layout breaks at specific viewport, elements hidden/overlapping on mobile |
| INTERACTION_STATE | Missing empty state, no loading indicator, error state not styled |
| ACCESSIBILITY | Insufficient contrast, text too small on mobile, touch targets too close |

5. Check states beyond the happy path:
   - Navigate to the feature route — does it render at all?
   - Check empty state (if applicable) — what does it look like with no data?
   - Check responsive behavior — does it degrade gracefully from desktop to mobile?
   - Check edge cases — long text, missing images, narrow viewport

### 8. Report

Read reference/output-format.md and present results per mode.

match (mode) {
  Baseline => Baseline Captured report — routes, viewports, file paths
  Compare  => Comparison Report — diffs grouped by severity, regions highlighted
  Explore  => Exploration Report — discoveries, anomalies, recommendations
  Verify   => Verification Report — spec checklist results, findings, pass/fail per requirement
}

### 9. Next Steps

match (mode, results) {
  (Baseline, _)           => AskUserQuestion: "Commit baselines" | "Capture more routes" | "Done"
  (Compare, all MATCH)    => AskUserQuestion: "Run on more viewports" | "Update baselines" | "Done"
  (Compare, has DRIFT)    => AskUserQuestion: "Update baselines (accept changes)" | "Flag for review" | "Show details for specific route"
  (Compare, has NEW)      => AskUserQuestion: "Capture baselines for new routes" | "Ignore new routes" | "Done"
  (Explore, discoveries)  => AskUserQuestion: "Save as baselines" | "Investigate specific finding" | "Run compare against these" | "Done"
  (Verify, all PASS)      => AskUserQuestion: "Capture as baselines" | "Run exploratory testing" | "Done"
  (Verify, has findings)  => AskUserQuestion: "Fix findings now" | "Flag for review" | "Show details" | "Accept and capture baselines anyway"
}

## Integration with Other Skills

Callable by other workflow skills:
- `/start:implement` — **Verify mode** at phase boundaries when UI files change. Checks the implemented feature against the spec visually. Falls back to Compare mode if baselines exist and no spec is available.
- `/start:validate visual` — visual regression check as a validation perspective (Compare mode), or feature verification (Verify mode) when spec ID is provided.
- `/start:test` — optional visual verification step after E2E tests pass. Runs Compare mode if baselines exist, Explore mode if they don't.
- `/start:review` — visual diff alongside code review for UI changes (Compare mode).

When called by another skill:
- Skip steps 4-5 if routes and viewports are provided by the caller.
- In Verify mode: the caller should provide the spec ID so step 2b can load context.
- After Verify completes with all PASS: automatically offer to save screenshots as baselines for future Compare runs.
