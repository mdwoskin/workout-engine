# Workout Engine — Cross-Machine Handoff

Portable build-state pointer. Read this **before** running any commands or
editing code on a new machine. Update as part of every commit per WE-46.

---

## Last updated

- **Build-state commit:** `fa4b870` — Rev 10 step 4 (WE-60 + WE-61 save
  buttons, in-memory, structure-only). Pushed to `origin/main`.
- **HANDOFF.md last edit:** 2026-05-17, ASUS session (session wrap —
  step 4 hash backfilled; ASUS machine gotchas captured from
  2026-05-16/17 work; step 4 sign-off defaulted to `(a) keep all` per
  CLAUDE.md §7 since user dispositioned by silence).

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
# expect HEAD = Rev 10 step 4 commit, preceded by 07a2fe6 (step 3
# sign-off bookkeeping), ec6bcb4 (Rev 10 step 3), 6cb5f73 (step 2
# sign-off bookkeeping), 35a0031 (Rev 10 step 2), ff497c7 (sign-off
# protocol codification), 88c62d7 (HANDOFF.md introduction), acc51e6
# (Rev 10 step 1).

git tag --list "we-v*"
# expect: we-v1.1   (annotated tag on bc095cc per WE-42)

git status
# expect: clean working tree on main, up to date with origin/main
```

If any of the above is stale (HEAD behind the step 4 commit, tag missing,
uncommitted work present), **stop and reconcile before starting step 5.**

---

## Current state

- ✓ Phase 1 deployed via GitHub Pages (2026-05-15).
- ⏳ **Phase 1B Rev 10 in progress.** Steps 1–4 shipped. Step 1
  (`acc51e6`): WE-57 + WE-58 cluster scaffold. Step 2 (`35a0031`): WE-58
  within-section move logic. Step 3 (`ec6bcb4`): WE-59 primary-group tag
  (Builder only). Step 4 (this commit): WE-60 + WE-61 save buttons —
  in-memory `savedWorkouts` / `savedSupersets`; structure-only;
  `prompt()` for name with auto-stub default. **4 of 8 Rev 10 steps
  remain.**
- `we-v1.1` annotated tag landed retroactively on `bc095cc` (Phase 1 deploy
  point per WE-42).
- Local `main` is one commit ahead of `origin/main` until the step 4 push.

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
5. ⏭️ **NEXT — WE-62 Saved Library screen.**
6. ☐ WE-63 auto-name logic.
7. ☐ CHANGELOG + CLAUDE.md updates; tag `we-v1.2`.
8. ☐ Push + verify GH Pages deploy.

---

## Next step

**Step 5 — WE-62 Saved Library screen.**

Scope (per CHANGELOG v1.3 entry + OUTSTANDING_ITEMS):

- New screen reachable from Home `📋 Saved templates` and Builder
  `📋 INSERT SAVED SUPERSET` buttons (entry points need to be added to
  Home / Builder if not present yet).
- WORKOUTS / SUPERSETS toggle at the top — flips the list between
  `savedWorkouts` and `savedSupersets`.
- Cards per saved entry: name, structure preview (exercise names in
  order, grouped by section for workouts), savedAt date, use count
  (placeholder — increments when WE-62's `▶ Use` action lands), `▶ Use`
  and `✕ Delete` actions.
  - `▶ Use` on a saved workout: load its structure into `buildState`
    via the same data shapes used in `+ Pull in`. Switches to Builder.
  - `▶ Use` on a saved superset: only available from the Builder's
    `📋 INSERT SAVED SUPERSET` entry point; injects the superset into
    the active section.
  - `✕ Delete`: remove from the in-memory array; toast confirm.
- Sort by `savedAt` desc (most recent first), per the rules-doc note
  about `last_used_at` once that exists.
- Empty-state message when the relevant array is empty.
- Phase 1B: still in-memory; no Dexie. State clears on reload.

Phone-test review gate at step 5 — new screen lands; visual treatment
needs phone verification before continuing to step 6 (WE-63 auto-name
logic).

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
- LAN IPv4 has been stable at `192.168.0.12` across SSID swaps in this
  network. Verify with `ipconfig` before assuming.
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
