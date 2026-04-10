# website-cloner

A Claude Code skill that clones any website into a pixel-perfect React component using Tailwind CSS and motion/react. Orchestrates four specialised sub-agents across a five-phase workflow: screenshot capture, asset extraction, React implementation, and iterative QA review.

## What it does

Run `/clone-website <url>` in any React project and the skill will:

1. Capture full-page screenshots at desktop, tablet, and mobile viewports
2. Download all images, videos, and icons to `public/`
3. Extract exact design tokens (hex colours, font stacks, pixel spacing) via `window.getComputedStyle()`
4. Build a single React component using Tailwind arbitrary values — no config changes needed
5. Run side-by-side QA in Playwright and iterate until PERFECT or ACCEPTABLE

## Requirements

- Claude Code with Playwright MCP configured
- A React project (Next.js, TanStack Start, Vite, or plain React — auto-detected)
- Tailwind CSS and `motion` (`npm install motion`) in your project

## Installation

Copy the contents of `.claude/` into your own `~/.claude/` directory:

```
~/.claude/
├── skills/website-cloner/     ← skill orchestrator
├── commands/clone-website.md  ← /clone-website slash command
└── agents/
    ├── website-screenshotter.md
    ├── website-extractor.md
    ├── website-cloner.md
    └── website-qa-reviewer.md
```

## Usage

```
/clone-website https://stripe.com/pricing
```

The skill handles everything from there. Output is a single component file at the correct path for your framework, plus assets in `public/`.

## File structure

```
.claude/
├── skills/website-cloner/
│   ├── SKILL.md                        ← orchestration logic and decision tree
│   ├── assets/clone-website.md         ← full phase workflow
│   └── references/
│       ├── workflow.md                 ← detailed phase reference
│       ├── subagents.md                ← sub-agent system prompts
│       └── tech-decisions.md           ← why Tailwind arbitrary values, motion/react, etc.
├── commands/clone-website.md           ← /clone-website trigger
└── agents/
    ├── website-screenshotter.md        ← Phase 1
    ├── website-extractor.md            ← Phase 2
    ├── website-cloner.md               ← Phase 3
    └── website-qa-reviewer.md          ← Phase 4
```
