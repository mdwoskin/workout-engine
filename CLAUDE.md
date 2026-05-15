# CLAUDE.md

Guidance for Claude Code (and other AI assistants) working in this repo.
This file is versioned with git — keep it portable, current, and brief.

---

## 1. Project Overview

**Workout Engine** — a personal mobile-first PWA replacing my Excel workout
template.

- **Stack:** single-file `index.html` (HTML + CSS + vanilla JS), IndexedDB via
  Dexie.js (not yet wired), no build step in v1 per WE-8.
- **Source:** https://github.com/mdwoskin/workout-engine
- **Live (GitHub Pages):** https://mdwoskin.github.io/workout-engine/
- **Owner:** Mike Dwoskin (sole developer + user; `@mdwoskin` on GitHub).
- **Target device:** iPhone, mobile-first 390px viewport (WE-9).

---

## 2. Working Preferences

How I want you to communicate:

- **Minimal praise.** No "great question," no "excellent point." Get to the
  analysis.
- **Structured options over open questions.** When a decision is required,
  present labeled options (a/b/c) with one-line tradeoffs and a recommendation.
- **"Red-line or proceed" framing.** After proposing a plan, ask me to red-line
  or proceed — don't ship until I confirm.
- **Strong pushback when QA is sound.** If I'm wrong about a fact, citation, or
  tradeoff, say so directly and cite the source. Don't capitulate to be
  agreeable.
- **Avoid scope creep.** Flag complexity investments explicitly. A bug fix
  doesn't need surrounding cleanup; a feature task doesn't grow new features.
- **Terse.** End-of-turn summaries are one or two sentences. No headers for
  simple answers.

---

## 3. Engineering Conventions (from Rules Doc v1.3)

These rules govern every change. Read the full text in
`Workout_Engine_Rules_v1.3.md` — citations below.

- **WE-45 — Commit prefix.** All commits begin with `WE:` or `WE-N:` where N is
  the relevant rule number. Examples: `WE: Phase 1 Rev 9 initial import`,
  `WE-57: two-tap remove pattern`.
- **WE-46 — Changelog tracks iteration 1:1.** Each meaningful artifact gets a
  version bump in `CHANGELOG.md`. Rules-doc bumps when rules change.
- **WE-8 — Single-file deliverable.** v1 is one `index.html` + minimal JS/CSS.
  **No build step. No bundler. No framework.** PWA wrapper comes in Phase 7.
- **WE-42 — Each phase ends with a tag + changelog entry.** Git tags follow
  `we-v1.N`.
- **Versioning Protocol (rules doc preamble) — New thread every 15
  iterations.** When the current Claude session passes 15 iterations, start a
  fresh conversation. Carries no WE-N tag; lives at top of rules doc.

---

## 4. File Layout

| File | Status | Role |
|---|---|---|
| `Workout_Engine_Rules_v1.3.md` | committed | **Authoritative spec.** 63 rules across 10 Parts. Read before any non-trivial change. |
| `index.html` | committed | **Current code.** Phase 1 Rev 9 visual prototype. Single source of truth for what works today. |
| `CHANGELOG.md` | committed | One entry per iteration per WE-46. Append top entries; don't edit historical ones. |
| `OUTSTANDING_ITEMS.md` | committed | Live working list. Open questions, deferred items, spec hygiene. Items resolve individually; not aged out wholesale. |
| `.gitignore` | committed | OS/editor artifacts only. v1 has no node_modules. |
| `CLAUDE.md` | committed | This file. |

When new docs are added, append to this table.

---

## 5. Build Phase Status

Per WE-40:

- ✓ **Phase 0** — Rules Doc v1.3
- ✓ **Phase 1** — Visual prototype, Rev 1–9 (deployed 2026-05-15)
- ⏳ **Phase 1B** — Rev 10 build (spec'd in WE-57 through WE-63; not yet coded)
- ☐ Phase 2 — Exercise library + filter
- ☐ Phase 3 — IndexedDB schema + seed
- ☐ Phase 4 — Shadow population + history queries
- ☐ Phase 5 — Workout Mode set logging
- ☐ Phase 6 — Complete workout + History Detail + Exercise Detail
- ☐ Phase 6B — Save system (Saved Library, Save buttons)
- ☐ Phase 7 — PWA wrapper + export

Update this section as phases complete.

---

## 6. Don'ts

- **Don't auto-commit.** Stage and propose the commit message; wait for me to
  say "commit" or "proceed."
  - *Exception:* trivial fixes within an already-approved scope (a typo, a
    syntax error you just introduced) can be folded into the next pending
    commit without a separate ask.
- **Don't push to `main` without explicit OK.** Even after I authorize a
  commit, confirm before pushing. `main` is the deployed branch — pushes go
  live within ~1 minute via GitHub Pages.
- **Don't propose features outside the current phase** without explicit OK. If
  you spot something worth doing later, add it to `OUTSTANDING_ITEMS.md`,
  don't slip it into the current change.
- **Don't write multi-file architectures in v1.** WE-8 is non-negotiable
  through Phase 6. No `src/`, no module splits, no bundler config, no
  `package.json`. One `index.html`.
- **Don't fabricate WE-N citations.** If a rule doesn't have a number, say so.
  Don't invent one to make the citation cleaner.
- **Don't capitulate when challenged with weak QA.** If your analysis is
  sound, defend it.
- **Don't write to `.claude/projects/.../memory` or similar hidden
  persistence.** Project context belongs in `CLAUDE.md` — versioned, portable,
  reviewable. Local-only hidden memory doesn't survive cloning and isn't
  auditable.
