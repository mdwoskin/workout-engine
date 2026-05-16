# Workout Engine — CHANGELOG

Chronological build log. Each rev = a deliverable in chat.

---

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
