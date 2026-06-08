# The Penalty Model

What sprite fetches cost, measured: a fixed 6-dot core, an alignment
overhead set by `(K + SCX) mod 8`, an exact composition rule for stacked
sprites, and two edge regimes — the prelude and the right edge.

```admonish abstract "At a glance"
- Every fetch core costs **exactly 6.000 dots**; the documented "6–11 dots
  per sprite" is the core plus 0–5 dots of BG-fetcher alignment.
- The maximum +11.000 plateau occurs at **(K + SCX) mod 8 = 0**.
- N sprites at one X: first sprite's penalty + **(N − 1) × 6.000 dots** —
  measured exact through N=10.
- SPRITEX < 8 hides the fetch in the startup prelude — same total cost,
  different shape.
- The K=167 right edge is **not special**: the single-sprite cost floors
  at the 6-dot core and the same composition rule applies per effective
  sprite.
```

## Penalty by SPRITEX

Single-sprite and four-sprite penalty by SPRITEX (= OAM X; displayed X + 8),
SCX=0 (dmg-sim measurement, gbmicrotest sprite ROMs; firing-scanline median
minus non-firing median):

| SPRITEX | Single sprite (dots) | Four sprites (dots) |
|---------|----------------------|---------------------|
| 0 | +11.000 (prelude-absorbed) | — |
| 8 | +11.000 | +45.030 |
| 9 | +10.000 | +40.937 |
| 10 | — | +36.843 |
| 11 | — | +32.749 |
| 12 | — | +27.646 |
| 13 | — | +24.562 |
| 14 | — | +27.895 (tile-boundary anomaly) |
| 15 | — | +38.643 (tile-boundary anomaly) |

The single-sprite cost decrements exactly 1 dot per +1 SPRITEX over [8, 9];
the four-sprite penalty decrements a clean ~4.094 dots per +1 SPRITEX over
the first three steps of [8, 13] (≈1 dot of alignment per sprite × 4
sprites). The SPRITEX=12 row (+27.646) carries a sub-dot wobble — its
non-firing baseline itself drifts ~1 dot by tile-boundary alignment, so the
11→12 and 12→13 steps read −5.103 / −3.084 rather than the clean −4.094. At
SPRITEX ∈ {14, 15} the sprites straddle a BG tile boundary and per-sprite
alignment overhead re-enters in a form the linear model does not capture
(dmg-sim measurement).

```admonish info "Measured: the plateau is a function of (K + SCX) mod 8"
The +11.000 value reproduces at (K, S) = (8, 0), (7, 1), and (167, 1) — all
with (K + S) mod 8 = 0 — with zero variance across 72 firing scanlines per
ROM (dmg-sim measurement).
```

## Stacked sprites at one X: the chain collapses to combinational

When N sprites share an OAM X byte, each runs its full fetch core but the
inter-fetch gap collapses to gate delays: WUTY↑ᵢ → VEKU → TAKA↓ᵢ → SOWO↑ →
TEKY↑ᵢ₊₁ with no clocked element on the path. SACU never resumes between
fetches, the BG counter stays at 5, LYRY stays high — which is exactly what
keeps the AND4 re-fire chain armed. Each completed fetch's per-slot
fetched-flag DFF (EBOJ/CEDY/FONO family, clocked by WUTY) clears that
slot's X latch, so the next unfetched same-X slot drives FEPO for the
re-fire; the chain ends when no unfetched same-X slot remains.

Measured cadence (zero variance, 7 firing scanlines per N, the gambatte
`10spritesPrLine` K=167 chain):

| Stage | Δ from TEKY↑ᵢ (dots) |
|-------|----------------------|
| SOBU↑ᵢ | +0.481 |
| TAKA↑ᵢ | +0.484 |
| WUTY↑ᵢ (counter=5) | +5.998 |
| TAKA↓ᵢ | +5.999 |
| TEKY↑ᵢ₊₁ (same dot) | +5.999 |

```admonish tip "Rule: stacked-sprite composition"
~~~
mode3_ext(N, K, S) = single_sprite_penalty(K, S) + (N − 1) × 6.000 dots
~~~
```

The rule's measured anchors:

| Configuration | Predicted | Measured |
|---------------|-----------|----------|
| N=1, K=0 (prelude-absorbed) | 11.000 | 11.000 |
| N=2, K=0 | 17.000 | 17.000 |
| N=3, K=0 | 23.000 | 23.000 |
| 5 × K=7 + 5 × K=167, SCX=1 | 70.000 | 70.000 |

The Mooneye `intr_2_mode0_timing_sprites` calibration table (extras of
2/4/5/7/8/10/11/13/14/16 M-cycles for N = 1..10) is M-cycle rounding of
this uniform 6-dots-per-sprite extension — there is no hardware
shared-fetch alternation, and the BG fetcher plays no per-sprite role
(it is frozen throughout the chain). Multi-group configurations compose
per X-group: when PX passes from group A to group B, FEPO drops, SACU
resumes, the BG fetcher advances normally, and group B re-pays its own
first-sprite alignment.

## The right edge: K = 167

At K = 167 the X match fires at the dot WODU would otherwise fire. The
cost is **not** right-edge-special: (167 + 0) mod 8 = 7 puts the single
sprite at the alignment model's 6-dot floor, and the stacked-composition
rule applies unchanged — `ext = 6.000 + (N − 1) × 6.000` per **effective**
sprite. Measured (dmg-sim measurement, gambatte `10spritesPrLine_10xposA7`
ROM — nine on-screen sprites — and N ∈ {1,2,3,4} patched variants, zero
variance, 7 firing lines each):

| Effective N | Mode 3 extension (dots) | Integer part |
|---|---|---|
| 1 | +5.978 | 6 = 1 × 6 |
| 2 | +11.975 (two ROM variants) | 12 = 2 × 6 |
| 3 | +17.978 | 18 = 3 × 6 |
| 9 | +53.977 | 54 = 9 × 6 |

The ~0.02-dot residual against exact N × 6 is the end-of-chain WODU rise
arriving through the FEPO↓/XENA arm rather than the PX-decode arm.
Per-sprite TEKY spacing in the long chain: 5.947 dots between #1 and #2,
then exactly 6.000 between each subsequent pair, ending in a final TEKY
glitch pulse too narrow for SOBU's capture as FEPO falls.

## Sprites at SPRITEX < 8: the fetch hides in the prelude

When the X match fires before the first SACU↑, the whole fetch lands inside
the AVAP→first-SACU prelude instead of between SACU pulses — same total
cost, different shape. Measured at SPRITEX=0 (zero variance, 23 firing
scanlines): the real TEKY fires at +11.255 dots from AVAP (the *second*
LYRY-high event — the first is consumed by the TAKA carry-over clear), the
fetch runs +11.766 → +17.399, the first SACU lands at +18.430 (vs 7.026
baseline), and total Mode 3 duration is identical to the SPRITEX=8 case.
