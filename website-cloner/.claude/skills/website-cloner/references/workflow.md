# Workflow Reference

Complete phase-by-phase breakdown. Read this before orchestrating any phase.

---

## Phase 0: Setup

**Purpose**: Prepare state files and detect project type. Every other phase depends on this.

**Actions**:
1. Extract domain from URL (e.g. `stripe.com` from `https://stripe.com/pricing`)
2. Create `.tasks/clone-{domain}/screenshots/`
3. Create `public/images/`, `public/videos/`, `public/icons/` if not present
4. Detect project type by reading `package.json` — see tech-decisions.md for detection logic
5. Write `context.md` with:
   - Source URL
   - Detected framework
   - Timestamp
   - Empty sections: `## Screenshots`, `## Colors`, `## Typography`, `## Spacing`, `## Animations`

**NEVER proceed to Phase 1 until `context.md` exists and contains the framework.**

---

## Phase 1: Screenshot Capture

**Agent**: `website-screenshotter`

**Purpose**: Build a complete visual reference that the cloner and QA reviewer can work from.

| Type | Filename pattern | Viewports |
|------|-----------------|-----------|
| Full page | `full-page-{viewport}.png` | 1920×1080, 1024×768, 375×812 |
| Sections | `section-{name}.png` | 1920×1080 |
| Components | `component-{name}.png` | as needed |
| Hover states | `component-{name}-hover.png` | as needed |

After capturing, the agent appends a screenshot inventory to `context.md` under `## Screenshots`.

**Can run in parallel with Phase 2.**

---

## Phase 2: Asset & Style Extraction

**Agent**: `website-extractor`

**Purpose**: Give the cloner exact values — no guessing, no approximating.

### Assets → `public/`
| Type | Destination | Naming |
|------|-------------|--------|
| Images | `public/images/` | `{section}-{purpose}.{ext}` |
| Videos | `public/videos/` | `{section}-video.{ext}` |
| Icons | `public/icons/` | `icon-{name}.svg` |

### Design tokens → `context.md`
The extractor writes exact computed values (via `window.getComputedStyle`) for:

- **Colors**: Every unique hex — primary, secondary, backgrounds, text, borders, gradients, overlays
- **Typography**: Font families (include full stack), sizes in px, weights, line-heights, letter-spacing
- **Spacing**: Section padding, container max-widths, gaps, margins
- **Components**: Border-radius per component type, box-shadows, button variants, card styles
- **Animations**: CSS transitions, keyframes, scroll triggers, hover effects with timing values
- **Layout**: Breakpoints, grid columns, flexbox patterns

**NEVER round values** — write `17px` not `~16px`. The cloner uses Tailwind arbitrary values and needs exact figures.

**Can run in parallel with Phase 1.**

---

## Phase 3: Implementation

**Agent**: `website-cloner`

**Purpose**: Produce a single React component that visually matches the source.

**Process**:
1. Read `context.md` completely — all sections
2. If `review-notes.md` exists, read it and prioritise flagged issues before implementing new sections
3. Detect framework from `context.md` (already written in Phase 0)
4. Write component to the correct path (see tech-decisions.md)
5. Use Playwright to open both original and clone side-by-side and do a quick visual check

**Component skeleton**:

```tsx
"use client" // Next.js App Router only — omit for other frameworks

import { motion } from "motion/react"

export default function ClonePage() {
  return (
    <div className="min-h-screen">
      {/* ============================================
          NAVIGATION
          ============================================ */}
      <nav>...</nav>

      {/* ============================================
          HERO SECTION
          ============================================ */}
      <section>...</section>

      {/* Continue for every section... */}
    </div>
  )
}
```

**Depends on**: Phase 1 and Phase 2 both complete.

---

## Phase 4: QA Review

**Agent**: `website-qa-reviewer`

**Purpose**: Identify every visual gap between original and clone.

**Process**:
1. Start dev server (`npm run dev`)
2. Open original URL and `localhost` clone side-by-side in Playwright
3. Compare at each viewport: 1920×1080, 1024×768, 375×812
4. Document every discrepancy with viewport, section, and screenshot evidence
5. Write `review-notes.md` with classified issues and a single status line

**Issue classification**:

| Severity | Criteria | Examples |
|----------|----------|---------|
| Critical | Blocks usability or represents a major structural failure | Missing section, broken layout, images 404ing |
| Major | Clearly visible difference a user would notice | Wrong colour, spacing off >10px, missing hover state |
| Minor | Subtle difference requiring close inspection | Spacing off ≤10px, animation timing ±100ms |

**`review-notes.md` must end with exactly one of**:
```
STATUS: PERFECT
STATUS: ACCEPTABLE
STATUS: NEEDS_WORK
```

**Depends on**: Phase 3 complete.

---

## Phase 5: Iteration Loop

```
Read STATUS from review-notes.md

PERFECT     → Output summary. Workflow complete.
ACCEPTABLE  → Ask user: "Minor issues remain. Accept or continue refining?"
              If accept → complete. If continue → increment counter → Phase 3.
NEEDS_WORK  → Increment counter.
              If counter > 5 → stop with warning, show review-notes.md.
              Else → Phase 3.
```

---

## State Files

| File | Written by | Purpose |
|------|-----------|---------|
| `.tasks/clone-{domain}/context.md` | Phase 0, extractor, screenshotter | Accumulated design tokens and inventory |
| `.tasks/clone-{domain}/screenshots/` | screenshotter | Visual references |
| `.tasks/clone-{domain}/review-notes.md` | qa-reviewer | Issue list and status |

---

## Error Handling

| Error | Response |
|-------|----------|
| Website requires authentication | Stop immediately. Report to user with the exact URL that redirected. |
| Bot protection / CAPTCHA | Stop. Suggest the user manually save the HTML and pass it as a file. |
| Playwright navigation fails | Retry once with a 3-second delay. If it fails again, report the error verbatim. |
| Asset download 403/404 | Skip that asset, log the URL in context.md under `## Failed Assets`, continue. |
| Max iterations reached | Stop. Output the component at its current state plus the review-notes.md. Do not delete partial work. |
| Sub-agent not found | Stop immediately. Report the exact agent name that failed so the user can check their agent list. |
