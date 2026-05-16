# Outstanding Items — Workout Engine

As of 2026-05-15.

## Immediate next steps (priority order)

### 1. Build Rev 10 in code
Rev 10 spec is captured in WE-57 through WE-63 of `Workout_Engine_Rules_v1.3.md`. In progress on Phase 1B.

Starting point: current `index.html` (Phase 1 Rev 9). Apply these deltas:

- ~~**Two-tap remove** (WE-57)~~ — shipped in Rev 10 step 1.
- ~~**Move buttons** (WE-58): cluster scaffold + `⊖` shipped in step 1 (`acc51e6`); `↑ ↓ ⤴ ⤵` move logic shipped in step 2 (`35a0031`).~~ Visual/behavioral refinements parked — see "Parked from 2026-05-16 Rev 10 step 2 sign-off" section below.
- **Primary-group tag** (WE-59): grey italic mono text adjacent to exercise name when primary group ≠ section. Show in Builder, Workout, History, ExerciseDetail.
- **Save Workout button** in Builder action bar (WE-60). Structure only.
- **Save Superset button** at bottom of each superset card (WE-61). Structure only.
- **Saved Library screen** (WE-62): accessible from Home `📋 Saved templates` and Builder `📋 INSERT SAVED SUPERSET` buttons. WORKOUTS/SUPERSETS toggle. Cards with name, preview, dates, use count, `▶ Use` and `✕ Delete`.
- **Auto-name logic** (WE-63): `Custom [GROUP] [Type] #N`; multi-group workouts join with `/`; supersets use section context; "lowest available N" rule via `is_auto_named` flag.
- ~~**Side effects** (WE-15, WE-57, WE-58)~~ — shipped in step 1 (empty-superset auto-delete + pulled-rec re-enable).

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
