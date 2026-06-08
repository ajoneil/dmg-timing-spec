# STAT Interrupts

The STAT interrupt (IF bit 1) is set by LALU, a DFF whose *clock* is the
combined STAT condition: SUKO, an AO2222 that OR-ANDs the four sources with
their enable bits. There is no periodic clock anywhere in this chain — LALU
fires when SUKO's output rises. Everything subtle about DMG STAT interrupts
("blocking", the write glitch, the boundary races) falls out of that one
fact plus per-leg gate depths.

```admonish abstract "At a glance"
- LALU (IF bit 1) is **clocked by the condition itself**: it fires only on
  a genuine SUKO through-zero — "STAT blocking" is emergent, not a rule.
- The Mode 0 leg reaches the interrupt in 0.011 dots — **0.425 dots before
  the CPU-visible mode change**.
- Leg swaps are decided by gate depth: ROPO (LYC) is the fastest leg, PARU
  (Mode 1) next, TARU/TAPA (Mode 0/2) slowest — VBlank entry and exit
  glitch (Cases 1/4); same-line swaps stay covered (Cases 2/3).
- The "DMG STAT-write glitch" is ordinary **transparent-latch bus
  settling** — whether it fires depends on SUKO's through-zero, nothing
  else.
- ROPO survives LCD-off (its reset is system reset, not VID_RST).
```

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| LALU | STAT interrupt latch (IF bit 1) | dffsr | Clock: VOTY; D tied high | Edge-triggered by the SUKO pulse |
| SUKO | Condition combining gate | ao2222 | (ROXE·TARU) + (RUFO·PARU) + (REFE·TAPA) + (RUGU·ROPO) | |
| TUVA / VOTY | Inverter pair | not | SUKO → LALU clock | |
| TARU | Mode 0 condition | and2 | WODU, TOLU | TOLU = NOT(mode1) — non-VBlank gate |
| PARU | Mode 1 condition | not_x1 | `popu_n` | = POPU.Q; also feeds TOLU |
| TAPA | Mode 2 condition | and2 | TOLU, SELA | SELA ≡ RUTU buffered — the line-end pulse |
| ROPO | LY==LYC synced match | dff17 | TALU rising | Captures PALY |
| ROXE / RUFO / REFE / RUGU | STAT.3/.4/.5/.6 enables | drlatch_ee | CUPA strobe | |
| WODU | Mode 0 condition | and2 | XENA, XANO | Full breakdown below; [mode-control](mode-control.md) is the primary chapter |
| PALY | LY==LYC comparator | not_x1 | combinational | Per-bit XNORs → SUBO/SOVU → RAPE → PALY |
| RUPO | STAT bit 2 visible latch | nor_latch | s = ROPO.Q; r = PAGO | Transparent in normal operation (below) |

## The enable bits and the FF41-write transient

The four enables are level-sensitive latches sharing the CUPA strobe — and
that transparency is the netlist origin of the "DMG STAT-write glitch". During a CPU FF41 write, the data bus settles bit by bit across
the write window, and the transparent latches follow it. Measured
(dmg-sim measurement, gambatte `enable_after_lyc_during` ROM, write
$40 → $08): the bits being *cleared* walk through a transient where **all
four enables read 1 simultaneously for ≈ 15.8 ns** before settling to the
written value.

Whether that transient fires an interrupt depends entirely on SUKO's
through-zero behaviour. A transiently-set enable matters only if its
paired condition is high — and if some *other* leg already held SUKO high,
no edge occurs.

In the measured scenario (WODU high mid-H-Blank, LYC leg
high then dropping), SUKO transitions exactly **zero** times: the Mode 0
leg takes over as the LYC leg drops, the output never dips, LALU never
clocks. The glitch is ordinary transparent-latch bus-settling: SUKO =
OR(EN·COND) settles per-leg, and LALU clocks only on a genuine
through-zero.

```admonish warning "Pitfall: the two-pass STAT-write model"
Single-step models miss the transient *and* its consequences consistently
(usually correct); two-pass models that raise-then-drop "all enables on"
without per-leg condition tracking **over-fire**.
```

## WODU: the Mode 0 condition

```
WODU = AND2(XENA, XANO)
├── XENA = NOT(FEPO) — no sprite fetch active
└── XANO = NOT(XUGU) — pixel counter at terminal count (PX = 167)
```

One signal, two consumers with very different latencies: the mode
transition (WODU → VOGA → XYMU, 0.436 dots —
[mode transitions](mode-transitions.md)) and the STAT chain
(WODU → TARU → SUKO → TUVA → VOTY → LALU, 0.011 dots). **The interrupt
fires 0.425 dots before the CPU-visible mode changes.**

### The terminal-count WODU pulse

WODU's inputs settle at different depths. On the advance onto PX=167,
XANO (a shallow NAND5 decode) rises while the deep sprite-match
aggregate FEPO is still low for the new count. With a sprite stacked
**at** X=167, this ordering produces a brief WODU pulse before the
terminal sprite's match suppresses it (dmg-sim measurement):

| Signal | Δ from XANO↑ (ps) | Event |
|--------|---:|---|
| XANO↑ | 0 | PX=167 decode asserts |
| **WODU↑ (early pulse)** | +483 | XENA still high |
| FEPO↑ | +3,141 | terminal sprite match |
| XENA↓ | +3,205 | |
| **WODU↓** | +3,494 | pulse ends — width ≈ 3 ns |

The two consumers treat it asymmetrically: the **combinational STAT
chain catches the pulse** and sets IF bit 1 at the early edge — ahead of
the visible Mode 3 end by the whole terminal-sprite fetch extension —
while **VOGA samples on the ALET edge after FEPO has settled and misses
it** entirely; the mode bits flip only at the sustained WODU rise after
the last terminal sprite. A sprite at X=166 or below produces no pulse
(it is fetched before the terminal count).

## Propagation: measured leg latencies

Mode 0 path (dmg-sim measurement, 1,289 assertions, ps-identical):

| Signal | Δ from WODU↑ (ps) |
|--------|---:|
| TARU↑ | +801 |
| SUKO↑ | +1,671 |
| TUVA↓ | +2,064 |
| VOTY↑ = LALU clock | +2,637 |
| **LALU.q↑ (IF bit 1)** | **+2,637** |

Mode 2 path (dmg-sim measurement, 1,289 assertions):

| Signal | Δ from RUTU↑ (ps) |
|--------|---:|
| SELA↑ | +2,860 |
| TAPA↑ | +3,011 |
| SUKO↑ | +3,881 |
| VOTY↑ = LALU clock | **+4,847** |

The 2,210 ps differential vs Mode 0 is entirely SELA's double-inverter;
the shared SUKO→VOTY tail (966 ps) and the AO2222 input delay (870 ps)
match the Mode 0 leg to ps resolution. The static analysis gives LALU a
tight-but-safe per-line race (diff 5.8 ge, deepest input VOTY at 64.8 ge).

## Edge-triggered behaviour and the leg-swap races

Because LALU clocks on SUKO's *rise*, overlapping conditions produce one
interrupt, not two — the "STAT IRQ blocking" behaviour emerges naturally.
The interesting cases are **leg swaps**: one source leg falls and another
rises on the same boundary. Whether SUKO dips (→ fresh interrupt) or stays
covered (→ nothing) is decided by gate-prop depth, and all four
configurations have been measured (dmg-sim measurements, gambatte
interrupt-precedence ROM family):

**Case 1 — VBlank entry, LYC + Mode 1 enabled: glitch fires.** On the TALU
edge ending scanline 143, the LYC arm drops via ROPO (one TALU-clocked
stage — fast) while the Mode 1 arm rises via NYPE → POPU → PARU (buffered,
~1.9 ns slower). All four arms sit at 0 for **1,802 ps**; SUKO dips; LALU
fires ~4.4 ns after POPU's capture. IF bit 1 sets even though "the LYC
interrupt ended and the VBlank interrupt began".

**Case 2 — line boundary with all four enables: covered.** The Mode 2 arm
rises ~1 ns after RUTU while the Mode 0 arm takes a further dot to drop
(WODU's deep pipeline-end conditions). At every instant some arm is high —
measured across 143 consecutive boundaries: zero SUKO transitions, zero
spurious interrupts. The structural ordering (Mode 2 rises fast, Mode 0
drops slowly) is intrinsic to the netlist.

**Case 3 — VBlank entry, Mode 0 + Mode 1 enabled: covered.** Both arms
cascade off the same POPU chain: the Mode 1 arm rises at PARU (2 stages
from POPU.q), the Mode 0 arm drops at TARU (4 stages, via TOLU). The
rising leg arrives 1,216 ps *before* the falling one — no dip.

**Case 4 — VBlank exit (LY 153→0), Mode 1 + Mode 2 enabled: glitch
fires.** The symmetric counterpart: the Mode 1 arm drops at PARU (2
stages) before the Mode 2 arm can rise at TAPA (4 stages). SUKO dips for
**1,524 ps**; LALU fires ~5.5 ns after POPU's fall. Measured identically
on two independent ROMs.

The general structure, from the shared propagation tree:

```
POPU.q → popu_n → PARU (mode1)            — 2 stages: the FAST leg
                    └→ TOLU → TARU (mode0) — 4 stages
                          └→ TAPA (mode2)  — 4 stages
ROPO (LYC) — 1 TALU-clocked stage: the only leg FASTER than PARU
```

```admonish tip "Rule: the through-zero test"
SUKO glitches low only when, between the falling leg's transition and the
rising leg's, no other (EN·COND) pair is true. On VBlank *entry* with
Mode 1 enabled, PARU's early rise covers everything except the
still-faster LYC drop (Case 1 fires, Case 3 doesn't). On VBlank *exit*,
PARU's early fall uncovers everything (Case 4 fires). Emulators must apply
each leg's transition on its own hardware edge and fire only on a genuine
through-zero: collapsing the POPU-driven arms into one update step misses
Cases 1/4; firing on every leg-rise breaks Cases 2/3.
```

```admonish question "Open question: Case 4 vs hardware annotation"
One ROM (gambatte `m2enable/late_m1disable_ly0_3`) expects IF[1]=0 at a read that
dmg-sim places *after* a Case 4 glitch with no intervening clear — the
simulation says 1. An equivalent-cascade ROM with the clear on the other
side of the boundary is netlist-consistent. The disagreement is
**margin-sensitive**: Case 4's dip is 1,524 ps wide, while the narrowest
dip that hardware-verified expectations *require* to fire is Case 1's
1,802 ps — a 278 ps gap, inside the scale where gate delays are model
predictions rather than hardware facts
([methodology](methodology.md#how-to-read-the-numbers)). A real unit
whose LALU clock threshold sits between the two would match the
annotation *and* the simulated Case 1 — the annotation may simply be
correct for the tested unit. Resolving it needs a hardware measurement.
See [Appendix C](open-questions.md).
```

## The LYC-match pipeline

```
LY bits + LYC bits → per-bit XNORs → SUBO/SOVU → RAPE → PALY (combinational)
  → ROPO (dff17, clk = TALU rising)         — the only registered stage
      ├→ SUKO's LYC arm (with RUGU)
      └→ RUPO (nor_latch) → tri-state → STAT bit 2
```

**RUPO is transparent in normal operation.** Its reset-side driver PAGO is
static-1 outside hardware reset, so the NOR latch continuously re-evaluates
and STAT bit 2 tracks ROPO.Q with gate delay only. A true latch-hold
requires hardware reset *and* a concurrent FF41 write — irrelevant to
normal operation.

**LCD on/off does not reset the pipeline.** ROPO's reset traces to the
system-level hardware reset, not VID_RST. Across an LCD-off span, ROPO
holds whatever it captured at the last TALU edge before the LCD went off;
after LCD-on, the first TALU edge (+1.483 dots after VID_RST deasserts —
[register writes](registers/writes.md)) recaptures (LY=0 == LYC). STAT
bit 2 in the brief window between LCD-on and that first TALU edge reads the
*pre-LCD-off* match value.

## The LYC=153 boundary race

At the 152→153 boundary, MYTA captures FRAME_END one TALU period after
POPU ([line counters](line-counters.md)) and async-resets LY to 0. The
reset propagates to PALY through six gate stages; ROPO's capture on the
same TALU edge completes through four shorter internal stages — **ROPO
wins the race and captures pre-reset PALY**, the LY=153 comparison.

Across three consecutive TALU rises at the boundary:

| TALU rise | Event | ROPO captures |
|-----------|-------|----------------|
| 1 | POPU's edge; LY already 153 | (153 == LYC) |
| 2 | MYTA fires; LY resets *after* the capture | (153 == LYC) — pre-reset, via the race |
| 3 | LY stably 0 | (0 == LYC) |

For LYC=153 the match window is therefore **two TALU periods**; for LYC=0
the match onsets at rise 3 — one TALU period *after* MYTA's edge. The
sub-phase window between PALY's combinational transition and ROPO's next
capture — where the comparator and the visible bit disagree — interacts
with CPU read sampling and is worked through in
[CPU-visible timing at mode boundaries](cpu-visible-boundaries.md).
Implementations that fire POPU and MYTA on a single collapsed edge place
MYTA one TALU period early and shift all of this downstream.
