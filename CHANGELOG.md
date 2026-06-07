# Workout Engine — CHANGELOG

Chronological build log. Each rev = a deliverable in chat.

---

## WE-63 case fix — Title Case — 2026-06-07
**Status: committed locally, push pending**

- Drop `.toUpperCase()` in `autoNameGroupKey()`. `mgNames` values are already Title Case (`Back`, `Biceps`, …) so the join produces `Back/Biceps Workout #N` directly. Resolves a Rev 10 step 6 design call (uppercase) that the user reversed post-`we-v1.2` ship: Rev 11 spec uses Title Case end-to-end, so leaving the deployed build on uppercase would be a contradicting state for as long as Rev 10 is current
- Comment on the saved-templates state block also updated from "group uppercase" to "Title Case mg names" so future readers don't get whiplash
- Pure case-style change. No schema, no algorithm, no UX flow change. In-memory saved entries clear on reload so there's no migration concern
- `we-v1.2` tag stays on the Rev 10 wrap commit (`ae264b6`); this is a fast-follow patch, not a re-tag

## we-v1.2 — Phase 1B Rev 10 complete — 2026-06-07
**Status: tagged locally, push pending**

Rev 10 wraps. Phase 1B build pipeline now spans WE-57 through WE-63
end-to-end, plus a palette refresh and OUTSTANDING refinements for
the upcoming Rev 11 Builder UX overhaul. Per-step detail in the
entries below; this entry is the cumulative rollup at the
`we-v1.2` annotated tag (WE-42).

- **WE-57 + WE-58 (steps 1–2)** — action cluster (`↑ ↓ ⤴ ⤵ ⊖`) + two-tap remove + within-section move logic (split / merge)
- **WE-59 (step 3)** — primary-group tag in the Builder Build and Rec areas; Workout / History / Exercise Detail expansion deferred to Phase 5/6
- **WE-60 + WE-61 (step 4)** — save-workout / save-superset buttons; in-memory `savedWorkouts` / `savedSupersets`; structure-only
- **WE-62 (step 5)** — Saved Library screen with WORKOUTS / SUPERSETS toggle, structure preview, `▶ Use` and `✕ Delete`; `useCount` schema; `hydrateInsertedEx` placeholder injection so saved-template Use doesn't crash the Build-area renderer
- **Palette refresh (between steps 5 and 6)** — teal/dark theme applied across `:root` + 18 hardcoded SVG hex literals
- **WE-63 (step 6)** — auto-name `Custom [GROUP] [Type] #N` with `[GROUP]` derived from sections that actually have exercises (uppercase, `/`-joined), `#N` = lowest available via gap-fill (scans only `isAutoNamed: true` entries), `isAutoNamed` schema field added to both saved types
- **Step 7 (this entry)** — CHANGELOG cumulative rollup; CLAUDE.md §5 Phase 1B status bumped to "Rev 10 complete (we-v1.2)"; `we-v1.2` annotated tag landed on the wrap commit per WE-42; Rev 11 OUTSTANDING refinements folded in (recommendations relocate into `+ ADD EXERCISE` picker; per-exercise action cluster three-state model — singleton-alone all-grey, singleton-with-peers merge-icon→modal, in-superset `↑`/`↓`/breakout; save-as-superset selector only on 2+ ex supersets, top-positioned — resolves the Phase 1B known issue on singleton save buttons)
- **Step 8 (pending)** — `git push origin main --tags` → GH Pages auto-deploy

## Phase 1B Rev 10 step 6 — WE-63 auto-name logic — 2026-06-07
**Status: committed locally, push pending**

- WE-63: Save Workout + Save Superset prompts now default to `Custom [GROUP] [Type] #N`. `[GROUP]` is the uppercase mg display name(s); multi-group workouts join with `/` (e.g. `Custom BACK/BICEPS Workout #3`); supersets use the section's mg. `[Type]` is `Workout` or `Superset`
- **`[GROUP]` derived from sections that actually have exercises**, NOT raw `selectedMG`. Selecting `Back + Biceps` but only adding back exercises yields `Custom BACK Workout #N`. The Save Workout handler now builds the `sections` object first, then derives `[GROUP]` from `Object.keys(sections)`. Sign-off-item-worthy: it'd be reasonable to argue raw `selectedMG` is the right source (user-intent-driven), but sections-driven matches the actual saved structure
- "Lowest available N" via new `lowestAvailableAutoNameN(items, baseName)` helper — scans `items` for entries with `isAutoNamed: true` AND `name.startsWith(baseName + ' #')`, extracts the numeric suffix, returns the smallest positive integer not in the used set. Gap-fill semantics: deleting an auto-named #2 means the next auto-save in that `[GROUP] [Type]` family picks #2, not #4
- Schema: both `savedWorkouts` and `savedSupersets` gain an `isAutoNamed: boolean` field. Set true when the final saved name equals the prompt default; false when the user typed anything different. User-edited names that happen to match the auto-name format (e.g., user types `Custom BACK Workout #99`) are treated as user-named and do NOT reserve slot #99 — matches WE-63's "user-edited names don't poison" intent
- Helper `autoNameGroupKey(mgs)` consolidates the uppercase-join logic so Save Workout (multi-mg) and Save Superset (single mg) share the same derivation
- Counter rename: `savedWorkoutSeq` → `nextSavedWorkoutId`, `savedSupersetSeq` → `nextSavedSupersetId`. They no longer carry naming semantics (just monotonic id assignment); the rename + comment update makes the intent obvious to anyone reading the state declarations
- Existing prompt mechanic preserved — cancel aborts, empty/whitespace falls back to the default (which is now the WE-63 string, so empty input auto-names per spec)
- No screen / CSS / UX changes. No `buildState` changes. No Saved Library render changes — the card just shows `item.name` regardless of source, which is correct
- HANDOFF.md updated: step 6 marked DONE, NEXT → step 7 (CHANGELOG + CLAUDE.md updates + `we-v1.2` tag)
- OUTSTANDING_ITEMS.md: WE-63 line struck

## Palette refresh — teal/dark — 2026-06-07
**Status: committed locally, push pending**

- New CSS variables in `:root` per the OUTSTANDING_ITEMS "## Palette" spec (approved 2026-06-07): `--bg: #0A0F0E` (deepest teal-dark page bg); `--bg-1` / `--bg-2`: `#1C2725` (flat surface/surf_el per spec); `--bg-3`: `#243330` (derived micro-surface); `--border`: `#243330` (matches bg-3) / `--border-hi`: `#2D4543` (derived); `--accent`: `#1F8A7E` (`accent`/`data`); `--accent-dim`: `#216F66` (txt_s repurposed — no dim-accent token in spec); `--ok`: `#1F8A7E` (= accent per spec's flat success); `--danger`: `#B14646` (destruct)
- **Text mapping non-literal** — spec's `txt_p: #1F8A7E` would collide with accent and fail AA contrast on the new bg, so per the recommendation accepted at decision time, `--text` uses `chart_lo` (`#B1E2DC`, ~10:1 on bg), `--text-dim` uses `chart_hi` (`#76DACE`, ~6.5:1), `--text-faint` uses `txt_s` (`#216F66`). Code comment in `:root` flags the deviation so a future v2 pass knows why
- 18 hardcoded hex literals in the SVG chart rendering (weight line + dots, reps bars, grid lines, axis labels) swapped to the new palette. Weight + reps series now both render `#1F8A7E`; shape (line vs. bars) carries the distinction that color previously did
- `--pair-good` / `--pair-bad` updated for theme consistency though both are currently unused (`.chip.pair-good` references `--accent-dim` and `.chip.pair-bad` is opacity-only). Left in `:root` for future use; not pulled out as a separate cleanup
- No JS / functional / UX changes — pure visual swap. Phone-tested 2026-06-07; approved as "perfect for now, fine tune later"
- OUTSTANDING_ITEMS.md "## Palette" section marked SHIPPED with cross-reference to this entry

## Phase 1B Rev 10 step 5 — Saved Library screen (in-memory) — 2026-06-07
**Status: committed locally, push pending**

- WE-62: new `screen-saved-library` with WORKOUTS / SUPERSETS toggle pills, card list, empty state. Header has `← BACK` that returns to whichever screen launched the library (Home or Builder)
- Entry points: Home `📋 SAVED TEMPLATES` block button (always visible regardless of saved count); per-section Builder `📋 INSERT SAVED SUPERSET` dashed button (non-cardio only) alongside `+ NEW SUPERSET`. The Builder entry sets a context (`{ mg }`) so the library opens in supersets mode pre-filtered to that section and `▶ Use` knows where to inject
- Cards render name (Bebas, like card titles elsewhere), meta row (`date · ex count · used N×`), structure preview, and `▶ Use` / `✕ Delete` actions. Workouts preview is section-grouped (`Back: Pull-Ups · Dumbbell Curls / Biceps: …`); supersets preview is a single section header + dot-separated ex names
- Sort by `savedAt` desc (ISO strings, lexicographic `localeCompare` reverse). Empty state per mode (with per-mg variant when the supersets mode is filtered to a Builder section)
- `▶ Use` on a saved workout clobbers `buildState` — clears `selectedMG` + `buildState` + `pulledRecs`, then walks the saved sections to re-seed. Re-uses the existing `updateChips()` cascade for re-render. Switches to Builder. Increments `useCount`. Matches `+ Pull in` replace semantics per the HANDOFF default
- `▶ Use` on a saved superset is only enabled when the library was opened from a Builder section (the only path where there's an unambiguous insertion target). Appends a new `{ exs, origin: 'normal' }` superset onto `buildState[mg]`, increments `useCount`, switches to Builder. Hidden in the Home-entry / no-context case (Delete still available)
- **Hydration on Use** — saved templates carry only `{ name, primary }` per WE-60/61 structure-only, but the Builder renderer needs `ex.sets` (4-cell grid) and `ex.reps`. `hydrateInsertedEx` injects placeholder `sets: ['—','—','—','—']` and `reps: '—'` at Use time so the renderer has something to draw. Phase 4 will replace these placeholders with shadow-populated values from history per WE-11. Diagnosed during 2026-06-07 phone test — initial draft skipped this and `updateSectionsArea` threw `undefined is not an object (evaluating 'ex.sets.forEach')` mid-execution, silently dropping the Use action
- `✕ Delete` splices the in-memory array and toasts `Deleted: [name]`. No confirm dialog, no undo — matches the lightweight in-memory phase. Use is also a toast (`Loaded:` / `Inserted:`)
- Schema bump: `savedWorkouts` / `savedSupersets` entries gained a `useCount: 0` field at save-time, incremented on `▶ Use`. Existing comment at the state declaration updated to reflect the new shape and that the Save handlers initialize it. No behavioral change for already-saved entries in a running session — they start at 0 the first time Use bumps them
- Delegated handler on `#sl-list` for both `▶ Use` and `✕ Delete` actions (re-renders the list on every change, so re-binding per card would be wasted). Toggle pills + `← BACK` + Home entry point are bound once on init
- New CSS: `.sl-toggle` pill row (40px, active = accent fill), `.sl-card` (border + name + meta + preview + actions), `.sl-preview` dashed-border block with section headers, `.sl-empty` dashed empty state
- No `buildState` shape changes. No Dexie. State still clears on reload (Phase 3 wires `saved_workouts` / `saved_supersets` from WE-34)
- HANDOFF.md updated: step 5 marked DONE; NEXT advances to step 6 (WE-63 auto-name logic) with phone-test review gate cleared at this step
- OUTSTANDING_ITEMS.md: WE-62 line struck

## Phase 1B Rev 10 step 4 — save buttons (in-memory) — 2026-05-16
**Status: committed locally, push pending**

- WE-60: `+ SAVE WORKOUT` full-width block button at the bottom of `#sections-area`. Renders only when `buildState` contains ≥1 exercise across the selected sections (no point saving empty). On click: prompts for a name with default `Custom Workout #N`; cancel aborts, empty/whitespace falls back to the default. Walks `buildState`, drops empty placeholder supersets, copies only `{ name, primary }` per exercise (structure-only per WE-60), pushes onto in-memory `savedWorkouts`, toasts
- WE-61: small `+ SAVE SUPERSET` inline button at the bottom of each non-empty superset card. Omitted on cardio (cardio isn't a typical superset structure for save/reuse) and on `+ NEW SUPERSET` empty placeholders. On click: same naming flow as Save Workout; copies the superset's exercises to `{ name, primary }`, captures `sectionMg`, pushes onto in-memory `savedSupersets`, toasts
- Naming: Phase 1B placeholder. WE-63 (step 6) will replace the default string with the proper `Custom [GROUP] [Type] #N` auto-name; the `prompt()` mechanic stays
- New CSS class `.save-superset-btn` — 24px tall, 10px font, sized smaller than the default `.btn.sm` to fit subtly under the superset's exercise rows
- No `buildState` shape changes — saved templates are a separate in-memory data model. No Dexie wiring (Phase 3). No Saved Library screen yet (WE-62, step 5) — saves are write-only until the screen lands
- HANDOFF.md updated: step 4 marked DONE, NEXT advances to step 5 (WE-62 Saved Library)

## Phase 1B Rev 10 step 3 — primary-group tag (Builder) — 2026-05-16
**Status: committed locally, push pending**

- WE-59: small grey-italic-mono tag renders adjacent to exercise name in the Builder when `ex.primary !== section`. Helper `primaryTag(exPrimary, sectionMg)` returns empty when on-section, emits a single `<span class="primary-tag">` otherwise
- Wired in Builder Build area (`renderBuildExRow`) and Builder Rec area (`rec-row` render). Position: immediately after the name span; flex layout puts it adjacent to the name with the existing 6px parent gap before the action cluster
- **Phase 1B scope:** Builder only. Workout Mode / History / Exercise Detail tag expansion deferred — those screens lack section-context data and would need Phase-1B shims that get thrown away when Phase 5/6 wire real plumbing. Logged in OUTSTANDING_ITEMS
- Demo data: every exercise in `shadowData` now carries a `primary` field (hardcoded). New third rec in `shadowData.back` — Dumbbell Curls (primary: `biceps`) — to make the off-section tag visible without needing the picker's "Show all" path, which lands later
- New CSS class `.primary-tag` — JetBrains Mono italic 10px, `var(--text-faint)`, smaller than the 12px `.build-ex-name` and 11px `.rec-name` it accompanies. `flex-shrink: 0` so it survives a long ex name + ellipsis
- No `buildState` shape changes (`+ Pull in` copies `primary` along with the rest via the existing spread)
- HANDOFF.md updated: step 3 marked DONE, NEXT advances to step 4 (WE-60 + WE-61 save buttons)

## Phase 1B Rev 10 step 2 — within-section move logic — 2026-05-16
**Status: committed locally, push pending**

- WE-58: `↑`/`↓` reorder exercises within their containing superset. No cross-superset moves. First/last positions stay visually greyed (from step 1 scaffold) and the helpers defensively no-op at boundaries
- WE-58: `⤴` splits the tapped exercise out into a new singleton superset inserted directly below the source. Disabled (`isAlone`) when the source has only one exercise — no meaningful split. New singleton gets `origin: 'normal'` so it auto-deletes on empty like any normal superset
- WE-58: `⤵` merges the containing superset with the one directly below. Disabled on the last superset in a section. Result lives in the source slot; target slot is removed
- Locked-in decision honored at the merge site: when target carries `origin: 'new'` (empty `+ NEW SUPERSET` placeholder), the merged result retains `'new'`, so the empty-superset auto-delete exception (WE-15 / WE-57 side effect) still applies if all exercises are later removed. Code comment at the merge site cites the HANDOFF.md "Locked-in decisions" entry
- Single dispatcher in `wireBuilderEvents` keyed on `data-move=up|down|split|merge` + `data-mg/data-ss/data-ex` coords. Move taps also collapse any expanded `⊖` REMOVE (mutual exclusion within the cluster)
- No `buildState` shape changes
- HANDOFF.md updated: build-state marker, current-state, Active Rev plan (step 2 → DONE, NEXT advances to step 3 with phone-test review gate)

## Phase 1B Rev 10 step 1 — action cluster + two-tap remove — 2026-05-15
**Status: committed locally, push pending**

- WE-57: `⊖` red icon per Build-area exercise row; first tap expands horizontally to red `REMOVE` button (no timeout); second tap commits removal; outside-tap or expanding another row's `⊖` collapses; mutual exclusion across rows
- WE-58 cluster scaffold: 5-icon cluster (`↑ ↓ ⤴ ⤵ ⊖`) renders per row; only `⊖` is wired in this step. `↑`/`↓` greyed at superset boundaries; `⤴` greyed for solo-exercise supersets; `⤵` greyed for the last superset in a section. Move logic ships in step 2.
- Side effects on remove: the superset auto-deletes when its last exercise is removed, EXCEPT supersets created by `+ NEW SUPERSET` (which persist as empty placeholder cards, each with its own `⊖`); removing an exercise that was pulled from a recommendation re-enables that rec card (back to bright `+ Pull in`)
- buildState shape upgraded: `[[ex,ex],[ex]]` → `[{exs:[ex,ex],origin:'normal'},{exs:[ex],origin:'new'}]`. Exercises gain optional `originRecIdx` for re-enable tracking
- `+ NEW SUPERSET` no longer injects a `[New exercise]` placeholder row — it creates a truly empty superset card (`origin='new'`) with its own `⊖`
- Dead code: `renderSupersetTable` removed (no callers after the new render path)
- Repo: `we-v1.1` git tag landed retroactively on `bc095cc` (Phase 1 deploy point) per WE-42, resolving the open tag-convention question in `OUTSTANDING_ITEMS.md` item 1

## Phase 1 deploy — 2026-05-15
**Status: live on GitHub Pages**

- `git init`, first commit, `.gitignore`
- Public repo: github.com/mdwoskin/workout-engine
- GH Pages enabled from `main` / root → https://mdwoskin.github.io/workout-engine/
- Docs scaffolding: CLAUDE.md, CHANGELOG.md, OUTSTANDING_ITEMS.md
- Rev 9 HTML unchanged from chat artifact; deployment infra only
- Excel data migration deferred to Phase 3 per WE-38 (Treadmill / DB Bench rows considered, punted, not forgotten)

## v1.3 rules doc + Rev 10 spec — 2026-05-14
**Status: spec captured, code NOT yet built**

- Two-tap remove: `⊖` → `REMOVE` → confirm; outside-tap collapses; no timeout
- Move buttons per exercise: `↑ ↓ ⤴ ⤵ ⊖` always-visible
- `⤴` = pull into own new superset; `⤵` = merge with superset below
- No cross-section moves; cross-group exercises added via picker's "Show all"
- Primary-group tag: grey italic mono, shown only when exercise ≠ section, in all contexts
- Save Workout / Save Superset (structure only, no weights/reps)
- Saved Library screen (WORKOUTS/SUPERSETS toggle, sort by `last_used_at`)
- Auto-name: `Custom [GROUP] [Type] #N`; multi-group workouts join with `/`; superset uses section context; "lowest available N" via `is_auto_named` flag
- New IndexedDB stores: `saved_workouts`, `saved_supersets`
- Side effects on remove: empty supersets auto-delete (except `+ NEW SUPERSET` placeholders); pulled rec re-enables

## Rev 9 — Build/Recommendations split
- Builder: empty Build area at top; Recommendations area below per section
- `+ Pull in` per rec, `+ Pull all` per section, per-exercise `+` for single pulls
- Pulled recs grey out, button changes to `✓ Pulled in`
- Empty Build = dashed placeholder

## Rev 8 — History inline preview
- `▾ PREV` button per exercise row in History
- Shows last 3 prior workouts in Option B mini-spreadsheet
- Exercise Detail subtitle: "N instances · last [date]"

## Rev 7 — Exercise Detail screen + pill pattern
- New Exercise Detail screen with full chart + full instance list
- History tab: exercise name → small framed pill with `›` chevron
- Sets table drops name column (name in pill above)
- Picker locked to Option B (A/B toggle removed)

## Rev 6.1 — A/B comparison
- Live toggle to compare Option A (`120×10, 120×10, ...`) vs Option B (mini-spreadsheet)
- Fixed broken `4×10 @ 120` format from earlier revs

## Rev 6 — Sign toggle + keyboard input + accordion picker
- `+/−` sign-mode toggle on log bar and edit modal
- Quick buttons (5/10/25) respect mode, turn red in minus mode
- Tap weight value → numeric keyboard input
- Picker rows accordion-expand to show last 3 instances

## Rev 5 — Unified supersets
- "— ISOLATED —" deprecated everywhere
- Every group called "superset" regardless of count
- "+ Insert Blank Row" → "+ NEW SUPERSET"
- Modal toggle: "Isolated/Join" → "Start new superset / Add to current superset"

## Rev 4 — Sequential per-set charts
- Each workout slot shows up to 4 set-points L→R within fixed slot
- Calendar-proportional gaps preserved between workouts
- Sequential line through all 20 points
- "+Nd" gap labels between workouts

## Rev 3 — Singular chips + trend charts + interactive editing
- 8 singular chips (Back, Biceps, Chest, Triceps, Shoulders, Legs, Core, Cardio)
- Legs unified (no split)
- Pairing emphasis: good = glow, bad = ~35% opacity
- Pairing % blurb (1 chip = top 3 historical, 2 chips = last-paired stat)
- Replaced stats cards with dual chart (weight clustered range + reps bars)
- Workout mode log button + cell tap edits wired

## Rev 2 — Refined visual spec (SVG mockups)
- Static visual spec for all 4 screens
- Locked aesthetic: industrial dark, Bebas Neue display, Inter Tight body, JetBrains Mono data
- Superset = yellow vertical accent bar + "SUPERSET" label
- Isolated = "— ISOLATED —" text divider (later deprecated in Rev 5)

## Rev 1 — Phase 1 HTML preview
- Initial interactive prototype: Home + Builder + Workout + History + Add modal
- Bottom nav, modal sheets, navigation between screens
- No persistence (IndexedDB blocked in Claude artifacts)

## v1.0 rules doc — pre-build abstract
- 47 rules across 8 Parts + 5 Appendices
- 8-phase build plan
- IndexedDB schema (5 stores, since extended)
- Plan/Log separation, long-format data model, mobile-first viewport
- Chose Option D rep-matching, last-5-instances exercise-scoped, GitHub Pages deployment

## Excel diagnostic — pre-rules
- Reviewed 2024_XX_XX_-_Workout_Template.xlsx
- Identified wide-format database as primary architectural debt
- Identified `Goblet Spuat` typo
- Confirmed long-format + IndexedDB + PWA as target architecture
