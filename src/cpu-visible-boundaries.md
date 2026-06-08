# CPU-Visible Mode Boundaries

What does a CPU read latch when it lands *on* a PPU transition? This chapter
answers that question edge-by-edge for the four STAT-readout boundaries and
the LY 153→0 wrap, plus the write-side analogue (an OAM write straddling the
Mode 2→3 boundary). The recurring structure: a PPU trigger fires at some
sub-dot phase; a gate cascade settles in nanoseconds; the *bus driver* takes
far longer; and the CPU latches at `data_phase_n`↑, +974,856 ps (+3.995 dots — the tail
of T4) after its M-cycle's CLK9↑. Which of those windows the read M-cycle straddles
decides PRE vs POST — and the proximate mechanism differs per transition.

```admonish abstract "At a glance"
- The CPU latches the bus at the **tail of T4** (+3.995 dots into its
  read M-cycle) — late;
  what matters is the bus state then, not the gate cascade.
- STAT mode-bit reads on a trigger M-cycle latch **PRE**: the `not_if1`
  bus drivers stay unresolved through the rest of the M-cycle (the
  x-window). LY reads have **no such window** (`not_if0` drivers).
- Mode 3→0 has no torn middle at single speed: VOGA quantises XYMU↑ onto
  the ALET grid — every read is provably fully-PRE or fully-POST.
- The Mode 2→3 cascade opens a measured **~6.5 ns OAM write permit**
  between Mode 2 closing and Mode 3 locking — a straddling write's strobe
  chain completes with ~4.6 ns to spare.
```

## The LYC stale window

Between PALY's combinational transition (the comparator seeing the new LY)
and ROPO's next TALU-edge capture, the comparator and the visible STAT bit 2
disagree ([STAT interrupts](stat-interrupts.md)). The CPU never sees the
stale value on a normal read: its `data_phase` sample lands late enough in
the read M-cycle that ROPO's transition has propagated to `cpu_port_d`
first.

Measured on a read M-cycle containing the transition (dmg-sim
measurement, gbmicrotest `line_153_lyc153_stat_timing` ROM family): the bus flips
0xC5 → 0xC1 at the instant `rupo_n` transitions, ≈117 ns before the CPU's
latch point (−116,838/−116,911 ps across two variants) — the CPU captures
0xC1, matching hardware. Emulators that latch CPU-visible reads early in
the M-cycle report the stale value instead.

## Mode 2 → 3: the AVAP dot

Two paths fan out from AVAP↑ to the STAT mode-bit gates SADU = NOR2(mode3,
mode1) and XATY = NOR2(mode2, mode3): the BESU branch clears mode2 in
~2.2 ns; the buffered XYMU branch raises mode3 at ~8.5 ns. In between, XATY
sees NOR(0,0) — a **~6.3 ns transient "Mode 0" glitch on bit 1** (dmg-sim
measurement, zero variance across 1,720 scanlines):

| Window from AVAP↑ | STAT[1:0] |
|---|---|
| before | 10 (Mode 2) |
| ~2.2 ns … ~8.5 ns | **00 (transient)** |
| after ~8.5 ns | 11 (Mode 3) |

The glitch closes ~350 ns before any plausible CPU latch — never directly
observable. What *is* observable is the bus: the mode-bit drivers are
`not_if1` tri-state cells whose output transition plus contention
holds the transitioning bit unresolved **through the rest of the trigger
M-cycle**. Measured on a read whose M-cycle contains AVAP↑ (dmg-sim
measurement, gbmicrotest `lcdon_to_stat3_c`): AVAP fires mid-T3 (+2.5 dots), the
gate cascade settles in ~8.5 ns, the bus bit starts transitioning at
~69 ns — and is still unresolved at the CPU's latch, ~360 ns later (it
resolves ~21 ns after).

Hardware latches the **PRE-transition value** (Mode 2 — the bus voltage
hasn't crossed the input threshold), matching the hardware-verified
expectation; the sibling ROM one M-cycle later reads Mode 3. **The threshold
is not the ~3 ns cascade — it is the bus-driver settling window.**

## Mode 0 → 2: the BESU-set boundary

The same bus-driver mechanism with a faster cascade: BESU.q rises
combinationally from CATU at the scanline boundary, mode2 rises at ~3.4 ns
(2.5× faster than the AVAP cascade — one combinational stage closer), only
bit 1 transitions. Measured on the trigger M-cycle (dmg-sim measurement,
gbmicrotest `lcdon_to_stat2_c`): gate settle at ~3.4 ns, bus bit unresolved from ~61 ns
through the CPU latch (it resolves ~20 ns after) — hardware latches **PRE**
(Mode 0); one M-cycle later reads Mode 2.

## Mode 3 → 0: the WODU cascade

This transition differs structurally: the cascade contains a DFF (VOGA,
ALET-clocked), so it takes ~106 ns rather than ~8.5 ns, and WODU↑'s phase
within the read M-cycle **varies with Mode 3's duration** (set by SCX,
sprites, and window). Both mode bits transition. Five measured anchors
span the regime structure (dmg-sim measurements; all five ROMs are
gbmicrotest; parenthesised offsets are dots from the M-cycle's start):

| Anchor | WODU↑ position | XYMU↑ vs CPU latch | Latches | Mechanism |
|---|---|---:|---|---|
| `lcdon_to_stat0_c` | read M-cycle, T4 (+3.556) | +0.006 dots after | **PRE** (0x83) | cascade misses the latch |
| `sprite_0_a` | next M-cycle, T3 (+2.556) | +3.00 dots after | **PRE** (0x83) | cascade entirely after the read |
| `sprite_0_b` | read M-cycle, T3 (+2.556) | −1.00 dots before | **POST** (0x80) | cascade flips the already-settled bus |
| `win0_b` | read M-cycle, T2 (+1.556) | −2.00 dots before | **POST** (0x80) | cascade beats the read driver; bus settles directly to POST |
| `lcdon_to_stat0_d` | prior M-cycle, T4 (+3.556) | −4.00 dots before | **POST** (0x80) | bus settled a full M-cycle earlier |

All five share a bit-identical gate cascade; only the alignment against the
read M-cycle differs. Every expectation matches hardware-verified ROM
results.

**The ambiguous middle does not exist at single speed.** VOGA quantises
XYMU↑ onto the ALET grid — one possible position per dot. The WODU→XYMU
*delay* is not a constant ("next ALET edge minus WODU, plus a DFF delay" —
measured shrinking exactly as WODU↑ slides toward the capture edge); the
*grid position* is the invariant. For the canonical FF41-read M-cycle, the
nearest grid points land +0.006 dots after the CPU latch and −1.00 before it —
nothing in between is reachable, so every single-speed read is provably
fully-PRE or fully-POST.

The residual metastability sits at **VOGA's capture**: a WODU↑ inside the
setup/hold window flips the outcome between two *clean* values a full dot
apart — whole-mode selection, not a torn read.


One scoping caveat from the anchor family: "PPU timing is
prelude-invariant" holds per-scanline only when that scanline's Mode 3 is
not coupled to in-flight OAM DMA. A sibling ROM pair whose preludes pause
DMA differently across HALT showed three mid-frame scanlines differing by
one 11-dot sprite penalty — because DMA's byte 1 (a sprite X write) landed
in one ROM's scan window and not the other's ([DMA](dma/boundaries.md)).

## LY 153 → 0: the MYTA reset

The LY wrap is the clean counter-example to the STAT cases. MYTA fires on a
TALU edge at the *start* of an M-cycle (T1); the LAMA reset cascade to
the LY DFFs and the FF44 read drivers is purely combinational (~3–4 gate
stages); and crucially the LY drivers are `not_if0` cells — single
tri-state inverters with **no companion-driver contention**, so no
hundreds-of-ns unresolved window exists.

- A read whose M-cycle starts on MYTA's edge latches **0x00** — the bus has
  been at the post-reset value for nearly the whole M-cycle.
- A read one M-cycle earlier latches **0x99** (153).

Measured anchors place two hardware-verified ROMs exactly one M-cycle
apart across this boundary (latch points 975,927 ps apart), with the
LCD-on→first-MYTA interval matching the analytic frame structure
(454 + 152 × 456 + 4 dots) to within 0.13 M-cycle (dmg-sim measurement,
gbmicrotest `line_153_lyc153_stat_timing` family). The same boundary M-cycle read
on FF41 catches the LYC stale window with bit 2 mid-transition, resolving
to the post-clear value per the first section above.

```admonish tip "Rule: the driver-cell distinction"
**`not_if1` readouts (STAT mode bits) have trigger-M-cycle bus-settling
windows; `not_if0` readouts (LY) do not.** When a transition lands inside
a read M-cycle, the driver cell type decides whether the read resolves PRE
(x-window) or simply follows the settled bus. The driver map is
netlist-enumerated for every register
([register reads](registers/reads.md)): besides STAT's live bits, only
DMA reads through `not_if1` — and its source register changes only on an
FF46 write, which cannot land inside an FF46 read M-cycle. STAT bits 0–2
remain the only x-window-capable readout.
```

## The write-side straddle: AJUJ's ~6.5 ns window

The Mode 2→3 cascade's mode2/mode3 gap exists on the *write-permit* path
too: AJUJ = NOR3(`dma_run`, mode2, AJON) sees mode2 fall at +1,951 ps and
AJON rise at +8,759 ps — the permit is open from **+2,318 to +8,820 ps
after AVAP↑, a 6,502 ps window** between Mode 2 closing and Mode 3
locking (dmg-sim measurement, 1,720 scanlines, zero variance).

A CPU OAM write whose CUPA strobe spans this window fires the full
WYJA → strobe chain. Measured in situ on the landing anchor (dmg-sim):

| Event | Δ |
|---|---|
| AVAP↑ | +122,925 ps into the write strobe |
| AJUJ opens (permit) | +2,318 ps after AVAP↑ |
| WYJA fires | +1,349 ps after the permit |
| YNYC byte strobe | +513 ps after WYJA |

That is **1,862 ps from permit to strobe, completing 4,640 ps before the
window closes**. The OAM SRAM cell — a transparent latch with no
setup/hold edge — captures the bus value during the strobe-high residue.
The hardware-verified anchor is gbmicrotest `oam_write_l1_c` (write
*lands* on DMG), bracketed by one-NOP siblings whose writes are blocked:
blocked / **LANDS** / blocked.

The structurally parallel boundaries behave differently: Mode 0→2 has no
transient gap (mode2 rises directly; AJUJ closes cleanly — write blocked),
and Mode 3→0 simply *opens* the permit: AJON releases at WODU↑ +111,054 ps
and AJUJ opens at +111,421 ps — 4,999 ps after the mode-bit change
(dmg-sim measurement, 1,721 scanlines, zero variance).

A straddling write across this opening is demonstrated directly with the
purpose-built `oam_write_m3end` sweep, with SCX = 3 phasing the boundary
into the strobe — fine scroll shifts Mode 3's end in 1-dot steps. The
landing position, from the CUPA strobe's rise (dmg-sim):

| Event | Δ from strobe rise |
|---|---|
| Strobe begins (permit still closed) | 0 |
| AJUJ opens (permit) | +245,254 ps (1.005 dots) |
| WYJA fires | +1,349 ps after the permit |
| YNYC byte strobe | +1,414 ps after WYJA |

That leaves 116,281 ps of strobe-high residue for the transparent OAM cell
to capture, and the WYJA latency matches the Mode 2→3 case. Ladder
positions either side bracket it: writes wholly inside Mode 3 never fire
WYJA (blocked); writes after the opening land 396 ps from the strobe's
rise. Blocked / **LANDS** / clean — the write-side mirror of the
Mode 2→3 straddle.
