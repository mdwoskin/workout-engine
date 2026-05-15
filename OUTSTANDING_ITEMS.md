# Outstanding Items — Workout Engine

As of 2026-05-15.

## Immediate next steps (priority order)

### 1. Build Rev 10 in code
Rev 10 spec is captured in WE-57 through WE-63 of `Workout_Engine_Rules_v1.3.md`. Code NOT yet written.

Starting point: current `index.html` (Phase 1 Rev 9). Apply these deltas:

- **Two-tap remove** (WE-57): `⊖` red circle, expands horizontally to `REMOVE` on first tap, removes on second. No timeout. Outside-tap collapses. Mutual exclusion across rows.
- **Move buttons** (WE-58): `↑ ↓ ⤴ ⤵` plus the `⊖` = 5-icon cluster per exercise row. All always-visible.
- **Primary-group tag** (WE-59): grey italic mono text adjacent to exercise name when primary group ≠ section. Show in Builder, Workout, History, ExerciseDetail.
- **Save Workout button** in Builder action bar (WE-60). Structure only.
- **Save Superset button** at bottom of each superset card (WE-61). Structure only.
- **Saved Library screen** (WE-62): accessible from Home `📋 Saved templates` and Builder `📋 INSERT SAVED SUPERSET` buttons. WORKOUTS/SUPERSETS toggle. Cards with name, preview, dates, use count, `▶ Use` and `✕ Delete`.
- **Auto-name logic** (WE-63): `Custom [GROUP] [Type] #N`; multi-group workouts join with `/`; supersets use section context; "lowest available N" rule via `is_auto_named` flag.
- **Side effects** (WE-15, WE-57, WE-58): empty supersets auto-delete (except `+ NEW SUPERSET` placeholders); pulled rec re-enables when its exercise is removed from Build.

### 2. Phase 2: Exercise library + filter wiring
- Hardcode library from cleaned Excel Control sheet
- Fix `Goblet Spuat` → `Goblet Squat`
- Wire picker filter (per WE-31): `This section only` vs `Show all` toggle
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
- Phase 6: Complete workout + History Detail + Exercise Detail (per WE-22, WE-32, WE-50)
- Phase 6B: Save system wiring (Saved Library, save buttons, insert saved superset picker)
- Phase 7: PWA wrapper + export (manifest, service worker, JSON export per WE-39)

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
