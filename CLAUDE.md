# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single self-contained `index.html` (HTML + inline CSS + inline JS, no build step, no
dependencies, no tests) that live-predicts the two Red Car Trolley trains on the
ImagineFun Minecraft server. `README.md` is the only other file. Served by GitHub Pages
from `main` (root) at https://use-ai-for-mc.github.io/rct/.

The data/model originate from the [imagine-more-fun](https://github.com/use-ai-for-mc/imagine-more-fun)
mod, which logs ride captures and GotG departure times (`/imf rct calib status`; raw log
at `config/imaginemorefun/rct-calibration.ndjson`).

## Workflow

- **Preview locally:** open `index.html` directly, or `python3 -m http.server` then visit
  `localhost:8000`. There is nothing to build, lint, or test.
- **Deploy:** commit and push to `main`; GitHub Pages redeploys in a minute or two.
  Verify against the live URL.

## The prediction model

The trolleys run a deterministic, wall-clock-locked cycle of period `C` (≈680 s). Two
**identical** trains run exactly half a cycle (`C/2`) apart. A car's position is its
**phase** `= (now − epoch) mod C`. Within one cycle, three landmark constants partition
the phase into segments (`→ Buena Vista`, `at BV`, `→ GotG`, `at GotG`):
`BV_ARRIVE`, `BV_DEPART`, `GOTG_ARRIVE`. `car(phase, C)` maps a phase to a car's
state/detail/track-fraction; `tick()` (every 250 ms) is a pure function of `Date.now()`
and the calibration `cal`, redrawing both car dots, both station ETAs, and the banner.

**The half-cycle / swap ambiguity is the key subtlety.** An anchor (a "Good car left
GotG" sighting) pins the phase only *mod C/2*, because the two trains are identical — so
which physical car is the "Good" one (car A) is genuinely unobservable from timing alone.
That choice is the user-toggled `swap` flag (localStorage `rct_swap`), applied as a `C/2`
offset in `tick()`. Calibration drift logic must never try to "derive" which car is which.

## Calibration engine (`recalc()`)

`anchors` is a list of Unix-second timestamps of "Good car left GotG" taps
(localStorage `rct_anchors`; old single-anchor keys `rct_anchored_at`/`rct_epoch0` are
migrated). `recalc()`:

1. Assigns each tap an integer cycle index, least-squares fits time-vs-index to get the
   period, **rejects taps off by > C/4** (these are taps of the *other* car), refits.
2. Blends the fitted period with the seed (`SEED_C` ± `SEED_SIGMA`) by inverse variance.
3. Locks `epoch` (phase reference) to the **most recent** tap — a fresh live sighting.
   The period `C` only sets the *drift rate* between sightings, never the phase.

`overrideC` (localStorage `rct_period`) forces the period and skips the fit. Taps older
than `PRUNE_SEC` (48 h) are dropped, capped at `MAX_ANCHORS`.

## Recalibrating the baked defaults (maintainer)

Defaults live at the top of the `<script>` block (around line 96). When the page is
off-sync and you're given fresh in-game numbers:

- **`SEED_C`** (period, s) is the value worth baking — it's stable across the day and is
  what causes long-term drift. Set it to the measured even-second cycle.
- **`SEED_SIGMA`** is how much to trust `SEED_C` vs live taps (smaller = trust more);
  tighten toward `0.05` only when confident.
- **`EPOCH0_DEFAULT`** is only the rough pre-anchor phase. Optionally refresh it to
  `anchor mod SEED_C`. Do **not** try to bake phase for accuracy: a server restart resets
  the cycle, so real accuracy always comes from a live in-browser anchor, not the source.

See `README.md` for the full maintainer measure → edit → push procedure.
