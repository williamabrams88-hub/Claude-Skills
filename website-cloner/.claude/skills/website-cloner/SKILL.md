---
name: website-cloner
description: Clone any website with pixel-perfect fidelity into a React component using Tailwind CSS and motion. Use when the user runs /clone-website, asks to clone or replicate a website, wants to copy a site's design into their project, or needs a React component that matches an existing URL. Orchestrates four specialized sub-agents across five phases: screenshot capture, asset extraction, React implementation, and iterative QA review.
---

# Website Cloner

Orchestrates pixel-perfect website cloning via four sub-agents and a structured five-phase workflow.

---

## Before You Start

**MANDATORY**: Read [`references/workflow.md`](references/workflow.md) completely before orchestrating any phase.

For technology choices, framework detection, or import patterns:
**MANDATORY**: Read [`references/tech-decisions.md`](references/tech-decisions.md).

For sub-agent system prompts and creation instructions:
**MANDATORY**: Read [`references/subagents.md`](references/subagents.md).

**Do NOT load** `tech-decisions.md` if you are only running the workflow — load it only when making implementation decisions or answering tech questions.

---

## Decision Tree

```
User invokes /clone-website <url>
  │
  ├─ Is .tasks/clone-{domain}/context.md missing?
  │     → Yes: Start at Phase 0. Read references/workflow.md first.
  │     → No: Read review-notes.md status and resume from Phase 3.
  │
  ├─ Does review-notes.md show PERFECT?
  │     → Yes: Output summary. Done.
  │
  ├─ Does review-notes.md show ACCEPTABLE?
  │     → Ask user: accept or refine?
  │
  └─ Does review-notes.md show NEEDS_WORK (or missing)?
        → Delegate to Phase 1 (first run) or Phase 3 (iteration).
```

---

## Agent Roster

| Agent | Phase | Responsibility |
|-------|-------|----------------|
| `website-screenshotter` | 1 | Full-page screenshots at 3 viewports + hover states |
| `website-extractor` | 2 | Download assets to `public/`, extract design tokens to `context.md` |
| `website-cloner` | 3 | Build single React component using Tailwind + motion |
| `website-qa-reviewer` | 4 | Side-by-side pixel comparison, classify issues, set status |

Phases 1 and 2 can run in parallel. Phase 3 requires both to be complete.

---

## NEVER

- **NEVER import from `framer-motion`** — the correct package is `motion/react`. They look identical but `framer-motion` is the legacy package; `motion` is the maintained fork with a smaller bundle.
- **NEVER put downloaded assets in `.tasks/`** — they must go in `public/images/`, `public/videos/`, `public/icons/`. Assets outside `public/` won't be served by the dev server; every `src="/..."` path will 404.
- **NEVER skip Phase 0** — if `context.md` doesn't exist, the cloner agent has no style values to reference and will hallucinate colours and spacing.
- **NEVER start Phase 3 without reading `context.md`** — the cloner must consume the extracted design tokens before writing a single line of JSX.
- **NEVER use named CSS classes or inline styles** — all styling must use Tailwind arbitrary values (`bg-[#1a2b3c]`, `py-[120px]`). Named classes defeat the single-component, zero-config goal.
- **NEVER exceed 5 QA iterations** — if PERFECT isn't reached after 5 loops, stop and report partial output with the current review-notes.md. An imperfect clone delivered is better than an infinite loop.
- **NEVER hardcode framework output paths** — always detect from `package.json` before writing the component file. Hardcoded paths break on every framework except the one you assumed.
- **NEVER conflate agent names** — the orchestrator invokes agents by exact string name. A single character difference silently fails with no error message.

---

## Output

```
.tasks/clone-{domain}/
├── context.md          ← design tokens, structure notes, screenshot inventory
├── screenshots/        ← visual references from screenshotter
└── review-notes.md     ← QA findings and current status

public/
├── images/             ← photos, logos, backgrounds
├── videos/             ← background videos
└── icons/              ← SVGs and icons

{framework-path}        ← single React component (see tech-decisions.md)
```
