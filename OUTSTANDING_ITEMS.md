# Outstanding Items — Workout Engine

As of 2026-05-15.

## Immediate next steps (priority order)

### 1. Build Rev 10 in code
Rev 10 spec is captured in WE-57 through WE-63 of `Workout_Engine_Rules_v1.3.md`. In progress on Phase 1B.

Starting point: current `index.html` (Phase 1 Rev 9). Apply these deltas:

- ~~**Two-tap remove** (WE-57)~~ — shipped in Rev 10 step 1.
- ~~**Move buttons** (WE-58): cluster scaffold + `⊖` shipped in step 1 (`acc51e6`); `↑ ↓ ⤴ ⤵` move logic shipped in step 2 (`35a0031`).~~ Visual/behavioral refinements parked — see "Parked from 2026-05-16 Rev 10 step 2 sign-off" section below.
- ~~**Primary-group tag** (WE-59): grey italic mono text adjacent to exercise name when primary group ≠ section. Builder Build area + Rec area shipped in step 3.~~ **Workout / History / Exercise Detail expansion deferred** — those screens lack section-context data; placeholder shims would be thrown away when Phase 5/6 wire real plumbing. Will land naturally with those phases.
- ~~**Save Workout button** in Builder action bar (WE-60). Structure only.~~ Shipped in step 4 — `+ SAVE WORKOUT` at the bottom of `#sections-area` when build is non-empty; structure-only; `prompt()` with auto-stub default.
- ~~**Save Superset button** at bottom of each superset card (WE-61). Structure only.~~ Shipped in step 4 — small `+ SAVE SUPERSET` inline button per non-empty, non-cardio superset; same naming flow.
- ~~**Saved Library screen** (WE-62)~~ — shipped in Rev 10 step 5. New `screen-saved-library` with WORKOUTS/SUPERSETS toggle. Home `📋 SAVED TEMPLATES` entry point + per-section Builder `📋 INSERT SAVED SUPERSET` (non-cardio). Cards show name, date, ex count, use count, structure preview, `▶ Use` + `✕ Delete`. Sort by `savedAt` desc. `useCount` field added to schema. `▶ Use` on workout clobbers `buildState`; on superset (only from Builder context) appends to that section.
- ~~**Auto-name logic** (WE-63)~~ — shipped in Rev 10 step 6. `Custom [GROUP] [Type] #N` with [GROUP] uppercase, joined with `/` for multi-mg workouts. Workouts derive [GROUP] from sections that actually have exercises (not raw selectedMG). `#N` = lowest available via `lowestAvailableAutoNameN` helper, scanning only entries with `isAutoNamed: true`. User-edited names get `isAutoNamed: false` and don't reserve slots — supports gap-fill on delete.
- ~~**Side effects** (WE-15, WE-57, WE-58)~~ — shipped in step 1 (empty-superset auto-delete + pulled-rec re-enable).

### 2. Phase 2: Exercise library + filter wiring
- Hardcode library from cleaned Excel Control sheet — enough entries to exercise WE-59 across multiple off-section cases (surfaced during 2026-05-16 step 3 phone test)
- Fix `Goblet Spuat` → `Goblet Squat`
- Wire picker filter (per WE-31): `This section only` vs `Show all` toggle
- **Wire the `+ ADD EXERCISE` modal's Done action to push selected exercises into `buildState`.** Currently the modal opens but doesn't update the Build area on Done — surfaced during 2026-05-16 step 3 phone test and re-confirmed 2026-06-07. WE-59 + WE-62 visual verification is bottlenecked until this works.
  - Sub-finding from 2026-06-07: inside the modal, tapping an exercise applies a green outline + "added" toast, but the `+`-in-circle icon stays as `+` instead of flipping to `✓` (or similar "selected" affordance). Need a clear visual state on selected rows so the user knows which items will land when they tap Done. Fold into the same wiring task.
- Tag each exercise with primary muscle group for WE-59 tag rendering

### 3. Phase 3: IndexedDB initialization
- Set up Dexie.js with full schema from WE-34 (now including `saved_workouts` and `saved_supersets`)
- Migrate small dataset from Mike's Excel:
  - Treadmill rows 45342-45356
  - DB Bench Press rows 45346-45350
- Flag imported records `imported_from_excel = true`
- Set up indexes per WE-35

### 4. Phase 4-7: Per WE-40 build phasing
- Phase 4: Shadow population + history queries (wire WE-15 + WE-36)
- Phase 5: Workout Mode set logging (set cycling per WE-13, log bar per WE-27)
  - **Log bar half-step `+/- .5` option** for fractional plate jumps (e.g., 2.5lb microplates). Surfaced during 2026-06-07 phone test. WE-27 currently spec'd as integer steps; check if rules-doc needs a v1.4 amendment or if "step size = .5 in fractional mode" can be a UI-only Phase 5 detail.
- Phase 6: Complete workout + History Detail + Exercise Detail (per WE-22, WE-32, WE-50)
- Phase 6B: Save system wiring (Saved Library, save buttons, insert saved superset picker)
- Phase 7: PWA wrapper + export (manifest, service worker, JSON export per WE-39)

## Rev 11 — Builder UX overhaul (proposal, locked-in 2026-06-07)

Substantial Builder redesign surfaced during 2026-06-07 phone-test
sign-off for Rev 10 step 5 and refined through the same day's
session. **Trigger to start:** Rev 10 has shipped (`we-v1.2` +
WE-63 case fix `3d9b2ca`); Rev 11 begins with a rules-doc bump to
`v1.4` since several Rev 10 rules are superseded.

### Locked-in design decisions

- **Chip grid stays** — WE-30 unchanged. User picks muscle groups at
  the top of Builder as today; sections appear per chip selection;
  pair-emphasis logic preserved. (Earlier draft of this entry said
  the chip grid went away — that was a misread of the user's
  exercise-derivation answer, which actually applies to superset
  naming, not workout-level group selection. Corrected 2026-06-07.)
- **Cardio chip is default-selected and pinned top** — Builder
  initialises with `cardio` already in `selectedMG`. The cardio
  section renders pinned at the top of the build area. If the user
  removes the cardio chip and later re-adds it, the cardio section
  re-pins to the top. Cardio's distinct parameters (incline, speed,
  distance vs. reps/weight) justify the dedicated treatment.
- **Recommendations leave the Builder** — no longer pinned below
  each muscle-group section. **New location: inside the `+ ADD
  EXERCISE` picker modal** — recs from history surface alongside the
  full exercise list. Recs only render when the user is actively
  choosing what to add. (Future v3 also surfaces recs on Home tab
  based on typical workout sequencing — out of scope for Rev 11.)
- **`+ ADD EXERCISE`** — adds individual exercises only. When the
  section already has supersets, a follow-up modal asks "where does
  this go?" with the existing supersets + `+ NEW SUPERSET` as options
  (see Behaviors). The `+ SAVED SUPERSET` button is the only path for
  inserting a pre-built group.
- **`+ SAVED SUPERSET` per section, `+ SAVED WORKOUT` workout-level.**
  `+ SAVED SUPERSET` lives inside each muscle-group section. The
  workout-level `+ SAVED WORKOUT` button sits at the top of the
  Builder tab, **distinct from `▶ START NOW`** (Save = load template
  into Builder for amending; Start = use the current build as a live
  workout). Tapping `+ SAVED WORKOUT` and selecting an entry
  **overwrites the current build** entirely (matches today's `▶ Use`
  workout clobber semantics). User can then amend (sets/reps/add
  more exercises/supersets) before starting.
- **Auto-name (revises WE-63 superset rule):**
  - **Workouts** — `Custom [GROUP] Workout #N`. `[GROUP]` = sections
    that actually have exercises, **Title Case**, `/`-joined.
    Already shipped post-`we-v1.2` case fix (`3d9b2ca`); no change
    needed in Rev 11.
  - **Supersets** — `Custom [GROUP] Superset #N`. `[GROUP]` derives
    from **exercises within the superset** (first exercise's primary
    mg, then `/` + disparate mgs in add-order), Title Case. **This
    is a behaviour change to shipped Rev 10 step 6 code**, which
    currently uses the section's mg. Rev 11 switches to exercise-
    derived. Single-mg superset → `Custom Back Superset #N`; mixed
    → `Custom Back/Biceps Superset #N`.

### Initial Builder layout

Top of Builder tab → down:

1. **Chip grid + pair blurb** (WE-30, unchanged). `cardio` chip is
   default-selected.
2. **`+ SAVED WORKOUT`** — workout-level button. Overwrites build on
   selection.
3. **`▶ START NOW`** — uses the current build live.
4. **Build area** — sections appear per chip selection. **Cardio
   section pinned top** when cardio chip is selected.
5. **Per section:** standard exercises + supersets + section-level
   `+ ADD EXERCISE` and `+ SAVED SUPERSET` buttons.

Section headers + workout name derive from chip selection + content,
as today.

### Behaviors

- **`+ ADD EXERCISE` flow** — opens picker modal. Picker surfaces
  recommendations from history alongside the full exercise list
  (recs are no longer a separate Builder area). On exercise pick:
  - If the section has no supersets yet: exercise lands as a new
    singleton.
  - If the section already has supersets: a follow-up "where does
    this go?" modal lists existing supersets (concise — just the
    exercise names, with section header) plus a `+ NEW SUPERSET`
    option. Picking an existing one appends the exercise into that
    superset; `+ NEW SUPERSET` lands it as a new singleton.
- **Swipe-to-delete** — universal remove mechanism for any exercise
  row (supersedes WE-57 two-tap `⊖`). Two-stage: first swipe reveals
  a DELETE affordance; second swipe (or tap on the affordance)
  commits. Outside-tap collapses, matching today's mutual-exclusion
  model. Swipe-to-delete also works on the cardio section block.
- **Per-exercise action cluster — three-state model.** Controls
  available depend on the exercise's context; supersedes WE-58
  cluster entirely.

  | State | Controls |
  |---|---|
  | Singleton in mg, no other exercises in same mg | All greyed (only swipe-to-delete works) |
  | Singleton in mg, other exercises exist in same mg | **Merge icon** active → chooser modal (existing supersets in this mg + `+ NEW SUPERSET`). All other controls greyed |
  | One of 2+ exercises in a superset | `↑` (grey at top) / `↓` (grey at bottom) / **breakout arrow** (split out into own singleton) |

  No `⤵` merge-down icon — merging is exclusively driven by the
  singleton-with-peers merge icon → modal flow above. The Rev 10
  `⤴` / `⤵` cluster goes away entirely. `↑` / `↓` stay as button
  arrows (NOT swipe).
- **Save-as-superset selector** — only renders on 2+ exercise
  supersets (singletons get nothing). **Positioned at the top** of
  the superset card. Resolves the Phase 1B known issue about
  misleading singleton save buttons (struck below).

### Rules-doc impact (v1.4 bump)

- **Supersedes WE-57** (two-tap remove) — replaced by swipe-to-delete.
- **Supersedes WE-58** (action cluster `↑ ↓ ⤴ ⤵ ⊖`) — `⊖` → swipe;
  `⤴` / `⤵` → three-state cluster: singleton+peers shows merge-icon
  → modal, in-superset shows `↑` / `↓` / breakout, lone singleton
  greyed. `↑` / `↓` stay as button arrows.
- **Supersedes WE-15 + WE-55 in part** — recommendations leave the
  Builder layout; new home is inside the `+ ADD EXERCISE` picker
  modal. The Builder Build / Recs split structurally goes away;
  Build area + sections + per-section action buttons remain.
- **Supersedes WE-61 in part** — save-as-superset selector renders
  only on 2+ ex supersets, top-positioned (was bottom).
- **Modifies WE-63 superset rule** — `[GROUP]` derivation for saved
  supersets changes from section-based to exercise-based (first
  exercise's primary + `/` + disparate mgs in add-order). Workout
  rule unchanged. **Code change required at Rev 11 time** (replaces
  the `autoNameGroupKey([mg])` call in the Save Superset handler
  with the new exercise-walking derivation).
- **Untouched and still valid:** WE-30 (chip grid + pair emphasis),
  WE-31 (Add Exercise picker — gains rec integration in Rev 11 but
  the accordion-rows-with-history pattern survives), WE-59 (primary-
  group tag), WE-60 (Save Workout), WE-62 (Saved Library screen),
  WE-63 workout rule, plus all of Parts 1–9 outside the above.
- **New rules needed (WE-64+):**
  - Builder top-level layout with workout-level `+ SAVED WORKOUT` +
    `▶ START NOW` placement
  - Cardio default-selected + pinned-top behaviour
  - `+ ADD EXERCISE` flow with rec integration + "where does this
    go?" follow-up
  - Three-state action cluster spec
  - Merge-icon modal flow spec
  - Swipe-to-delete behaviour spec
  - `+ SAVED WORKOUT` overwrite-current-build semantics
  - iPhone long-press drag-to-group — Phase 7 / PWA-era deferred
    rule, spec'd here for forward reference

### Deferred to Phase 7 (PWA-era touch polish)

- **iPhone long-press drag-to-group** — touch-native version of the
  merge flow, modeled on iOS home-screen folder creation. Per user
  note: "this last part may be added to outstanding items while
  still working on ui/ux pre-iPhone." Phase 7 territory; track here
  so the Rev 11 merge-icon modal is built in a way that doesn't
  make the long-press flow harder to bolt on later.

## Palette

**SHIPPED 2026-06-07** — see CHANGELOG entry "Palette refresh — teal/dark". Spec retained below as v2-fine-tune reference; sub-bullets on flat tokens (`success`/`accent`, `surface`/`surf_el`) and the non-literal text mapping noted in the CHANGELOG remain open for a future palette pass.

**Palette refresh — "Teal/dark" (approved 2026-06-07)**

Tokens to apply across `index.html` when palette refresh ships:

```
bg:        0A0F0E  (use dark version — current BFF3E7 is inverted, darkest readable bg in this family)
surface:   1C2725
surf_el:   1C2725
txt_p:     1F8A7E
txt_s:     216F66
accent:    1F8A7E
data:      1F8A7E
success:   1F8A7E
destruct:  B14646
chart_lo:  B1E2DC
chart_hi:  76DACE
```

Notes:
- Several tokens share the same value (`surf_el = surface`, `txt_p = accent = data = success`). May want to differentiate surface/surf_el and success/accent before shipping — or keep flat and revisit in v2.
- `bg` token as captured (`BFF3E7`) appears to be a light version; replace with a dark teal bg when implementing. Suggest `0A0F0E` or `0D1A18`.
- Commit when ready as: `WE: palette refresh — teal/dark`
- Phone-test gate applies before push per CLAUDE.md

## Carryover from 2026-05-15 deploy session

1. ~~**Tag convention question (WE-42 vs WE-46).**~~ Resolved 2026-05-15: `we-v1.1` tagged retroactively on `bc095cc` (Phase 1 deploy point); Rev 10 will ship as `we-v1.2`. Preserves the WE-46 1:1 changelog ↔ pipeline mapping.
2. **`gh` CLI PATH gotcha (deploy-session machine only).** Installed at `C:\Program Files\GitHub CLI\gh.exe`. Not on Bash PATH in the deploy-session shell; PowerShell needed `& "..."` call operator. Fresh shell may pick up the system PATH update — test first. Auth persists in OS keyring; no re-login expected. Surface Pro 7 (current build env) presumed clean until proven otherwise.
3. **Section 1 above is a working checklist, not archival.** As Rev 10 deltas (WE-57–WE-63) ship, strike them rather than check them in place. When all 8 ship, the section collapses to one line or disappears.
4. **WE-N citation discipline.** The handoff that mis-cited WE-29 for the 15-iteration rule (actually in the unnumbered Versioning Protocol preamble) is logged. Folding into v1.4 spec hygiene; tracked below — no further action now.

## Parked from 2026-05-16 Rev 10 step 2 sign-off

Items parked at step 2 sign-off, each with a trigger for when to revisit.
Per CLAUDE.md §7 (d) park disposition.

1. **`↑`/`↓` reorder — verify with 3+ exercise supersets.** Step 2 ships
   against 1–2 ex supersets only (current demo data). **Trigger:**
   revisit when Phase 2 add-exercise picker is wired and 3+ ex
   supersets are constructable.

2. **`⤴` split position — currently inserts singleton DIRECTLY BELOW
   source.** Plan didn't fix direction; "below" pairs symmetrically with
   `⤵` ("both ops act on the boundary below this superset"). Alternative
   was "above" (closer to the literal ↑-arrow direction). **Trigger:**
   revisit with 3+ ex supersets in the UI to observe whether "below"
   feels right when splitting first/middle/last exercises.

3. **`⤴` split origin — currently `'normal'` regardless of source.**
   Rationale: splits don't involve `+ NEW SUPERSET`, so the singleton
   isn't a reserved placeholder. Alternative was inherit-from-source
   (matters only when splitting out of a merged `origin:'new'` superset).
   **Trigger:** revisit when the merge-into-`'new'` path is exercisable
   end-to-end and a split out of that state can be tested.

4. **`⤵` merge anchor — result-in-source-slot, target removed.** User
   anchored on source (tapped `⤵` there); source slot survives to
   minimize apparent movement. Alternative was result-in-target-slot
   (merged group appears to "move down"). **Trigger:** revisit with
   multi-superset merge testable end-to-end in the UI.

5. **`⤵` merge-into-`'new'` origin preservation — needs end-to-end
   verification.** Implemented per the HANDOFF.md locked-in decision and
   the WE-15 / WE-57 auto-delete exception is in place from step 1, but
   the full lifecycle ((a) merge content into `+ NEW SUPERSET`
   placeholder, (b) later remove all exercises, (c) confirm placeholder
   card persists) hasn't been behaviorally tested. **Trigger:** test
   when picker enables build-up + tear-down of a merged `'new'` superset.

## Phase 1B known issues — phone testing

### ~~`+ SAVE SUPERSET` button on singleton supersets reads as misleading (2026-06-07)~~ — resolution locked-in via Rev 11 (selector renders only on 2+ ex supersets, top-positioned). Original entry retained below for context.

Reported during 2026-06-07 phone test of Rev 10 step 5:

> "Once I pull in more than 1 exercise, each individual exercise has its
> own `+ SAVE SUPERSET` button. Need better UI to show which of the
> individual exercises are being lumped together to create a larger
> superset."

Current behavior: WE-61 renders `+ SAVE SUPERSET` at the bottom of every
non-empty, non-cardio entry in `buildState[mg]`, regardless of exercise
count. A 1-exercise singleton (`{ exs: [ex], origin: 'normal' }`) gets
the same save button as a true multi-exercise superset. The buildState
shape is technically correct ("a singleton IS a 1-element superset"),
but visually it implies that each row is a save-as-superset candidate,
and offers no visual grouping cue that distinguishes the singletons from
real multi-ex supersets.

**Two intertwined sub-concerns:**
1. **Affordance:** showing `+ SAVE SUPERSET` on a singleton is
   confusing — it suggests there's something supersetty to save when
   there isn't. Options:
   - Suppress the button on singletons; only render when `ss.exs.length >= 2`
   - Rename to `+ SAVE GROUP` so it doesn't claim superset semantics on a singleton
   - Keep as-is and accept that saved singletons are valid (a saved
     "1-ex superset" is mechanically a saved exercise template, which
     might actually be useful for later reuse via `📋 INSERT SAVED
     SUPERSET`)
2. **Visual grouping:** even when there ARE real multi-ex supersets,
   the boundary between supersets isn't loud enough on the phone. The
   existing `.superset` div wraps multi-ex groups but the dashed
   border isn't visually distinct from the inter-row dashed separator
   inside a single superset. Need a stronger visual treatment (heavier
   border, background tint, "SUPERSET" label, numbered group, etc.).

**Trigger:** revisit when Phase 2 picker lands and real multi-ex
supersets become exercisable end-to-end. UX call may depend on what the
test data looks like once the demo library expands.

### ⊖ multi-remove visual confusion with duplicate-name exercises (2026-06-07)

Reported during 2026-06-07 phone test of Rev 10 step 5:

> "When I remove more than one exercise, the notification says the correct
> next exercise is being removed, but the tile for the removed exercise
> doesn't actually go away."

Almost certainly a **data-confound, not a code bug**: `shadowData` has
`Dumbbell Curls` in *both* `back` recs (off-section, `primary: 'biceps'`)
*and* `biceps` recs. With the default `Back + Biceps` selection, pulling
both gives two `Dumbbell Curls` tiles in the build. Removing one leaves
the other visible — toast names the just-removed instance, but the
remaining instance has the same name, so it reads as "the tile didn't go
away." User confirmed during phone test that the toast text matched the
visible remaining tile, which fits this hypothesis.

**Static read of the code path** (`removeExerciseFromBuild` → splice →
`updateSectionsArea` full re-render): no path where toast name + splice
target diverge, and re-render rebuilds from `buildState` so a removed
exercise can't ghost. Indices baked into button HTML are regenerated on
every render.

**Triggers to resolve:**
1. Phase 2 brings a real exercise library + picker. Once user pulls
   *specific* instances deliberately (vs. shadowData's two-Dumbbell-Curls
   demo data), the confound disappears organically.
2. If the same symptom reproduces against *non-duplicate* names after
   Phase 2 lands, escalate as a real render bug and instrument
   `removeExerciseFromBuild` with a debug toast / console line.
3. Consider whether to differentiate same-name instances visually (e.g.,
   section badge already exists via WE-59 — for off-section instances
   that's enough; for same-section duplicates we'd need a counter or
   coords).

## Parked from 2026-05-16 Rev 10 step 3 sign-off

1. **WE-59 tag position — should sit immediately after the exercise name
   text, not at the right edge of the name's flex container.** Current
   implementation places the tag after `.build-ex-name` / `.rec-name`,
   but the `flex: 1` on those name spans expands them to fill available
   space, pushing the tag to the right edge — adjacent to the action
   cluster (Build) or `sets×reps` meta (Rec) instead of tight to the
   name text. **Fix sketch:** wrap `[name + tag]` in a `flex: 1` parent
   with the tag at `flex-shrink: 0` and the name itself shrinking with
   ellipsis. **Trigger:** address when Builder visual polish lands, or
   fold into the WE-59 expansion in Phase 5/6 (Workout / History /
   Exercise Detail), or sooner if it grates during ongoing phone tests.

## Open nuances (not deeply resolved)

### A. Reuse of saved workout templates
When user pulls a saved workout into the Builder (Phase 6B), how do weights/reps populate?
- Per WE-60, saved templates are structure-only (no weights stored)
- Per WE-10, weights surface from history via the recommendation engine
- **Expected behavior:** loaded template auto-shadow-populates from each exercise's most recent history per WE-11
- **Open question:** what if an exercise in the template has no history (first time)? Empty shadows? Prompt user?

### B. In-progress plan resume + saved templates
If user starts a workout from a saved template (creating an in-progress plan), then later edits the template, does the in-progress plan reflect changes?
- **Likely answer:** no. Templates and plans are decoupled at "Use" time (deep copy). Confirm during Phase 6B build.

### C. Cardio in Workout Mode
Cardio block UI for logging not fully mocked through Rev 9. Deferred to Phase 5 build:
- How does cardio "log" interact with the sticky bottom log bar? (Currently the log bar is weight-set-oriented)
- Suggestion: cardio gets its own top card with inline fields, not the bottom log bar

### D. Fuzzy focus area matching (deferred to v2 per WE-43)
- Today: pairing emphasis + recommendations require exact chip match
- v2 idea: select Back/Chest → matches "Back/Biceps + Chest/Triceps" historical workouts via fuzzy logic

## Spec hygiene

- **Add WE-N tag to the 15-iteration rule.** Currently lives in the unnumbered Versioning Protocol preamble of `Workout_Engine_Rules_v1.3.md`. **Trigger:** fold into the v1.4 rules-doc bump that ships after Rev 10.

## v2 candidates

v2 candidates live in WE-43 of the rules doc.

## Testing setup

- **Phase 1 (now):** GitHub Pages deployment + Safari "Add to Home Screen" on iPhone for visual/interaction verification (IndexedDB not yet wired)
- **Phases 3+:** Same deployment path, now with real persistence once Dexie is wired
- **Active dev sessions:** local edit on laptop, push to `main`, GH Pages rebuild in ~30-60s, iPhone reload
