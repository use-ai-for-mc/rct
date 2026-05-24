# Red Car Trolley — live ETA

A single-page live predictor for the two Red Car Trolley trains on the ImagineFun
Minecraft server (Disney California Adventure: the DCA gate ⟷ Guardians of the Galaxy:
Mission BREAKOUT!).

**Live:** https://use-ai-for-mc.github.io/rct/

The trolleys run a deterministic, wall-clock-locked cycle (~680 s), and the two
*identical* trains run exactly half a cycle apart. The page computes each car's phase
from one **anchor** (a "GotG departure" time) plus the **cycle period**.

---

## Using it (in the browser, per visitor)

- It's accurate by default — no setup. If a car looks off, **re-sync**: tap the button the
  instant a trolley pulls out of GotG. Repeat over time and the page refines the period itself.
- Or paste mod data (see below): the **cycle** into the *advanced* period override, and
  the **anchor** Unix time into *Anchor from a timestamp*.
- The trains are identical, so a departure can't say which one is "Good." If the labels
  land on the wrong cars, hit **⇄ swap A/B** once.

This calibration lives only in *that browser* (localStorage). To improve the page for
**everyone** — and to make it right by default — recalibrate the source and push:

---

## Recalibrating the page (maintainer)

The defaults live near the top of `index.html`'s `<script>`:

| Constant | Meaning | Current |
|----------|---------|---------|
| `SEED_C` | cycle period, seconds | `679.71` |
| `SEED_SIGMA` | how much to trust `SEED_C` vs live taps (smaller = trust more) | `0.15` |
| `DEFAULT_ANCHOR` | baked GotG-departure (Unix s) the page predicts from by default | `1779649933.0` |

The page predicts from `DEFAULT_ANCHOR` + `SEED_C` and is **accurate out of the box** — the
in-browser calibration is an optional re-sync, not a required step. The **period** is stable
across the day; the **anchor** is what goes stale after a server restart (which resets the
cycle), so re-bake `DEFAULT_ANCHOR` when that happens.

### 1. Measure in-game (the mod)

- Make sure the **Red Car Trolley tracker** is enabled in the mod config.
- Ride the trolley **from GotG** 2–3 times, ideally spread over **30+ minutes** — the
  wider the span of departures, the sharper the period.
- Each departure prints to chat, or run **`/imf rct calib status`**. Read:
  - the green even-second **cycle** (e.g. `680s`), and
  - the **anchor** Unix time (`anchor=<unix>s`).
- The raw log is at `config/imaginemorefun/rct-calibration.ndjson` if you want to refit
  offline across sessions.

### 2. Edit `index.html`

- Set the anchor: `var DEFAULT_ANCHOR = 1779649933.0;` → a recent GotG-departure Unix time
  from the mod (just paste it — no modulo needed). This is what the page predicts from by
  default; refresh it after a server restart.
- Set the period: `var SEED_C = 679.71;` → your measured value.
- *(optional)* If you're confident, tighten `var SEED_SIGMA = 0.15;` (e.g. `0.05`) so the
  baked period dominates over a few stray taps.

### 3. Commit & push

```bash
git -C ~/if-local/rct commit -am "recalibrate: cycle 680.0s"
git -C ~/if-local/rct push
```

GitHub Pages redeploys in a minute or two. Verify by opening the live URL and checking a
car against what you see in-game.

---

## Hosting

Self-contained static `index.html`, served by GitHub Pages from `main` (root). The data
and model come from the [imagine-more-fun](https://github.com/use-ai-for-mc/imagine-more-fun)
mod, which logs the underlying ride captures and departure times.
