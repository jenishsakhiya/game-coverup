# COVERUP

A browser game about fencing off territory. You walk the border of a polygon, cut across the
open middle, and claim whatever you managed to enclose — while things that kill you bounce
around inside it.

In the Qix / Volfied lineage, if you know it. One file, no dependencies, no build step.

```
open index.html in a browser
```

That's the whole install. It also runs fine straight off `file://`.

---

## How it plays

The arena starts as a polygon with a claimed border and an unclaimed interior.

- **Hold** a direction to walk the border. Holding a key only ever steers you along it — it
  will never carry you off the edge into the open.
- To cut inward, **press again** where the amber dashes appear. Now you're carving, and you
  keep going on your own momentum until you turn or reach a border. Presses only steer.
- Reach any border again and the loop closes. Every unclaimed pocket with **no guard inside it**
  becomes yours.
- Your cut line is only deadly **while you're drawing it**. Standing on a border, guards can't
  touch you.
- Claim 80% of the interior to clear the level. Three lives.

### Controls

| | |
|---|---|
| Walk / steer | `←` `↑` `→` `↓` or `W` `A` `S` `D` |
| Start, continue, resume | `Space` or `Enter` |
| Pause | `P` |
| Mute | `M` |
| Touch | swipe to steer (acts as a held key), tap to stop or advance |

## What's trying to kill you

**Guards** ricochet through the unclaimed interior, DVD-logo style, reflecting off walls —
including the new edges you just made. They kill you by touching your cut line, or you while
you're drawing it. As you claim ground their box shrinks, so the game gets *harder* as you win.

**Hunters** are guards wearing a ring. They bounce like the rest until you commit to a cut,
then they bend toward the head of your line. Long cuts are a real gamble.

**Sparks** are the purple stars, and they're why you can't wait anywhere. They spawn on the
far side of the board and crawl the border network hunting you along it. Standing still isn't
safe, and neither is the spot you were planning to come home to.

On top of that, every guard leans on the throttle the longer a level runs, up to 1.7× its
starting speed. Dawdling costs you.

## Scoring

A cut is worth what you dared. Score scales with the **share of what's left** you took in one
sweep, not the raw cell count:

```
share = cells claimed / cells that were still open
mult  = 1 + floor(share × 14),  capped at 12
score = cells × 10 × mult
```

Nibbling the edge pays ×1. Taking a third of the remaining board in one sweep pays ×5. Cutting
the board in half pays ×8, for the same cells you could have taken in twenty safe bites. The
capture chime rises with the multiplier, so you can hear how good the cut was.

Clearing a level adds `1500 × lives remaining`, plus a speed bonus that decays over the first
100 seconds. Best score is kept in `localStorage` under `coverup.best`.

## Level ramp

| Level | Guards | Hunting | Sparks | Guard speed | First spark at |
|---|---|---|---|---|---|
| 1 | 1 | – | 1 | 9.5 | 7.8s |
| 2 | 1 | – | 1 | 10.3 | 6.6s |
| 3 | 2 | – | 1 | 11.1 | 5.4s |
| 4 | 2 | 1 | 1 | 11.9 | 4.2s |
| 5 | 3 | 1 | 2 | 12.7 | 3.0s |
| 7 | 4 | 1 | 2 | 14.3 | 2.5s |
| 8+ | 4 | 2 | 2 | 15.1+ | 2.5s |

Speeds are in cells per second. You move at 20.

---

## How it works

Everything lives in [`index.html`](index.html) — markup, styles, and about 830 lines of plain
JavaScript against a 2D canvas. No framework, no modules, no toolchain.

### The grid

A 52 × 34 grid of cells, each `OUT`, `SAFE` (claimed), `OPEN` (unclaimed), or `TRAIL`
(the line you're currently drawing). Grid-snapped movement makes captures and collision exact
rather than approximate — no polygon clipping, no floating-point edge cases.

### Roads: what you're allowed to walk

The one rule that needed real care. You walk borders, never the inside of claimed land — but
"border" can't mean *currently touches open space*, because a border stops qualifying once both
of its sides get claimed, and the walkable network then falls into disconnected pieces. Split
the open area into two pockets and you'd be stranded in one of them forever.

So borders are **permanent**. A separate `road` mask marks the arena's own edge plus every line
you have ever cut, and it only grows. This gives connectivity by construction rather than by
patching: every cut begins and ends on existing road and is itself a connected path, so the
network can only ever grow as one piece. The line that splits two pockets *is* a road between
them. An abandoned cut is erased along with its road, since an unfinished line was never a
border — and a dangling spur can't disconnect anything when it goes.

The side effect is nice: old cut lines survive as shortcuts, so you're building a road network
as you play, and *where* you cut matters beyond the area it claims. Sparks use those roads too.

### Capture

Closing a loop turns the trail to `SAFE`, then flood-fills every remaining `OPEN` region and
claims the ones no guard is standing in. Guard containment is tested against the guard's whole
bounding box, not just its centre, which is what guarantees a guard is never sealed inside solid
ground.

### Guards

Continuous position, axis-aligned reflection off blocky walls. Motion is integrated in
sub-steps of at most 0.2 cells, so a fast guard can't tunnel through a one-cell wall. After a
bounce a hunter coasts for 0.4s before re-aiming — otherwise one aiming at a trail behind a
wall would just buzz against it.

### Rendering

The board is pre-rendered to an offscreen canvas and only redrawn when the grid actually
changes, on capture or death. Per frame it's a blit plus the trail, guards, sparks, player, and
popups. Backing store is scaled by `devicePixelRatio`; the canvas is CSS-scaled to fit, so it
works down to phone width.

## Other arena shapes

The arena is defined as a polygon in normalised 0–1 space and rasterised to cells, so other
shapes are a one-liner in `arenaFor(level)` — return a hexagon's or a star's points instead of
`RECT`. Nothing else needs to change.

The rasteriser already repairs the diagonal-only elbows that angled edges produce. Without that,
a 45° border comes out as a staircase of cells connected only at their corners, which four-way
movement cannot walk.

## License

Not chosen yet — add a `LICENSE` file and update this line before making the repo public.
