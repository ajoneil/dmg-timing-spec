# Startup and Fine Scroll

Every scanline's Mode 3 opens with the same deterministic cascade from the
scan-complete pulse (AVAP) to the first pixel clock — and fine scroll rides
on it, holding the pixel clock off for exactly `SCX & 7` extra dots. This
page covers the fine-scroll counter, the startup pipeline stage by stage,
and the measurements that pin both.

```admonish abstract "At a glance"
- AVAP↑ → first SACU↑ = **7.026 dots** at SCX=0, every stage at a fixed
  offset, zero variance across scanlines.
- Fine scroll is **clock suppression**: ROXY holds SACU high until the fine
  counter matches `SCX & 7` — exactly +1.000 dot per step. Mode 3 core duration
  is 173.045 + (SCX & 7) dots.
- The SACU count stays **167 at every SCX** — there is no push-and-discard.
- SCX is read **live**: whatever `ff43` holds at the count=N ROXO↑ capture
  edge decides the discard, measured to a 0.014-dot bracket.
```

## The fine scroll counter

A 3-bit ripple counter that discards `SCX & 7` pixels at Mode 3 startup by
delaying SACU's enable. The counter runs at CLKPIPE cadence (via ROXO =
NOT(SEGU)) while ROXY holds SACU high; a two-stage match pipeline (PUXA on
ROXO, NYZE on MOXE) captures the match, and POVA then clears ROXY.

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| PECU | Fine-counter gated clock | nand2 | Inputs: ROZE, ROXO | Frozen when ROZE=0 (counter at 7) |
| ROZE | Fine-counter self-stop | nand3 | Inputs: RUBU, ROGA, RYKU | 0 only at count 7 — MOCE's analogue |
| RYKU | Fine count bit 0 | dffr | PECU | Async reset by PASO |
| ROGA | Fine count bit 1 | dffr | RYKU (~Q ripple) | Async reset by PASO |
| RUBU | Fine count bit 2 | dffr | ROGA (~Q ripple) | Async reset by PASO |
| PASO | Fine-counter reset | nor2 | Inputs: PAHA, TEVO | Asserted outside Mode 3 or at tile boundaries |
| SUHA | Per-bit SCX match | xnor | `ff43_d0` vs RYKU | 1 when bit 0 matches |
| SYBY | Fine-count bit 1 match | xnor | `ff43_d1` vs ROGA | 1 when bit 1 matches |
| SOZU | Fine-count bit 2 match | xnor | `ff43_d2` vs RUBU | 1 when bit 2 matches |
| RONE | Drain-gated match aggregate | nand4 | Inputs: ROXY, SUHA, SYBY, SOZU | |
| POHU | Match comparator | not_x1 | Input: RONE | POHU = AND4(ROXY, SUHA, SYBY, SOZU) — self-terminating via the ROXY term |
| PUXA | Match capture (odd) | dffr | ROXO | Data: POHU |
| NYZE | Match capture (even) | dffr | MOXE | Data: PUXA |
| POVA | Match pulse | and2 | Inputs: NYZE.Q_n, PUXA | Single-dot pulse; drives ROXY's reset |
| ROXY | Fine-scroll gate | nor_latch | Set: PAHA; Reset: POVA | The SACU suppressor |

```admonish tip "Rule: PAHA polarity, stated once for the whole book"
PAHA = NOT(mode3) where `mode3` = XYMU.Q_n. The double inversion means PAHA
follows XYMU.Q — PAHA=1 *outside* Mode 3, 0 during it.
```

The match pipeline:

- Per-bit XNORs read `ff43` (SCX) **live** — there is no SCX snapshot at
  Mode 3 entry.
- POHU = AND4(ROXY, match bits): the ROXY term makes the comparator
  self-terminating — it collapses as soon as POVA clears ROXY.
- PUXA latches POHU on ROXO↑; NYZE mirrors PUXA on the next MOXE↑; POVA =
  AND2(NYZE.Q_n, PUXA) is high exactly between those two edges — a
  single-dot pulse, by construction unable to re-fire within the scanline
  (PUXA/NYZE reset to 0 outside Mode 3, guaranteeing NYZE.Q_n=1 at entry).

```admonish tip "Rule: the ff43 value at the count=N ROXO↑ wins"
The discard terminates on whatever `ff43` holds at the count=N ROXO↑
capture edge. A CPU SCX write reaches the comparison only if its CUPA
transparency window has made the value visible by that edge; a write
arriving even fractionally later is missed, and the pre-write SCX decides
the discard — measured to a 0.014-dot bracket
([late SCX writes](#late-scx-writes-during-the-discard)).
```

## The startup pipeline: AVAP to first SACU

Each scanline's Mode 3 begins with a 7-dot cascade from AVAP↑ to the first
SACU rising edge — 7.026 dots at SCX=0, with every stage at a deterministic
offset, zero variance across scanlines (dmg-sim measurement, purpose-built
`scx0_steady`
ROM, n=1,612):

| Stage | Δ from AVAP (dots) | Type | Mechanism |
|-------|--------------------|------|-----------|
| AVAP↑ | 0.000 | Combinational | OAM scan complete |
| XYMU↓ | 0.000 | Latch | AVAP drives XYMU.r directly |
| NYXU↓ | 0.002 | Combinational | NOR3 delay (439 ps) |
| LAXU/MESU/NYVA reset | 0.002 | Async reset | Fetch counter cleared |
| LAXU↑ | 0.986 | Clocked | First counter toggle after NYXU releases |
| NYKA↑ | 5.481 | DFF (ALET↑) | Fetch-complete captures after state 5 |
| PORY↑ | 5.991 | DFF (MYVO↑) | Half-dot later |
| SUVU↓, TAVE↑ | 5.996 | Combinational | First TEVO of the scanline fires |
| PYGO↑ | 6.480 | DFF (ALET↑) | |
| POKY↑ | 6.481 | NOR latch | Data-ready latches (+342 ps) |
| ROXY↓ | 6.490 | NOR latch | Cleared through the fine-scroll match chain (SCX=0: immediate match) |
| SACU↓ (glitch) | 6.511 | Combinational | ROXY opens the OR before the first proper pulse |
| **SACU↑** | **7.026** | — | **First pixel clock** |
| XEHO↑ | 7.026 | Clocked | PX 0→1, within the rounding of the same edge |
| RYFA↑ | 13.991 | DFF | Drain-detect chain reaches the first tile boundary |

The "AVAP at 0.000" row is the dot-count reference, not an ALET-edge
alignment — AVAP transitions on the ALET *falling* edge preceding the table
(it derives from BYBA's XUPY-clocked capture); the downstream DFFs are
ALET/MYVO-edge aligned. The cascade is identical on every scanline including
the first (whose 2-dot shortening lives entirely in Mode 0 — see
[Register writes](../registers/writes.md)). The window's MOSU trigger
feeds NYXU through the same NOR3 and produces the same 7-dot pipeline
([Window control](../window.md)). The NYXU pulse itself runs 0.492 dots
when AVAP drives it and 0.495 dots from TEVO (0.503 from MOSU).

## Fine-scroll startup suppression

```admonish tip "Rule: fine scroll is clock gating, not push-and-discard"
ROXY holds SACU high until the fine counter matches `SCX & 7`, POVA clears
ROXY, and the pipe starts shifting. No pixels enter the FIFO during the
gated window — the register is frozen, not loaded-and-discarded. The SACU
count stays 167 (a push-and-discard model would predict 167 + (SCX & 7)).
```

The cascade (NYKA → PORY → PYGO → POKY) fires at the same offsets regardless
of SCX; only ROXY's release moves. Measured at three SCX values (dmg-sim
measurements, purpose-built `scx{0,3,7}_steady` ROMs, n=1,151–1,612 scanlines each, zero variance):

| SCX | Mode 3 core, AVAP → WODU (dots) | SACU count | First SACU (dots from AVAP) |
|-----|--------------------------------|-----------|------------------------------|
| 0 | 173.045 | 167 | 7.026 |
| 3 | 176.045 | 167 | 10.026 |
| 7 | 180.045 | 167 | 14.026 |

The model is exact: 1.000 dot per `SCX & 7` step; first SACU at
7 + (N & 7) + 0.026; Mode 3 core duration 173 + (N & 7) + 0.045.

## Edge-by-edge startup waveform

The internal transitions, measured at SCX ∈ {0, 3, 7} (dmg-sim measurement,
purpose-built `scx{0,3,7}_steady` ROMs; AVAP↑ = 0.0000; PASO re-arms for good shortly
after ROXO's first rise):

Common cascade (SCX-independent):

| Δ dots | Signal | Mechanism |
|--------|--------|-----------|
| +0.0349 | PAHA↓ | NOT(mode3) — XYMU.Q_n rose on the AVAP reset just before |
| +0.0390 | PASO↑ | Fine-counter reset released |
| +5.4805 | NYKA↑ | Fetch state 5 complete |
| +5.9910 | PORY↑ | |
| +5.9963 | TEVO↑ | Startup tile pulse |
| +5.9973 | PASO↓ | Briefly re-asserted by TEVO |
| +6.4800 | PYGO↑ | |
| +6.4814 | POKY↑ | Data-ready |
| +6.4827 | SEGU↓ | First CLKPIPE enable |
| +6.4838 | TEVO↓ | |
| +6.4846 | ROXO↑ | Clocks PUXA and the fine counter |

Per-SCX match pipeline:

| Stage | SCX=0 | SCX=3 | SCX=7 |
|-------|-------|-------|-------|
| Counter ticks (one per ROXO↑) | 0 | 3 (+6.993 … +8.993) | 7 (+6.993 … +12.993) |
| POHU↑ | already 1 at reset | +8.9949 | +12.9949 |
| PUXA↑ | +6.4885 | +9.4864 | +13.4864 |
| POVA↑ | +6.4897 | +9.4876 | +13.4876 |
| ROXY↓ | +6.4902 | +9.4881 | +13.4881 |
| POHU↓ (self-termination) | +6.4906 | +9.4886 | +13.4886 |
| SACU↓ (glitch) | +6.5111 | +9.5090 | +13.5090 |
| NYZE↑ (NYZE.Q_n↓) | +6.9825 | +9.9825 | +13.9825 |
| POVA↓ | +6.9881 | +9.9881 | +13.9881 |
| **SACU↑** | **+7.0259** | **+10.0259** | **+14.0259** |

Each SCX increment shifts the block by exactly 1.0000 dots with identical
internal shape. The counter self-stop at SCX=7 does not suppress ROXO
(PUXA's clock path is independent of PECU), so the SCX=7 match fires with
the same geometry.

## The tail-of-Mode-3 drain

The drain detector fires on a fixed 8-dot cadence at +13.997, +21.997, …,
+165.997, **+173.997** dots from AVAP — independent of SCX. Mode 3 ends at
+(173.045 + N). The 21st drain edge at +173.997 therefore lands:

- **at SCX=0**: *after* Mode 3 ends (by 0.952 dots) — suppressed: the
  VOGA/WEGO transition and the mode3 reset on PYGO/RENE/RYFA clear the
  cascade first;
- **at SCX&7 ≥ 1**: *inside* Mode 3 — SEKO fires, TEVO → NYXU → a final
  parallel-load of the in-flight fetch's tile data.

The 21st load completes within the closing dot but produces no observable
LD output — the SACU count is already at 167, so the freshly loaded content
never meets another CP-gated edge before H-Blank. Both planes load from the
same tile index — there is no hybrid-fetch path here (dmg-sim measurement,
AGE `m3-bg-scx-nocgb` ROM: post-load shifter content 0b11111111 on both
planes where tilemap column 21 holds an all-$FF tile).

## Late SCX writes during the discard

The live-SCX rule, measured to its edge (dmg-sim measurement, gambatte
`ly0_late_scx7_m3stat_scx3_{1,2}` ROMs — SCX=3 held, late SCX=7 write during
the first-scanline discard):

| ROM | SCX=7 CUPA↑ | count=3 ROXO↑ | SCX seen at capture | Discard ends | First WODU↑ |
|-----|------------:|--------------:|:-------------------:|--------------|------------:|
| `_1` (hardware: mode 3) | +5.496 | +9.483 | **7** (write in time) | +13.488, count=7 | +180.045 (= 173+7) |
| `_2` (hardware: mode 0) | +9.496 | +9.483 | **3** (write 0.014 dots late) | +9.488, count=3 | +176.045 (= 173+3) |

In `_2` the write becomes visible 0.025 dots after the capture edge — and
the already-cleared ROXY cannot re-open (POHU collapsed at ROXY↓). The rule
is purely "the `ff43` value at the count=N ROXO↑ wins"; both hardware
outcomes follow from it.

## Startup invariance under stacked sprites

The whole cascade is configuration-invariant: with 5+5 sprites stacked at
two X positions chosen so `(X + SCX) mod 8 == 0` (the wilbertpol
`intr_2_mode0_timing_sprites_scx1_nops` worst case), every BG-fetch and cascade
transition between AVAP↑ and first SACU↑ lands at an identical sub-dot
offset across SCX ∈ {0..4}, and the sprite-chain edges (FEPO/TEKY/SOBU/
RYCE/TAKA/WUTY/SUDA) are likewise SCX-invariant (dmg-sim measurement). The
transitions nearest integer-dot boundaries sit 83–469 ps clear of them,
deterministically — real-silicon process variance cannot produce
per-configuration selectivity from a uniform per-die offset.
