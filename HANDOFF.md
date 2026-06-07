# Workout Engine — Cross-Machine Handoff

Portable build-state pointer. Read this **before** running any commands or
editing code on a new machine. Update as part of every commit per WE-46.

---

## Last updated

- **Build-state commit:** `<pending>` — Rev 10 step 6 (WE-63 auto-name
  logic). Local only; push pending.
- **HANDOFF.md last edit:** 2026-06-07, ASUS session (step 6 ship —
  WE-63 auto-name `Custom [GROUP] [Type] #N` with lowest-available-N
  gap-fill via `isAutoNamed` flag).

---

## Read first

In this order:

1. `CLAUDE.md` — project instructions, working preferences, don'ts, workflow
   rules (§7).
2. `Workout_Engine_Rules_v1.3.md` — authoritative spec, 63 rules across 10
   Parts. Engine prefix: `WE`.
3. Latest `CHANGELOG.md` entry — current top entry is **Phase 1B Rev 10
   step 1** (2026-05-15).
4. `OUTSTANDING_ITEMS.md` — live build checklist; section 1 is the Rev 10
   working list (strike, don't check, as items ship).

---

## Verification commands

Run before touching code. Expected output noted inline.

```
git log --oneline -8
# expect HEAD = Rev 10 step 5 commit, preceded by 996a2ad (ASUS session
# wrap), fa4b870 (Rev 10 step 4), 07a2fe6 (step 3 sign-off), ec6bcb4
# (Rev 10 step 3), 6cb5f73 (step 2 sign-off), 35a0031 (Rev 10 step 2),
# ff497c7 (sign-off protocol codification).

git tag --list "we-v*"
# expect: we-v1.1   (annotated tag on bc095cc per WE-42)

git status
# expect: clean working tree on main
```

If any of the above is stale (HEAD behind the step 5 commit, tag missing,
uncommitted work present), **stop and reconcile before starting step 6.**

---

## Current state

- ✓ Phase 1 deployed via GitHub Pages (2026-05-15).
- ⏳ **Phase 1B Rev 10 in progress.** Steps 1–6 shipped. Step 1
  (`acc51e6`): WE-57 + WE-58 cluster scaffold. Step 2 (`35a0031`): WE-58
  within-section move logic. Step 3 (`ec6bcb4`): WE-59 primary-group tag
  (Builder only). Step 4 (`fa4b870`): WE-60 + WE-61 save buttons —
  in-memory `savedWorkouts` / `savedSupersets`; structure-only. Step 5
  (`f85215d`): WE-62 Saved Library screen with WORKOUTS/SUPERSETS toggle,
  card preview, ▶ Use, ✕ Delete; `useCount` added to schema. Palette
  refresh (`bee5c6a`, teal/dark) landed between steps 5 and 6, pure
  visual swap. Step 6 (this commit): WE-63 auto-name
  `Custom [GROUP] [Type] #N` with lowest-available-N gap-fill scoped per
  `[GROUP] [Type]` key; `isAutoNamed` schema field added so user-edited
  names don't poison the auto-name space. **2 of 8 Rev 10 steps remain
  (step 7 changelog/tag + step 8 push).**
- `we-v1.1` annotated tag landed retroactively on `bc095cc` (Phase 1 deploy
  point per WE-42). `we-v1.2` lands at step 7.

---

## Active Rev plan (Rev 10, 8 steps)

Locked in prior session. Do not reorder without explicit OK.

1. ✓ **WE-58 cluster + WE-57 two-tap remove + side effects** — DONE
   (`acc51e6`).
2. ✓ **WE-58 within-section move logic** (`↑ ↓ ⤴ ⤵`) — DONE (`35a0031`).
   Sign-off 2026-05-16: 5 items parked to OUTSTANDING_ITEMS.md
   ("Parked from 2026-05-16 Rev 10 step 2 sign-off" section), 3 kept
   as-is.
3. ✓ **WE-59 primary-group tag** — DONE (`ec6bcb4`). **Builder only**
   (Build area + Rec area). Workout / History / Exercise Detail
   expansion deferred to Phase 5/6 (in OUTSTANDING_ITEMS). Sign-off
   2026-05-16: 1 item parked (tag position — `flex: 1` on name spans
   pushes tag to the right edge instead of adjacent to name text), 7
   kept as-is. New concern from phone test folded into Phase 2 task
   list: `+ ADD EXERCISE` modal needs its Done action wired to
   `buildState`, plus a larger demo library to exercise WE-59 properly.
4. ✓ **WE-60 + WE-61 save buttons** (in-memory; no Dexie until Phase 3) —
   DONE (`fa4b870`). `+ SAVE WORKOUT` block button at bottom of Builder
   when build is non-empty; `+ SAVE SUPERSET` small inline button at the
   bottom of each non-empty, non-cardio superset. `prompt()` with
   auto-stub default name; cancel aborts; empty/whitespace falls back to
   default. Sign-off 2026-05-17: 9 items defaulted to `(a) keep all` per
   CLAUDE.md §7 (user dispositioned by silence; nothing parked).
5. ✓ **WE-62 Saved Library screen** (`f85215d`). New
   `screen-saved-library` with WORKOUTS/SUPERSETS toggle. Cards show
   name, date, ex count, use count, structure preview, `▶ Use` and
   `✕ Delete`. Sort by `savedAt` desc. Empty states per mode (and
   per mg when entered from Builder section). Entry points: Home
   `📋 SAVED TEMPLATES`, Builder per-section `📋 INSERT SAVED SUPERSET`
   (non-cardio only). `▶ Use` on a workout clobbers `buildState` and
   switches to Builder (matches `+ Pull in` replace semantics). `▶ Use`
   on a superset is enabled only when arrived from a Builder section;
   appends to that section. Both bump `useCount`. `✕ Delete` removes
   in-memory and toasts (no undo). Sign-off 2026-06-07: (a) all kept;
   one mid-test bug (`ex.sets.forEach` on saved-template Use) fixed in
   same commit per (b) revise via `hydrateInsertedEx`.
6. ✓ **WE-63 auto-name logic** — DONE (this commit). Save Workout +
   Save Superset prompts default to `Custom [GROUP] [Type] #N` where
   `[GROUP]` is uppercase mg display name(s) joined with `/`. Workouts
   derive `[GROUP]` from `sections.keys()` (only mgs with actual
   exercises, not raw `selectedMG` — selecting `Back+Biceps` but only
   adding back exercises yields `Custom BACK Workout #N`, not
   `Custom BACK/BICEPS Workout #N`). Supersets use the section's mg.
   `#N` = lowest available positive integer via
   `lowestAvailableAutoNameN`, scanning only entries with
   `isAutoNamed: true`. User-edited names get `isAutoNamed: false` and
   don't reserve slots — so gap-fill works (deleting an auto-named #2
   means the next auto-save picks #2, not #4). Counters renamed:
   `savedWorkoutSeq` / `savedSupersetSeq` → `nextSavedWorkoutId` /
   `nextSavedSupersetId` (id role only, no name semantics).
7. ⏭️ **NEXT — CHANGELOG + CLAUDE.md updates; tag `we-v1.2`.**
8. ☐ Push + verify GH Pages deploy.

---

## Next step

**Step 7 — CHANGELOG + CLAUDE.md updates; tag `we-v1.2`.**

Scope:

- `CHANGELOG.md` — step 6 entry already landed in this commit; step 7
  rolls up the Rev 10 summary entry under the cumulative tag heading,
  noting WE-57 through WE-63 + the palette refresh that landed between
  steps 5 and 6.
- `CLAUDE.md` — bump §5 "Build Phase Status" Phase 1B line from
  "Step 1 of 8 shipped" to "Rev 10 complete (we-v1.2)"; refresh
  Phase 2 entry if any framing changed.
- `git tag -a we-v1.2 -m "Rev 10 complete (WE-57 through WE-63)"`
  pointing at HEAD per WE-42.
- Phone-test gate before push at step 8 — sanity-check the tagged
  build via local serve OR after the step 8 push via GH Pages.

---

## Locked-in decisions (append-only)

Decisions that survive across sessions. Do not edit historical entries.
Append new ones; supersede with a dated follow-up entry if reversed.

### 2026-05-15 — `we-v1.1` tag placement (WE-42)

`we-v1.1` annotated tag lives on `bc095cc` (Phase 1 deploy point), not on
any later commit. Rev 10 will ship as `we-v1.2`. Preserves the WE-46 1:1
changelog ↔ pipeline mapping. Resolves the open tag-convention question
that was item 1 in `OUTSTANDING_ITEMS.md`.

### 2026-05-15 — `⤵` merge into `origin:'new'` placeholder keeps `origin:'new'`

When `mergeWithBelow` (step 2 logic) targets an empty superset created by
`+ NEW SUPERSET` (i.e., the target has `origin: 'new'`), the resulting
merged superset **retains `origin: 'new'`**.

**Why:** the `+ NEW SUPERSET` placeholder slot represents an explicit user
choice to reserve that superset position. Content arriving via `⤵` does
not erase that intent — if the user later removes all exercises from the
merged superset, the empty-superset auto-delete exception (WE-15 + WE-57
side effect) should still apply so the placeholder card persists.

**How to apply:** in `mergeWithBelow`, preserve the *target* superset's
`origin` field when copying exercises into it. Document this with a code
comment at the merge site referencing this HANDOFF.md entry.

---

## Workflow rules

Canonical text for the cross-session build discipline. Lives here (not in
`CLAUDE.md`) so it stays adjacent to the Active Rev plan and Locked-in
decisions it governs.

- **Commit after each Rev 10 step per WE-45/46.** Don't batch.
- **After each commit, pause and give a short status report** (files
  touched, line counts, syntax check, proposed scope for next step). Wait
  for "go" before starting the next step.
- **Run the sign-off protocol** at every status report — see `CLAUDE.md`
  §7. Each meaningful built item gets a (a) keep / (b) revise / (c)
  remove / (d) park disposition. The commit body lists items in the order
  they'll be prompted.
- **Stop at step 3 for phone-test review.**
- **Don't push to `main` without explicit OK.** Pushes go live on GH Pages
  within ~1 minute.
- **Update HANDOFF.md** ("Last updated", "Current state", "Next step", and
  the DONE markers in the Active Rev plan) as part of every commit that
  changes build state, per WE-46 discipline.

---

## Machine-specific gotchas

### Surface Pro 7 (deploy-session machine)

- `gh` CLI installed at `C:\Program Files\GitHub CLI\gh.exe`. Was **not**
  on Bash PATH in the deploy-session shell — PowerShell needed `& "..."`
  call operator. A fresh shell may pick up the updated system PATH; test
  first. Auth persists in OS keyring; no re-login expected.
- `python` on PATH is the Microsoft Store stub. Use Node tooling for local
  serving (see below) instead of `python -m http.server`.

### ASUS (current machine, captured 2026-05-17)

- Node + `npx` confirmed at `C:\Program Files\nodejs\`. Both on PATH.
  First-run `npx --yes serve` is slow (downloads `serve` from npm); plan
  for ~10-20s before the listener binds.
- **Wi-Fi: connect to `Apt 6F WiFi` (the main router), NOT
  `Apt 6F WiFi_EXT`.** The extender SSID does client isolation by
  default — phone-on-extender can't reach laptop-on-anything (verified
  2026-05-17). Switching the laptop's Wi-Fi to the main router SSID
  fixed phone connectivity immediately.
- Network profile is Public — that's fine. Windows Defender auto-
  prompted on first `npx serve` launch and Allow rules for Node
  (Inbound, Public, all ports, both TCP and UDP) are now in place
  pointing at `C:\Program Files\nodejs\node.exe`. No manual rule needed.
- LAN IPv4 last observed at `192.168.0.14` (2026-06-07; previously
  `192.168.0.12` on 2026-05-17). The router doesn't pin a reservation,
  so the laptop can land on a different `.1x` after a reboot or lease
  expiry. **Verify with `ipconfig` at the start of each session.**
- **Keep this IP current.** If `npx serve` (or `ipconfig`) shows a
  different IPv4 than the one logged just above, update this gotcha
  entry **as part of the current step's commit** — don't defer. Future
  sessions trust this as the phone-test URL; stale IPs waste debug
  time (cost us a phone-test cycle on 2026-06-07).
- iOS Safari on phone: hard-refresh by killing the tab and reopening to
  bust cached HTML if pull-to-refresh doesn't pick up the new build
  (especially after a GH Pages push — local serve picks up immediately).

### Local rendering (any machine)

```
npx --yes serve -l 8000 .
```

- Node + `npx` are on PATH on Surface; confirm on each new machine.
- `serve` binds to `0.0.0.0` — iPhone reaches it at
  `http://<wifi-ipv4>:8000/` once Windows Firewall allows Node on the
  Private network profile.
- Both machines must be on the same Wi-Fi network.
