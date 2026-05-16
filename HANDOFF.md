# Workout Engine — Cross-Machine Handoff

Portable build-state pointer. Read this **before** running any commands or
editing code on a new machine. Update as part of every commit per WE-46.

---

## Last updated

- **Build-state commit:** `acc51e6` — WE-57 two-tap remove + WE-58 action
  cluster scaffold. Unchanged since.
- **HANDOFF.md last edit:** 2026-05-16, ASUS session (sign-off protocol
  codified in `CLAUDE.md` §7).

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
git log --oneline -3
# expect HEAD = acc51e6 (Rev 10 step 1); previous: 59aa97b, bc095cc

git tag --list "we-v*"
# expect: we-v1.1   (annotated tag on bc095cc per WE-42)

git status
# expect: clean working tree on main, up to date with origin/main
```

If any of the above is stale (HEAD ahead of `acc51e6`, tag missing,
uncommitted work present), **stop and reconcile before starting step 2.**

---

## Current state

- ✓ Phase 1 deployed via GitHub Pages (2026-05-15).
- ⏳ **Phase 1B Rev 10 in progress.** Step 1 shipped in `acc51e6`
  (WE-57 two-tap remove + WE-58 cluster scaffold; render-checked in browser,
  approved as-is). **7 of 8 Rev 10 steps remain.**
- `we-v1.1` annotated tag landed retroactively on `bc095cc` (Phase 1 deploy
  point per WE-42).
- No code pushed to `origin` since `acc51e6`. Local-only state on each
  machine is the HANDOFF.md scaffold commit (this one).

---

## Active Rev plan (Rev 10, 8 steps)

Locked in prior session. Do not reorder without explicit OK.

1. ✓ **WE-58 cluster + WE-57 two-tap remove + side effects** — DONE
   (`acc51e6`).
2. ⏭️ **NEXT — WE-58 within-section move logic** (`↑ ↓ ⤴ ⤵`).
3. ☐ WE-59 primary-group tag (off-section, grey italic mono).
   **STOP HERE for phone-test review gate.**
4. ☐ WE-60 + WE-61 save buttons (in-memory; no Dexie until Phase 3).
5. ☐ WE-62 Saved Library screen.
6. ☐ WE-63 auto-name logic.
7. ☐ CHANGELOG + CLAUDE.md updates; tag `we-v1.2`.
8. ☐ Push + verify GH Pages deploy.

---

## Next step

**Step 2 — WE-58 within-section move logic.**

Implementation plan (settled in prior session):

- Replace `data-todo` placeholders on `↑ ↓ ⤴ ⤵` (currently at
  `index.html:790-793`) with `data-move=` attrs carrying superset and
  exercise coords.
- Add helpers: `moveUp`, `moveDown`, `splitExercise`, `mergeWithBelow`.
- Single dispatcher in `wireBuilderEvents` (currently at
  `index.html:895`).
- **No `buildState` shape changes needed for step 2.**
- See Locked-in decisions below for the `⤵` merge-into-`origin:'new'` edge
  case — document the rationale at the merge site with a code comment.

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

### ASUS (current machine)

- Status TBD on first use beyond `git`. Presumed clean until proven
  otherwise. **Before relying on any tool, run `where.exe <tool>` to
  confirm.**
- Add new entries here as quirks surface, dated.

### Local rendering (any machine)

```
npx --yes serve -l 8000 .
```

- Node + `npx` are on PATH on Surface; confirm on each new machine.
- `serve` binds to `0.0.0.0` — iPhone reaches it at
  `http://<wifi-ipv4>:8000/` once Windows Firewall allows Node on the
  Private network profile.
- Both machines must be on the same Wi-Fi network.
