# Red Car Trolley — live ETA

A single-page live predictor for the two Red Car Trolley trains on the ImagineFun
Minecraft server (Disney California Adventure: the DCA gate ⟷ Guardians of the Galaxy:
Mission BREAKOUT!).

**Live:** https://use-ai-for-mc.github.io/rct/

## How it works

The trolleys run a deterministic, wall-clock-locked cycle (~680 s), and the two
*identical* trains run exactly half a cycle apart. The page computes each car's phase
from a single anchor — a "GotG departure" time — plus the cycle period.

## Calibrating

- Tap **▶ the Good car just left GotG** the instant a trolley pulls out of GotG; that
  locks the phase. Repeat over time and the page refines the period itself (a longer
  span of taps → a sharper period → slower drift).
- Or feed it data from the in-game mod (`/imf rct calib`): paste the **cycle** into the
  *advanced* period override and the **anchor** Unix time into *Anchor from a timestamp*.
- The two trains are identical, so a departure timestamp can't say which one is "Good".
  If the labels land on the wrong cars, hit **⇄ swap A/B** once.

## Hosting

Self-contained static `index.html`, served by GitHub Pages from `main` (root).
The data and model come from the [imagine-more-fun](https://github.com/use-ai-for-mc/imagine-more-fun)
mod, which logs the underlying ride captures.
