# Workout Engine — Rules Document

**Version:** v1.3
**Status:** Pre-build abstract — visual + interaction spec locked through Rev 10 (including save system)
**Engine prefix:** WE
**Author:** Mike Dwoskin
**Last updated:** 2026-05-14

---

## Versioning Protocol

- **Major bump (v1.0 → v2.0):** structural reorganization, new Part added, fundamental architectural shift
- **Minor bump (v1.0 → v1.1):** content additions, material edits, new rules within existing Parts
- **No bump:** typo fixes, formatting cleanup
- **Changelog version tracks pipeline iteration 1:1** (per WE-46)
- **New thread at every 15 build iterations**

---

## Rules-Doc Changelog

| Version | Date | Notes |
|---------|------|-------|
| v1.0 | 2026-05-14 | Pre-build abstract — initial rules doc establishing data model, architecture, UI patterns, build phasing. |
| v1.1 | 2026-05-14 | (Skipped — folded into v1.2.) |
| v1.2 | 2026-05-14 | After Rev 9. Locked singular muscle-group chips with pairing emphasis, exercise trend charts, unified all-supersets language, spreadsheet-style workout mode, Option B history display, exercise pill + Exercise Detail screen, inline preview, sign-mode toggle, keyboard input, Build/Recommendations split. |
| v1.3 | 2026-05-14 | After Rev 10 spec. Added: two-tap remove, within-section reorder + superset split/merge, primary-group grey-italic tags, save workout / save superset system, Saved Library, auto-name fallback. WE-57 through WE-63 added. |

---

## Document Structure

- **Part 1 — Universal Engine Rules (WE-1 to WE-9)**
- **Part 2 — Cross-Feature Patterns (WE-10 to WE-20)**
- **Part 3 — Workflow (WE-21 to WE-24)**
- **Part 4 — UI/UX Rules (WE-25 to WE-33)**
- **Part 5 — Database & Data Aggregation (WE-34 to WE-39)**
- **Part 6 — Build Phasing (WE-40 to WE-42)**
- **Part 7 — Open Items & Deferred (WE-43 to WE-44)**
- **Part 8 — Reference (WE-45 to WE-47)**
- **Part 9 — Visual & Interaction Spec (WE-48 to WE-56)**
- **Part 10 — Edit/Move/Save Capabilities (WE-57 to WE-63)** *NEW in v1.3*

Applicability tags: `Home | Builder | Workout | History | ExerciseDetail | SavedLibrary | Library | Engine-wide`

---

# Part 1 — Universal Engine Rules

## WE-1: Long-format data model is sacrosanct
`Applicability: Engine-wide`
All performance data stored long-format (one row per set), never wide-format. New exercises require zero schema changes.

## WE-2: ID conventions
`Applicability: Engine-wide`
UUID v4 strings for all entity IDs. `set_number`, `superset_id`, `position_in_superset` are 1-indexed positive integers. IDs immutable.

## WE-3: Date handling
`Applicability: Engine-wide`
ISO 8601 strings. Workout date = COMPLETE button timestamp. Eastern Time canonical for display; UTC stored internally.

## WE-4: Persistence layer
`Applicability: Engine-wide`
IndexedDB via Dexie.js. v1: local-only, manual JSON export. No cloud sync.

## WE-5: Edits memorialize via overwrite
`Applicability: History, Workout`
Edits silently overwrite. No audit trail. Most recent value is canonical.

## WE-6: Engine versioning
`Applicability: Engine-wide`
Engine and schema versions in `meta` IndexedDB store. Schema bumps trigger migrations.

## WE-7: Backup limitations to flag
`Applicability: Engine-wide`
- IndexedDB is per-browser, per-device
- iOS Safari may evict IndexedDB data from PWAs unopened ~7 weeks
- No multi-device sync without manual export/import

## WE-8: Single-file deliverable preference
`Applicability: Engine-wide`
v1: single `index.html` + minimal JS/CSS. Phase 7 PWA wrapper adds installability. No build step in v1.

## WE-9: Mobile-first, desktop-tolerant
`Applicability: Engine-wide`
Primary viewport: 390px. Builder tested at desktop width. Workout Mode and History Detail mobile-only optimized.

---

# Part 2 — Cross-Feature Patterns

## WE-10: Rep-matching uses Option D (pick from history table)
`Applicability: Builder, Workout`
No algorithmic rep-matching. Display last 5 instances with all rep×weight combinations. User enters target reps/weight manually. Engine surfaces history, never auto-selects.

## WE-11: "Last 5 instances" is exercise-scoped, not date-scoped
`Applicability: Builder, Workout, History, ExerciseDetail`
Query: `SELECT * FROM workout_logs WHERE exercise_id = ? ORDER BY date DESC LIMIT 5`. History strip open by default in picker; collapsed behind `▾` in workout mode. Pad with "no prior data" placeholders if fewer than 5.

## WE-12: Superset definition — unified terminology
`Applicability: Builder, Workout, History`
Every exercise group is a "superset," regardless of count:
- 1-exercise superset = sequential sets with normal rest
- 2- or 3-exercise superset = exercises performed back-to-back
- Section boundaries always break supersets
- Cardio is its own isolated section by nature
- All supersets get identical visual framing (yellow vertical accent bar)
- "— ISOLATED —" terminology is deprecated and removed

## WE-13: Superset set-cycling behavior
`Applicability: Workout`
- Multi-exercise superset: cycle through ALL exercises before incrementing set
  - [Pull-Ups, Curls] × 4 sets → P1 → C1 → P2 → C2 → P3 → C3 → P4 → C4
- Single-exercise superset: sets advance sequentially
- Across supersets: complete current superset entirely before moving on
- ACTIVE badge moves with cycling pointer

## WE-14: Plan vs. Log are separate entities
`Applicability: Engine-wide`
`workout_plan` and `workout_log` tables distinct. "Complete Workout" is the only action that creates a log row. Plans without logs don't count toward stats or appear in Recent.

## WE-15: Builder uses Build/Recommendations split
`Applicability: Builder`

For each selected muscle group, two areas:

**Build area (top, empty by default):**
- Empty placeholder text
- User-constructed supersets render here
- Buttons: `+ ADD EXERCISE`, `+ NEW SUPERSET`, `📋 INSERT SAVED SUPERSET`

**Recommendations area (below, divider line):**
- Header: "Recommended · from last [GROUP] workout" + `+ Pull all`
- Each recommended superset as dashed `rec-card` with `+ Pull in` and per-exercise `+`
- Pulled recs grey out, show `✓ Pulled in`
- No history → Recommendations area hidden entirely

**Recommendations stay scoped to the originally-selected chip muscle group.** Cross-group exercises pulled in per WE-58 don't cause Recommendations to reshuffle.

## WE-16: Workout naming convention
`Applicability: Engine-wide`
Auto-generated: `YYYY.MM.DD - [GROUP 1] / [GROUP 2] / [GROUP 3]`
- Groups from chip selection (per WE-30), NOT derived from exercise content
- Cross-group exercises added per WE-58 don't modify the workout name
- User can override the auto-name
- Cardio not auto-included (always implicit, top of workout)

## WE-17: Workout date = COMPLETE button timestamp
`Applicability: Workout`
Stamped at completion. Plans never completed don't get a date. Manual override available in History Detail.

## WE-18: Warm-up tracking deferred
`Applicability: Engine-wide`
No warm-up tracking in v1. Deferred to v2: `is_warmup = true` tag.

## WE-19: Exercise library is closed for v1
`Applicability: Library`
Read-only, seeded from cleaned Control sheet. New exercises added via Claude Code request. "Goblet Spuat" → "Goblet Squat" fix.

## WE-20: Cardio has a dedicated data shape
`Applicability: Builder, Workout, History`
Top of every workout, separate from muscle group sections. Fields: `cardio_type | duration_min | distance_mi | avg_speed_mph | incline_pct | notes`. No superset framing. Always available regardless of Cardio chip selection.

---

# Part 3 — Workflow

## WE-21: Canonical user flow
`Applicability: Engine-wide`
1. Home: "+ Create new workout" OR "📋 Saved templates"
2. Builder: select chips (or skip if from template)
3. Builder: Recommendations auto-populate below
4. Builder: pull recommendations, insert saved supersets, add manually, create new empty supersets
5. Builder: adjust the plan (reorder, split/merge supersets, remove, save items as templates)
6. Builder: "Save & Continue" or "▶ Start now"; optional "Save as Template"
7. Workout Mode: log each set
8. Workout Mode: "Complete Workout" → date stamped, log row created
9. Home: workout in "Recent"
10. History Detail / Exercise Detail: edit cells, drill into per-exercise history

## WE-22: Plan state machine
`Applicability: Builder, Workout, Home`
States: `draft → planned → in_progress → completed` (any → `deleted`). In-progress shows "▶ Resume."

## WE-23: Complete-workout commits instantly
`Applicability: Workout`
No confirmation dialog. Exception: zero-sets-logged → single confirm prompt.

## WE-24: Resume in-progress plans where left off
`Applicability: Workout`
Reopening lands user in Workout Mode with prior sets preserved. Pointer advances to next un-logged set per WE-13.

---

# Part 4 — UI/UX Rules

## WE-25: Visual conventions are locked
`Applicability: Engine-wide`

Approved through Rev 10:
- Superset framing: yellow vertical accent bar (3px) + "SUPERSET" label
- No isolation divider
- Active state: yellow badge "ACT" + 2px border + pulse animation
- Done state: green check / green-tinted cell
- Empty state: dashed border, em-dash
- Shadow state: muted grey, faint text (planned weight)
- Section header: uppercase muscle group name + count meta
- Exercise pill: drillable chip with `›` (per WE-50)
- Inline preview toggle: `▾ PREV` button (per WE-51)
- Primary-group tag (Rev 10): grey italic mono text shown when exercise's primary group differs from section, in ALL contexts (per WE-59)

## WE-26: Touch targets ≥ 44px
`Applicability: All screens`

## WE-27: Bottom action bar in Workout Mode
`Applicability: Workout`
Sticky bar:
1. Active label
2. Sign-mode toggle + weight stepper + LOG button
3. Quick-add buttons (bw, 5, 10, 25) respecting sign-mode
4. COMPLETE WORKOUT button

## WE-28: History strip behavior
`Applicability: Builder, Workout`
Picker: open by default (Option B). Workout Mode: collapsed behind `▾`.

## WE-29: Workout Mode = spreadsheet-style cells
`Applicability: Workout`
One row per exercise, sets stretch left-to-right (S1-S4). REPS column shared per exercise. Cell states: Done (green), Active (yellow pulse), Shadow (grey faint), Empty (dashed).

## WE-30: Singular muscle group chips with pairing emphasis
`Applicability: Builder`
8 chips: Back, Biceps, Chest, Triceps, Shoulders, Legs, Core, Cardio. Multi-select toggleable. Pairing emphasis (good = glow, bad = ~35% opacity). Pairing logic hardcoded. Blurb formats:
- 1 chip → "You've paired Biceps with: Back 64% · Triceps 28% · Shoulders 8%"
- 2 chips → "Last paired Biceps + Back on May 10 · 14× in last 90 days"
- 3+ chips → hidden

## WE-31: Add Exercise picker — accordion rows with history
`Applicability: Builder`
Bottom-sheet modal. Search + `This section only / Show all` toggle (KEPT in v1.3). Placement toggle: "Start new superset" / "Add to current superset." Exercises grouped by sub-muscle. Each row accordion-style: tap body to expand history, tap `+` to add (don't trigger expansion). Already-added: greyed with green check.

## WE-32: Edit granularity in History Detail
`Applicability: History`
Individual cell taps for single-set edits. Edit modal: stepper + sign-mode toggle + quick buttons + keyboard input.

## WE-33: Mobile-first typography
`Applicability: Engine-wide`
Bebas Neue (display), Inter Tight (body), JetBrains Mono (sets/history). Dark bg, yellow accent.

---

# Part 5 — Database & Data Aggregation

## WE-34: IndexedDB schema (v1.1, updated for Rev 10 save system)
`Applicability: Engine-wide`

Object stores: `exercises`, `workout_plans`, `plan_exercises`, `workout_logs`, `set_logs`, `saved_workouts`, `saved_supersets`, `meta`

**`saved_workouts`:**
```
{
  id: UUID,
  name: String,
  is_auto_named: Boolean,
  focus_groups: Array<String>,
  cardio_template: Object | null,
  created_at: ISO datetime,
  last_used_at: ISO datetime | null,
  use_count: Integer (default 0),
  source_workout_id: UUID | null
}
```

**`saved_supersets`:**
```
{
  id: UUID,
  name: String,
  is_auto_named: Boolean,
  section_group: String,
  created_at: ISO datetime,
  last_used_at: ISO datetime | null,
  use_count: Integer (default 0)
}
```

Saved items reference exercise structure via embedded subdocs. Structure-only per WE-60 means no weight/rep values stored.

## WE-35: Indexes for query performance
`Applicability: Engine-wide`
- `set_logs.exercise_id`
- `set_logs.log_id`
- `workout_logs.workout_date`
- `workout_plans.status`
- `exercises.muscle_group`
- `saved_workouts.last_used_at`
- `saved_supersets.last_used_at`
- `saved_supersets.section_group`

## WE-36: The "last 5 instances" query pattern
`Applicability: Builder, Workout, ExerciseDetail`

```javascript
const sets = await db.set_logs
  .where('exercise_id').equals(exerciseId)
  .reverse()
  .sortBy('logged_at');

const grouped = groupBy(sets, 'log_id');
return Object.entries(grouped).slice(0, N).map(([logId, sets]) => {
  const log = await db.workout_logs.get(logId);
  return {
    date: log.workout_date,
    reps: sets[0].reps,
    weights: sets.sort((a,b) => a.set_number - b.set_number).map(s => s.weight_lb)
  };
});
```

## WE-37: Exercise trend chart data
`Applicability: Home, ExerciseDetail`
Per-set sequential layout with calendar-proportional gaps. Per WE-52.

## WE-38: Migration path from Excel
`Applicability: Engine-wide`
Phase 3 one-time import. Treadmill → cardio. DB Bench Press → standard set_logs. Flag `imported_from_excel = true`.

## WE-39: Export format for backup
`Applicability: Engine-wide`
Manual export → JSON file with all stores including saved items. Import accepts same shape.

---

# Part 6 — Build Phasing

## WE-40: Eight-phase build plan
`Applicability: Engine-wide`

- **Phase 0 — Rules Doc (v1.3):** ✓ COMPLETE
- **Phase 1 — Visual prototype (Rev 1-9):** ✓ COMPLETE in chat
- **Phase 1B — Rev 10 spec:** ⏳ DECISIONS COMPLETE, BUILD NOT YET DONE
- **Phase 2 — Exercise library + filter**
- **Phase 3 — IndexedDB schema + seed** (includes saved_* stores)
- **Phase 4 — Shadow population + history queries**
- **Phase 5 — Workout Mode set logging**
- **Phase 6 — Complete workout + History Detail + Exercise Detail**
- **Phase 6B — Save system** (Saved Library, Save buttons, Insert Saved Superset picker)
- **Phase 7 — PWA wrapper + export**

## WE-41: Build tool is Claude Code
`Applicability: Engine-wide`
Primary build environment: Claude Code. Phase 1 Rev 9 HTML is canonical visual reference; Rev 10 deltas specified in WE-57 through WE-63.

## WE-42: Each phase ends with a tag and changelog entry
`Applicability: Engine-wide`
Git tags `we-v1.N`. Changelog appended in rules doc + `CHANGELOG.md`.

---

# Part 7 — Open Items & Deferred Decisions

## WE-43: V2 candidate features
- Drag-to-reorder (Rev 10 uses ↑↓⤴⤵ buttons instead)
- Fuzzy matching of focus areas
- In-app exercise creation
- Warm-up set tagging
- Heart rate / additional cardio fields
- Multi-device cloud sync
- Trend chart enhancements (1RM estimates, PR markers)
- Rest timer between sets
- Plate calculator
- Save WITH weights/reps (v1.3 saves structure only per WE-60)
- Section auto-rename based on content (v1.3 keeps names tied to chip selection)

## WE-44: Known nuances post-v1
- Same-day duplicate workouts — dedup logic deferred
- Cross-section supersets — not supported
- Bodyweight exercises with added weight — weight field is `Number | "bw"`

---

# Part 8 — Reference

## WE-45: Engine prefix in commits
`WE:` prefix. Examples: `WE: Phase 1 skeleton complete`, `WE-32: cell-level edit modal`.

## WE-46: Changelog version = pipeline iteration 1:1
Each iteration producing a meaningful artifact gets a version number. Rules doc bumps when rules change.

## WE-47: Reserved
Placeholder.

---

# Part 9 — Visual & Interaction Spec

## WE-48: REPS column shared per exercise
`Applicability: Builder, Workout, History`
REPS represents count for ALL sets of that exercise. Tap to override per set. Avoids forcing repeated entry.

## WE-49: Home Recent rows inline-expandable
`Applicability: Home`
Tap to expand. Expanded view = full workout in compressed spreadsheet. Actions: "Open" + "Duplicate."

## WE-50: Exercise-name pill + Exercise Detail screen
`Applicability: History, Home, ExerciseDetail`

Pill: small chip with `›`, above sets table in History; equivalent "View full history ›" link in Home Exercise Trends.

Exercise Detail:
- Header: name + "N instances · last [date]" + back
- Top: dual chart (weight + reps per set)
- Below: "Full History · newest first" with mini-spreadsheet cards, tap → that workout's History Detail
- Back returns to source screen

## WE-51: Inline history preview in History tab
`Applicability: History`
Each row has `▾ PREV` button. Tap reveals dashed-border panel showing last 3 prior workouts (excluding current) in Option B. Independent toggles.

## WE-52: Exercise trend chart specification
`Applicability: Home, ExerciseDetail`

**Weight chart:**
- X-axis: workouts left (oldest) to right (newest), calendar-proportional gaps
- Within each workout: each set as separate point left-to-right
- Connecting line through all points sequentially
- Y-axis auto-scaled +20% padding
- Bodyweight: line flat at midline, dots still rendered
- Date labels per workout slot; gap labels ("+Nd")

**Reps chart:**
- Same X-axis as weight chart (alignment preserved)
- Each set as small bar
- Vertically aligned with weight dots

## WE-53: Sign-mode toggle (+/−)
`Applicability: Workout, History`
Sticky toggle in log bar AND edit modal. Minus mode: button red, quick-add buttons (5/10/25) red AND subtract. `bw` mode-independent. Stepper +/− always direction-explicit. Default `+`. Independent per modal.

## WE-54: Keyboard input for weights
`Applicability: Workout, History`
Weight values tappable. Tap replaces with numeric `<input>` (iOS `inputmode="decimal"`). Empty commits as `bw`. Enter or blur commits.

## WE-55: Builder Build/Recommendations split
`Applicability: Builder`
Build: solid borders, full opacity, yellow accent. Recommendations: dashed borders, ~85% opacity. Pulled recs: ~40% with green-bordered "✓ Pulled in." Divider line separates the two.

## WE-56: Empty Build area placeholder
`Applicability: Builder`
Empty: dashed-border placeholder with explanatory text. No recommendations available → Recommendations hidden entirely; placeholder simplifies.

---

# Part 10 — Edit/Move/Save Capabilities *(NEW in v1.3)*

## WE-57: Two-tap remove pattern
`Applicability: Builder`
- Each exercise row has a small red `⊖` icon (circle with minus) in its action cluster
- **First tap:** expands horizontally into a `REMOVE` button (red bg, white text)
- **Second tap on REMOVE:** exercise removed
- **No timeout** — expanded button stays open until acted on
- **Outside-tap collapses** back to `⊖`
- **Mutual exclusion:** expanding REMOVE on one exercise collapses any other expanded REMOVE
- **Side effects on removal:**
  - Last exercise in a superset → empty superset auto-deleted (per WE-58)
  - Exercise originated from pulled recommendation → recommendation re-enables (returns to bright `+ Pull in`)

## WE-58: Within-section move controls
`Applicability: Builder`

Action cluster on every exercise row in Build area, always-visible (not behind edit toggle):
- **`↑`** — move up within current superset; greyed at top
- **`↓`** — move down within current superset; greyed at bottom
- **`⤴`** — pull out into new single-exercise superset inserted immediately after current
- **`⤵`** — merge with superset directly below; greyed if none
- **`⊖`** — remove (per WE-57)

All small (~24px), tight cluster.

**No cross-section moves.** Section boundaries hold; sections tied to chip selection. Cross-group exercises added via picker's "Show all" toggle (per WE-31) and tagged per WE-59.

Empty supersets auto-delete (after last exercise removed/moved), EXCEPT supersets created by `+ NEW SUPERSET` — those persist as empty placeholders until populated or manually removed.

## WE-59: Primary-group tag (grey italic)
`Applicability: Builder, Workout, History, ExerciseDetail`
- When exercise's primary group ≠ section it lives in → render primary group as a tag adjacent to the name
- Format: small uppercase mono, **grey italic**, e.g., `Triceps Pulldown *TRI*`
- Only shown when off-section
- Shown consistently in all contexts: Build, Workout, History, ExerciseDetail
- Does NOT change section header name (per WE-16)

## WE-60: Save Workout — structure only
`Applicability: Builder`
- "Save as Template" button in Builder action bar
- Saves STRUCTURE only (exercises, supersets, sections) — no weights, no reps, no logged values
- Reasoning: weights change every workout; saving them defeats the recommendation engine
- Includes: focus chips, all sections, all supersets, all exercises, cardio template if present
- Cardio saved as TYPE only (no duration/speed values)

## WE-61: Save Superset — structure only
`Applicability: Builder`
- `⭐ SAVE SUPERSET` button at bottom of each superset card
- Saves the superset's STRUCTURE: exercises in order
- Tagged with `section_group`: muscle group of section where saved (e.g., saving from Chest section → `section_group = "Chest"`, regardless of contained exercises)

## WE-62: Saved Library screen
`Applicability: SavedLibrary`

Entry points:
- Home: `📋 Saved templates` button next to "+ Create new workout"
- Builder: `📋 INSERT SAVED SUPERSET` button (opens filtered to Supersets, scoped to current section)

Layout:
- Header: "Saved Items" + back
- Top toggle: `WORKOUTS` / `SUPERSETS`
- Card per saved item:
  - Custom name (or auto-name per WE-63)
  - Mini-spreadsheet preview of structure
  - Created date
  - Times used count
  - Last used date
  - Actions: `▶ Use` (pulls into Builder), `✕ Delete`

Sort: most-recently-used by default (`last_used_at DESC`, tiebreaker `created_at DESC`)

Edit name: tap to rename inline. Flips `is_auto_named` to false.

## WE-63: Save auto-name fallback logic
`Applicability: Builder, SavedLibrary`

When user doesn't provide a custom name at save time, auto-generate.

**Format:** `Custom [GROUP] Workout #N` or `Custom [GROUP] Superset #N`

**GROUP determination:**
- **Workouts:** join all selected chips with `/`. Example: `Custom Back/Biceps/Triceps Workout #1` (per Option A)
- **Supersets:** uses the SECTION the superset was saved from. Example: `Custom Chest Superset #1` (per Option A) — even if it contains a Triceps exercise from cross-group pull

**N determination — "lowest available unused N" rule:**
1. At save time, scan all currently-saved items of same TYPE and same GROUP where `is_auto_named = true`
2. N = lowest positive integer not currently used in an auto-name within that filtered set

**Example sequence for Back muscle group:**
- Save Back workout, leave default → `Custom Back Workout #1` (is_auto_named=true)
- Save another Back workout, leave default → `Custom Back Workout #2`
- Rename #1 to "Reverse Fly High Reps" → is_auto_named flips to false
- Save another Back workout, leave default → `Custom Back Workout #1` (because #1 is available; only #2 is still auto-named)

The `is_auto_named` flag is the key — it tracks whether the slot is still "owned" by the auto-naming system.

---

# Appendices

## Appendix A — Home / Dashboard
- "+ Create new workout"
- "📋 Saved templates" (per WE-62)
- "In Progress" (yellow-bordered cards)
- "Recent" (per WE-49)
- "Exercise Trends" (per WE-52 + WE-50)
- Settings gear stub

## Appendix B — Plan Builder
- Header: workout name (editable) + back + Save
- Step 1: chip grid + pairing blurb
- Step 2+: section per chip with Build + Recommendations
- Build buttons: `+ ADD EXERCISE`, `+ NEW SUPERSET`, `📋 INSERT SAVED SUPERSET`
- Each exercise row: name (+ tag if off-section per WE-59) + action cluster (`↑↓⤴⤵⊖`) + sets row
- Each superset card: `⭐ SAVE SUPERSET` at bottom
- Bottom action bar: `Save & Continue`, `Save as Template`, `▶ Start now`

## Appendix C — Workout Mode
- Header: pause, section names + timer, End
- Cardio block at top if present
- Section cards; active full styling, queued 55% opacity
- Supersets framed; spreadsheet cells per WE-29
- Sticky bottom log bar per WE-27

## Appendix D — History Detail
- Header: back, date + section names, ⋯ menu
- Edit-mode banner
- Same section/superset layout as Workout Mode
- Each row: pill (drill in) + `▾ PREV` + sets table + optional preview + off-section tag
- Bottom: "📋 Duplicate as new plan"

## Appendix E — Exercise Detail
- Header: exercise name + "N instances · last [date]" + back
- Top: dual chart per WE-52
- Below: "Full History · newest first" with tappable instance cards
- Return target tracked

## Appendix F — Saved Library *(NEW)*
- Header: "Saved Items" + back
- Top toggle: WORKOUTS / SUPERSETS
- Cards sorted most-recently-used
- Each card: name (editable inline), preview, created, use count, last used, `▶ Use`, `✕ Delete`
- Empty state: "No saved [workouts|supersets] yet."

## Appendix G — Library (stub)
v1: read-only, ~26 exercises seeded from cleaned Control sheet.

---

**END OF v1.3**

Next iteration: build Rev 10 in code (per WE-57 through WE-63), then migrate to Claude Code as real files, then begin Phase 2.
