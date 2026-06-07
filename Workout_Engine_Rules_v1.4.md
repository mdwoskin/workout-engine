# Workout Engine — Rules Document

**Version:** v1.4
**Status:** Rev 10 shipped (`we-v1.2` + WE-63 case fix `3d9b2ca`). Rev 11 Builder redesign spec'd in Part 11. Pre-build abstract for Rev 11.
**Engine prefix:** WE
**Author:** Mike Dwoskin
**Last updated:** 2026-06-07

---

## Versioning Protocol

- **Major bump (v1.0 → v2.0):** structural reorganization, new Part added, fundamental architectural shift
- **Minor bump (v1.0 → v1.1):** content additions, material edits, new rules within existing Parts
- **No bump:** typo fixes, formatting cleanup
- **Changelog version tracks pipeline iteration 1:1** (per WE-46)
- **New thread at every 15 build iterations** (formalised as WE-47 in v1.4 — moved out of this preamble)

---

## Rules-Doc Changelog

| Version | Date | Notes |
|---------|------|-------|
| v1.0 | 2026-05-14 | Pre-build abstract — initial rules doc establishing data model, architecture, UI patterns, build phasing. |
| v1.1 | 2026-05-14 | (Skipped — folded into v1.2.) |
| v1.2 | 2026-05-14 | After Rev 9. Locked singular muscle-group chips with pairing emphasis, exercise trend charts, unified all-supersets language, spreadsheet-style workout mode, Option B history display, exercise pill + Exercise Detail screen, inline preview, sign-mode toggle, keyboard input, Build/Recommendations split. |
| v1.3 | 2026-05-14 | After Rev 10 spec. Added: two-tap remove, within-section reorder + superset split/merge, primary-group grey-italic tags, save workout / save superset system, Saved Library, auto-name fallback. WE-57 through WE-63 added. |
| v1.4 | 2026-06-07 | After Rev 10 ship (`we-v1.2`) and Rev 11 spec session. **Modifies** WE-15, WE-31, WE-55, WE-61, WE-63 (recs leave Builder into picker modal; save-as-superset 2+-only top-positioned; superset auto-name `[GROUP]` derived from exercises in superset, Title Case). **Replaces** WE-57 (two-tap → swipe-to-delete) and WE-58 (5-icon cluster → three-state model). **Adds** Part 11 (WE-64–69) for Rev 11 Builder redesign. **Adds** WE-47 (15-iteration thread-cycling rule, promoted from Versioning Protocol preamble). Title Case across all auto-names (was uppercase per Rev 10 step 6; reversed post-`we-v1.2`). |

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
- **Part 10 — Edit/Move/Save Capabilities (WE-57 to WE-63)** *NEW in v1.3; WE-57/58 fully rewritten in v1.4 for Rev 11; WE-61/63 modified in v1.4*
- **Part 11 — Rev 11 Builder Redesign (WE-64 to WE-69)** *NEW in v1.4*

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

## WE-15: Builder section layout (Rev 11+)
`Applicability: Builder`

*Rule history:* v1.0–v1.3 specified a Build / Recommendations split per
section. v1.4 (Rev 11) relocates recommendations into the `+ ADD
EXERCISE` picker modal (per WE-66); the Builder section layout
simplifies accordingly.

For each selected muscle group, one area:

**Build area:**
- Empty placeholder text when no exercises present
- User-constructed supersets + singleton exercises render here
- Section action buttons: `+ ADD EXERCISE` (per WE-66 — picker
  includes recs from history inline) and `+ SAVED SUPERSET` (opens
  Saved Library filtered to this section's mg)

Recommendations no longer pinned below each section. History-derived
recs surface inside the `+ ADD EXERCISE` picker per WE-66, so they
only render when the user is actively choosing what to add.

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

## WE-31: Add Exercise picker — accordion rows with history + inline recs
`Applicability: Builder`
Bottom-sheet modal. Search + `This section only / Show all` toggle (KEPT in v1.3). Exercises grouped by sub-muscle. Each row accordion-style: tap body to expand history, tap `+` to add (don't trigger expansion). Already-added: greyed with green check.

**Rev 11+ (v1.4):**
- **Recommendations surface inline within the picker**, alongside the full exercise list — relocated from the per-section Recommendations area that v1.3 had under each Builder muscle group (per WE-15 rewrite).
- The v1.3 "Start new superset / Add to current superset" placement toggle goes away. Replaced by the WE-66 "where does this go?" follow-up modal that fires after exercise pick when the section already has supersets. `+ ADD EXERCISE` only adds individual exercises; group membership is decided in the follow-up.

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

## WE-47: New thread at every 15 build iterations
`Applicability: Engine-wide`

When the current build session passes 15 meaningful iterations (commits
that materially advance the build pipeline, per WE-46), start a fresh
conversation/thread. Carry forward via `HANDOFF.md` + `CLAUDE.md` +
`OUTSTANDING_ITEMS.md` per the cross-machine handoff discipline.

*Rule history:* lived as an unnumbered bullet in the Versioning
Protocol preamble through v1.3; promoted to WE-47 in v1.4 (was
"Reserved" in v1.3). Fulfills the spec hygiene item logged in
`OUTSTANDING_ITEMS.md` at Rev 10 step 1.

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

## WE-55: Builder Build visual treatment
`Applicability: Builder`

*Rule history:* v1.3 specified the visual treatment for the Build /
Recommendations split. v1.4 (Rev 11) drops the Recommendations
half (moved into the `+ ADD EXERCISE` picker per WE-15 + WE-66);
only the Build half's visual spec remains.

Build area: solid borders, full opacity, accent color per the active
palette (teal as of 2026-06-07 — see CHANGELOG "Palette refresh"; the
`yellow accent` description in earlier rules-doc versions is
superseded). Supersets framed with the same heavier border treatment
as Workout Mode (per WE-29).

## WE-56: Empty Build area placeholder
`Applicability: Builder`
Empty: dashed-border placeholder with explanatory text. No recommendations available → Recommendations hidden entirely; placeholder simplifies.

---

# Part 10 — Edit/Move/Save Capabilities *(NEW in v1.3)*

## WE-57: Swipe-to-delete (Rev 11 — replaces v1.3 two-tap)
`Applicability: Builder`

*Rule history:* v1.3 specified a two-tap `⊖` REMOVE pattern (red icon
→ expand to REMOVE button → second tap commits). v1.4 (Rev 11) fully
replaces with a swipe gesture.

- **Two-stage swipe** on any exercise row in the Builder Build area:
  - **First swipe (left)** reveals a DELETE affordance to the right of
    the row
  - **Second swipe** (or tap on the revealed DELETE) commits the
    removal
- **No timeout** — the revealed DELETE stays open until acted on or
  collapsed
- **Outside-tap collapses** the affordance back to the resting state
- **Mutual exclusion:** revealing DELETE on one row collapses any
  other open DELETE
- **Side effects on removal:**
  - Last exercise in a superset → empty superset auto-deleted (per
    WE-58 three-state model — empty supersets are not a persistent
    state in Rev 11; `+ NEW SUPERSET` placeholders are gone)
  - Exercise originated from a saved-template injection → no rec
    re-enable (recs no longer live in the Builder per WE-15 rewrite;
    they're picker-bound per WE-66, so there's nothing to re-enable)
- **Cardio section block** also swipe-removes; if re-added later via
  the cardio chip, re-pins to top per WE-65

## WE-58: Per-exercise action cluster — three-state model (Rev 11 — replaces v1.3 five-icon cluster)
`Applicability: Builder`

*Rule history:* v1.3 specified an always-visible 5-icon cluster
(`↑ ↓ ⤴ ⤵ ⊖`) on every exercise row. v1.4 (Rev 11) replaces with a
three-state context-keyed model. `⊖` moves to swipe-to-delete (WE-57);
`⤵` merge-down is dropped entirely (merging now driven by the
singleton-with-peers merge icon → modal flow, per the table below
and WE-67).

Controls available depend on the exercise's context:

| State | Controls available |
|---|---|
| Singleton in mg, no other exercises in same mg | All greyed (only swipe-to-delete works per WE-57) |
| Singleton in mg, other exercises exist in same mg | **Merge icon** active → opens chooser modal (existing supersets in this mg + `+ NEW SUPERSET`) per WE-67. All other controls greyed |
| One of 2+ exercises in a superset | `↑` (grey at top) / `↓` (grey at bottom) / **breakout arrow** (split out into own singleton) |

- `↑` / `↓` stay as button arrows (not swipe).
- Breakout arrow replaces v1.3's `⤴`. Splits the tapped exercise out of its current superset into a new singleton inserted immediately after.
- No `⤵` merge-down icon in Rev 11. Merging is driven by the singleton-with-peers state's merge icon → modal flow only.
- **No cross-section moves.** Section boundaries hold; sections tied to chip selection (WE-30, unchanged).
- **Empty supersets do not persist** — `+ NEW SUPERSET` placeholders are gone in Rev 11. When the last exercise is removed from a superset, the superset is deleted. New supersets emerge organically via the merge-icon modal flow (WE-67) or via the `+ ADD EXERCISE` "where does this go?" follow-up modal (WE-66).

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

*Rule history:* v1.3 specified a `⭐ SAVE SUPERSET` button at the
bottom of every superset card, with `section_group` derived from the
section the save was triggered from. v1.4 (Rev 11) constrains
visibility (2+ exercises only) and moves the selector to the top of
the card. `section_group` is replaced by exercise-derived [GROUP] per
WE-63 (v1.4 superset rule).

- `⭐ SAVE SUPERSET` selector at the **top** of each superset card
- Only renders when the superset has **2 or more exercises**; singletons get no save selector
- Saves the superset's STRUCTURE: exercises in order
- `[GROUP]` for the saved entry's auto-name derives from the **exercises within the superset** per WE-63 v1.4 superset rule (first exercise's primary mg + `/` + disparate mgs in add-order, Title Case), NOT from the section it was saved from

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

**Format:** `Custom [GROUP] Workout #N` or `Custom [GROUP] Superset #N`. **Title Case throughout** (case-fixed post-`we-v1.2` per `3d9b2ca`; v1.3's earlier uppercase variant is superseded).

**`[GROUP]` determination:**
- **Workouts** — sections that actually have exercises (NOT raw chip selection — selecting `Back + Biceps` but only adding back exercises yields `Custom Back Workout #N`, not `Custom Back/Biceps Workout #N`). Title Case, `/`-joined in section insertion order. Example: `Custom Back/Biceps Workout #1`.
- **Supersets (v1.4 / Rev 11 — supersedes v1.3 section-based derivation)** — derived from **exercises within the superset**. First exercise's primary mg, then `/` + disparate mgs in add-order. Title Case. Examples: `Custom Back Superset #1` (all-back superset); `Custom Back/Biceps Superset #1` (back exercise + biceps exercise together). v1.3 used the section the superset lived in regardless of contents (`Custom Chest Superset #1` for a chest-section superset that contained a triceps exercise); v1.4 changes this so the name reflects the actual exercise composition.

**N determination — "lowest available unused N" rule:**
1. At save time, scan all currently-saved items of same TYPE and same `[GROUP]` where `isAutoNamed = true`
2. N = lowest positive integer not currently used in an auto-name within that filtered set

**Example sequence for Back muscle group:**
- Save Back workout, leave default → `Custom Back Workout #1` (`isAutoNamed: true`)
- Save another Back workout, leave default → `Custom Back Workout #2`
- Rename #1 to "Reverse Fly High Reps" → `isAutoNamed` flips to false
- Save another Back workout, leave default → `Custom Back Workout #1` (because #1 is available; only #2 is still auto-named)

The `isAutoNamed` flag is the key — it tracks whether the slot is still "owned" by the auto-naming system. User-edited names that happen to match the auto-name string format are still `isAutoNamed: false` and do not reserve slots.

---

# Part 11 — Rev 11 Builder Redesign *(NEW in v1.4)*

Spec covers the Builder UX overhaul that follows Rev 10 ship.
Supersedes WE-15/55/57/58 fully (rewritten in place above) and
modifies WE-31/61/63 (also rewritten in place above). Rules below
describe the new behaviors that didn't exist in v1.3.

## WE-64: Builder top-level layout
`Applicability: Builder`

Top of the Builder tab, in this order:

1. **Chip grid + pairing blurb** per WE-30 (unchanged; cardio chip is default-selected per WE-65)
2. **`+ SAVED WORKOUT`** — workout-level button; tap → Saved Library in WORKOUTS mode, picking an entry triggers the WE-68 overwrite-current-build action
3. **`▶ START NOW`** — uses the current build as a live workout, transitions to Workout Mode

Below: per-section build areas per WE-15 (Rev 11 single-area layout).

**`+ SAVE WORKOUT`** (save-the-current-build-as-a-template) action lives at the top of the build area as a separate selector when the build is non-empty. Distinct from `+ SAVED WORKOUT` (load) and `▶ START NOW` (use). Saves are structure-only per WE-60.

## WE-65: Cardio chip default-selected and pinned-top section
`Applicability: Builder`

- The `cardio` chip in the WE-30 chip grid is **default-selected** on Builder open.
- When `cardio` is in `selectedMG`, the cardio section renders **pinned at the top** of the build area, above all strength sections.
- If the user removes the cardio chip (deselects via the grid) and later re-adds it, the cardio section **re-pins to the top** of the build area regardless of the order in which other chips were added.
- Cardio's distinct parameters (incline, speed, distance vs. reps/weight) get the dedicated UI block per WE-20; this rule governs only its placement and default-selection state.
- The cardio block obeys swipe-to-delete per WE-57; deleting it removes the cardio section entirely but does NOT deselect the cardio chip (the chip stays on; the section re-renders empty until exercises are added).

## WE-66: `+ ADD EXERCISE` flow with inline recs + "where does this go?" follow-up
`Applicability: Builder`

`+ ADD EXERCISE` on a section opens the WE-31 exercise picker, which now surfaces history-derived recommendations inline alongside the full exercise list (recs are no longer a separate Builder area per WE-15 rewrite).

On exercise pick:

- **Section has no supersets yet:** exercise lands as a new singleton in the section.
- **Section already has supersets:** a follow-up "where does this go?" modal opens with:
  - One card per existing superset in the section, concise: exercise names only, with section header for context (no per-row history or weights — visual matches the Saved Library superset preview style for consistency)
  - Plus a `+ NEW SUPERSET` option at the bottom
  - Picking an existing superset card appends the exercise into that superset
  - Picking `+ NEW SUPERSET` lands the exercise as a new singleton

`+ ADD EXERCISE` adds individual exercises only. To insert a pre-built group, use `+ SAVED SUPERSET` (per-section) or `+ SAVED WORKOUT` (workout-level, per WE-68).

## WE-67: Merge-icon modal chooser
`Applicability: Builder`

Triggered by the merge icon in the WE-58 three-state cluster (the "singleton in mg, other exercises exist in same mg" state).

- Opens a modal listing existing supersets **in the same muscle group section** as the singleton being merged in. Concise: exercise names only, with section header.
- Plus a `+ NEW SUPERSET` option, which creates a new superset containing the tapped singleton + a target selected from a follow-up exercise picker — or alternatively (simpler MVP) just creates a 2-exercise superset by pairing the tapped singleton with one of the existing other singletons via direct pick.
- Picking an existing superset → the tapped singleton is appended into that superset.
- Modal shape mirrors the WE-66 "where does this go?" follow-up for consistency.

## WE-68: `+ SAVED WORKOUT` overwrites the current build
`Applicability: Builder, SavedLibrary`

Tapping `+ SAVED WORKOUT` (per WE-64) opens the Saved Library in WORKOUTS mode (per WE-62). Picking an entry triggers an **overwrite-current-build** action:

- `selectedMG` is cleared and re-seeded from the saved workout's sections
- `buildState` is cleared and re-seeded from the saved workout's structure (with placeholder weights/reps per the hydration rule — saved templates are structure-only per WE-60)
- The Builder screen renders the loaded build; the user can amend (sets/reps/add or remove exercises/supersets) before tapping `▶ START NOW`
- Matches the existing `▶ Use` workout clobber semantics (WE-62 action) — both paths converge on the same overwrite behavior

**No "merge into current build" option** — the action is always clobber. If the user wants to keep the current build, they cancel out of the picker.

## WE-69: iPhone long-press drag-to-group *(Phase 7 deferred)*
`Applicability: Builder (PWA / iOS Safari touch context)`

Touch-native version of the WE-67 merge-icon modal flow, modeled on iOS home-screen folder creation:

- Long-press on an exercise row → row "lifts" visually
- Drag onto another exercise row → release creates a new superset containing both, OR appends the dragged exercise into the target's existing superset
- Drag-and-release outside any target → no-op, restore original position

**Status: Phase 7 / PWA-era deferred.** Tracked here so the WE-67 merge-icon modal implementation is built in a way that doesn't make this drag flow harder to bolt on later. No code work until Phase 7 begins.

---

# Appendices

## Appendix A — Home / Dashboard
- "+ Create new workout"
- "📋 Saved templates" (per WE-62)
- "In Progress" (yellow-bordered cards)
- "Recent" (per WE-49)
- "Exercise Trends" (per WE-52 + WE-50)
- Settings gear stub

## Appendix B — Plan Builder *(Rev 11 layout — v1.4)*
- Header: workout name (editable) + back
- **Top of Builder (per WE-64):**
  - Chip grid + pairing blurb (per WE-30; cardio chip default-on per WE-65)
  - `+ SAVED WORKOUT` — workout-level; overwrites current build on selection (per WE-68)
  - `▶ START NOW` — uses the current build as a live workout
  - `+ SAVE WORKOUT` — appears at top of build area when build is non-empty; saves the current build as a template per WE-60
- **Build area (per WE-15, Rev 11 layout):**
  - Cardio section pinned top when cardio chip is selected (per WE-65); the cardio block has its own data shape per WE-20
  - One section per non-cardio selected chip
  - Per section: standard build area + section action buttons `+ ADD EXERCISE` (per WE-66 — picker includes recs) and `+ SAVED SUPERSET` (per WE-62 — opens Saved Library filtered to this section's mg)
- **Each exercise row:** name (+ off-section tag per WE-59) + three-state action cluster (per WE-58 v1.4 model) + sets row. Swipe-to-delete per WE-57 (v1.4 — replaces the v1.3 `⊖` two-tap).
- **Each superset card (2+ ex only):** `⭐ SAVE SUPERSET` selector at the **top** per WE-61 (v1.4 — was bottom in v1.3; singletons get no save selector).

The v1.3 "Save & Continue / Save as Template" bottom action bar is superseded by the WE-64 top-level layout in Rev 11.

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

**END OF v1.4**

Rev 10 shipped at `we-v1.2` (WE-57 through WE-63 + palette refresh). Next iteration: build Rev 11 in code per Part 11 (WE-64 through WE-69), supersedes WE-15/55/57/58 fully and modifies WE-31/61/63. Phase 2 (exercise library + filter) remains queued behind Rev 11.
