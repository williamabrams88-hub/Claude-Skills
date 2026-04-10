# Technology Decisions

Why these specific choices were made — and what NOT to use instead.

---

## Tailwind CSS with Arbitrary Values

### The non-obvious part

Tailwind's JIT compiler handles arbitrary values without any config changes:

```tsx
className="bg-[#1a2b3c] text-[#f5f5f5]"      // exact hex
className="py-[120px] px-[24px] gap-[32px]"   // exact pixels
className="shadow-[0_4px_24px_rgba(0,0,0,0.1)]" // exact shadow
className="rounded-[16px]"                    // exact radius
```

This is the only approach that achieves pixel-perfect matching without a separate CSS file or build config changes. No other method handles arbitrary exact values as cleanly.

### NEVER use these alternatives for cloning

| What to avoid | Why |
|---------------|-----|
| Plain CSS / CSS Modules | Separate files break the single-component goal; specificity cascades cause maintenance pain |
| Styled Components | Runtime CSS-in-JS adds overhead; template literals are harder to iterate with QA feedback |
| Inline `style={{}}` | Works for simple cases but breaks on pseudo-classes (`:hover`) and media queries |
| Named Tailwind classes (e.g. `bg-primary`) | Requires config extension; adds indirection that slows QA iteration |

---

## motion (not framer-motion)

**NEVER write `import { motion } from "framer-motion"`.**

`motion` is the direct evolution of framer-motion. Same API, maintained fork, smaller bundle, better tree-shaking. The old package is in maintenance mode.

```tsx
// Correct
import { motion } from "motion/react"

// Wrong — legacy package, will work but shouldn't be used
import { motion } from "framer-motion"
```

### Common patterns

```tsx
// Fade in on scroll
<motion.div
  initial={{ opacity: 0, y: 20 }}
  whileInView={{ opacity: 1, y: 0 }}
  viewport={{ once: true }}
  transition={{ duration: 0.6 }}
>

// Hover lift
<motion.button
  whileHover={{ scale: 1.02 }}
  whileTap={{ scale: 0.98 }}
>

// Staggered children
<motion.div
  initial="hidden"
  whileInView="visible"
  variants={{
    visible: { transition: { staggerChildren: 0.1 } }
  }}
>
  <motion.div variants={{ hidden: { opacity: 0 }, visible: { opacity: 1 } }}>
    ...
  </motion.div>
```

---

## Single Component Architecture

The goal is visual fidelity, not optimal code architecture. A single file:

- Maps QA feedback directly to sections without hunting across files
- Eliminates prop-drilling and import chain decisions
- Lets the user refactor into components after the clone is accepted

### Section organisation

Use multi-line comment blocks as section delimiters — not separate files:

```tsx
{/* ============================================
    HERO SECTION
    ============================================ */}
<section className="relative min-h-screen">
  ...
</section>

{/* ============================================
    FEATURES SECTION
    ============================================ */}
<section className="py-[80px]">
  ...
</section>
```

**NEVER split into multiple component files during cloning.** The user can extract sections later once the clone is accepted.

---

## Framework Auto-Detection

**NEVER hardcode the output path.** Always detect from `package.json`:

```bash
# Detection order (check in this sequence)
grep '"next"' package.json               → Next.js
grep '"@tanstack/start"' package.json    → TanStack Start
grep '"vite"' package.json               → Vite
# fallback                               → Standard React (src/pages/Clone.tsx)
```

### Output paths and special handling

| Framework | Output path | Special requirement |
|-----------|-------------|---------------------|
| Next.js App Router | `app/clone/page.tsx` | Add `"use client"` as first line |
| Next.js Pages Router | `pages/clone.tsx` | No directive needed |
| TanStack Start | `src/routes/clone.tsx` | Use TanStack route conventions |
| Vite / other | `src/pages/Clone.tsx` | Standard React export |

### Image component by framework

```tsx
// Next.js only
import Image from "next/image"
<Image src="/images/hero.jpg" alt="..." fill className="object-cover" />

// All other frameworks
<img src="/images/hero.jpg" alt="..." className="object-cover w-full h-full" />
```

**NEVER use `next/image` outside a Next.js project** — it will throw at runtime.

---

## public/ for Assets

**NEVER save assets to `.tasks/`** — the dev server doesn't serve that directory.

All frameworks serve static files from `public/`:

```
public/images/logo.png  →  referenced as  /images/logo.png
public/videos/bg.mp4    →  referenced as  /videos/bg.mp4
public/icons/check.svg  →  referenced as  /icons/check.svg
```

Assets are downloaded once and persist across QA iterations. The cloner reads paths from `context.md` and references them directly.

---

## Playwright MCP

Required for both the screenshotter and QA reviewer because it provides:

- **Real computed styles** — `window.getComputedStyle()` returns what the browser actually renders, including inherited values and CSS variable resolution
- **Real hover states** — CSS `:hover` can't be captured without actual mouse interaction
- **Side-by-side comparison** — opens two browser tabs simultaneously for QA

| Capability | Use |
|------------|-----|
| `navigate` | Load original URL and localhost clone |
| `screenshot` | Capture visual references |
| `evaluate` | Extract computed styles via JS |
| `hover` | Trigger `:hover` states before screenshotting |
| `click` | Open dropdowns, modals, interactive elements |

```javascript
// Extracting exact computed values
page.evaluate(() => {
  const el = document.querySelector('.hero-title')
  const s = window.getComputedStyle(el)
  return {
    color: s.color,
    fontSize: s.fontSize,
    fontFamily: s.fontFamily,
    letterSpacing: s.letterSpacing,
    lineHeight: s.lineHeight
  }
})
```
