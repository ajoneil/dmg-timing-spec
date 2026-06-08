# LCD Output

The LCD output path combines the BG and sprite pixel MSBs through a
pixel-multiplexer pipeline gated by LCDC.0 and the sprite priority pipe,
then drives eight physical pads to the LCD glass. This chapter covers the
pixel mux, the per-pixel emission edge (`cp_pad`) and its interaction with
mid-Mode-3 register writes, the horizontal-sync driver, and the full pin
inventory.

```admonish abstract "At a glance"
- `cp_pad`↑ — the glass's pixel-capture edge — lags SACU↑ by **+1,019 ps,
  purely combinationally**: no latch sits between the pixel mux and the
  pads.
- For a mid-Mode-3 write at CUPA↑, the OLD/NEW boundary falls between the
  emissions at **−0.466 and +0.534 dots** around the write.
- CPL/FR keep alternating during LCD-off on **borrowed APU clocks** —
  glass protection; LD0/LD1/CP/CPG hold at 0 under VID_RST.
- The LCDC.0 overlay is the third **below-netlist** LCD-interface
  behaviour: every mid-Mode-3 write defers one column, bidirectionally.
- The first frame after LCD-on emits **no VSYNC pulse**.
```

## The pixel multiplexer

| Gate | Role | Type | Inputs | Notes |
|------|------|------|--------|-------|
| TADE | BG plane B output (LCDC.0 gated) | and2 | SOHU (`bg_px_b`), VYXE (`ff40_d0`) | LCDC.0=0 forces plane B to 0 |
| RAJY | BG plane A output (LCDC.0 gated) | and2 | PYBO (`bg_px_a`), VYXE | Same gating, plane A |
| RYFU | BG plane A gated by sprite priority | and2 | RAJY, VAVA | VAVA from the priority pipe ([sprite pipeline](sprite-pipeline/storage-matching.md)) |
| RUTA | BG plane B gated by sprite priority | and2 | TADE, `sprite_px_priority` | Plane-B analogue of RYFU |
| POKA | BG-pixel combine | nor3 | NULY, RUTA, RYFU | `bgpx` = NOT(POKA); feeds the MOKA/NURA/WUFU combiners ([sprite pipeline](sprite-pipeline/lcdc-paths.md) drives NULY here) |
| MOKA | OBP1 palette AO2222 combiner | ao2222 | `bgpx` vs sprite selection | |
| NURA | BGP palette AO2222 combiner | ao2222 | `bgpx` vs sprite selection | |
| WUFU | OBP0 palette AO2222 combiner | ao2222 | `bgpx` vs sprite selection | |
| PATY | Pixel-mux output | or3 | MOKA, NURA, WUFU | |
| RAVO / REMY | LD1 / LD0 drivers | not | PATY / PERO | |

## CPL and FR: the column alternators

`cpl` and `fr` alternate the LCD column drivers. Their sources are AO22
muxes selected by LCDC.7:

- KAHE = AO22(`ff40_d7`, KASA, KEDY, UMOB) — CPL source
- KUPA = AO22(`ff40_d7`, KEBO, KEDY, USEC) — FR source
- KEDY = NOT(LCDC.7) selects the second arm

With the LCD **on**, KAHE follows KASA = RUTU.Q (once-per-scanline LINE_END
cadence) and KUPA follows KEBO = NOT(MECO). With the LCD **off**, both pins
are muxed onto buffered APU audio dividers — UMOB (~8 kHz) for CPL, USEC
(~4 kHz) for FR.

The rationale is glass protection: LCD column cells must see a net-zero
average voltage, so CPL/FR must keep alternating even with the LCD off —
the silicon borrows APU clocks to do it. Measured across a full pre-LCD-on
boot window: `cpl_pad` alternates at ~8 kHz and `fr_pad` at ~4 kHz
throughout, then both switch to per-scanline cadence at the LCD-on edge
without a glitch (dmg-sim measurement).

The other four pins (`cp`, `cpg`, `ld0`, `ld1`) are *not* audio-muxed —
they hold at 0 during LCD-off. Keep the two mechanisms apart: LCDC.7=0
holds the whole rendering machinery in VID_RST (the pixel pipeline never
advances), whereas LCDC.0=0 merely gates the BG planes at RAJY/TADE during
active rendering.

## `cp_pad`: the pixel-emission edge

`cp_pad` is the pixel clock the LCD glass uses to capture LD0/LD1 — one
pixel per rising edge. The drive path from SACU is purely combinational:

> **SACU↑ → TOBA↑ (= AND2(PX9, CLKPIPE)) → SEMU↑ (= OR2(TOBA, POVA)) →
> RYPO↓ → `cp_pad`↑**

Net delay SACU↑ → `cp_pad`↑: **+1,019 ps**; the falling edge lags SACU↓ by
+1,711 ps — identical across 68,256 rises (dmg-sim measurement), with the
once-per-line POVA-arm capture at its own +4,493 ps offset. TOBA's PX ≥ 9
gate is why the first 8 CLKPIPE cycles never reach
the glass, and the POVA arm of SEMU supplies the end-of-line 160th-pixel
capture ([BG pipeline](bg-pipeline/pixel-clock.md)).

### The palette-write timeline, glass-side

For a palette write whose CUPA↑ lands at dot N, the emission edges bracket
the on-die race ([palette latches](registers/palette-latches.md)) like this:

| Δ from CUPA↑ | Event |
|---|---|
| −0.466 dots | **Prior `cp_pad`↑ — the LCD captures dot N−1's pixel.** LD0/LD1 still hold the OLD palette result |
| 0 | CUPA↑ |
| +0.005 / +0.076 / +0.080 dots | SACU↓ / PERO / LD-driver transition to the NEW result |
| **+0.534 dots** | **Next `cp_pad`↑ — the LCD captures the NEW-palette pixel** |

No latching exists between the pixel-mux output and the pad.

```admonish warning "Pitfall: no extra pipeline stage"
The OLD/NEW boundary falls between the columns emitted at −0.466 and
+0.534 dots around the write — a model that adds any further one-dot lag
between mux output and LCD column mis-places the boundary one column
right.
```

### The LCDC.0 mid-Mode-3 overlay

At netlist resolution, a mid-Mode-3 LCDC.0 write behaves exactly like the
palette write above: VYXE settles within 41–69 ps of CUPA↑, the
combinational chain follows, and the `cp_pad`↑ at +0.534 dots samples the
NEW value (dmg-sim measurement across five anchor scanlines; the static
race analysis lists no per-dot entries anywhere in the
VYXE→RAJY/TADE→NURA→PATY chain).

On real hardware there is one more wrinkle — third in the family of
below-netlist LCD-interface overlays (with the BGP OR-overlap and the
LCDC.1 OFF transient):

> When a CUPA write transitions LCDC.0 inside Mode 3, the first `cp_pad`↑
> after that CUPA↑ emits a pixel computed using the **OLD VYXE state**; the
> NEW state takes effect from the second `cp_pad`↑ onward. The rule is
> **bidirectional** (both 1→0 and 0→1) and applies to every Mode-3 write —
> no first-write-of-scanline exception.

Characterised against the Mealybug `m3_lcdc_bg_en_change` hardware
reference, which shows each transition deferred by exactly one LCD column
relative to the netlist prediction, in both directions, across the
affected scanline range.

The three family members have distinct scopes but share one signature:

| Member | Scope |
|---|---|
| LCDC.0 | bidirectional, every write |
| LCDC.1 | OFF direction only |
| BGP | second-or-later write |

In every case the netlist captures CUPA-edge propagation cleanly, and the
hardware exhibits a +1-column overlay at the LCD-pipeline boundary. The
silicon mechanism (glass-side sample-and-hold vs pad-driver residue) is a
shared **open question** ([Appendix C](open-questions.md)).

## The ST (horizontal sync) driver

ST is driven by a three-cell combinational feedback loop gated by a
PX-bit-3 tracker and synchronised per scanline by AVAP:

| Gate | Role | Type | Inputs | Notes |
|------|------|------|--------|-------|
| POME | Loop AVAP gate | nor2 | AVAP, POFY | |
| RUJU | Loop OR combiner | or3 | PAHO, POME, TOFU | TOFU is the video-reset arm |
| POFY | Loop inverter + feedback | not_x1 | RUJU | Feeds POME and the pad buffer |
| PAHO | PX-bit-3 capture | dffr | ROXO | Data: XYDO (PX bit 3); reset by XYMU outside Mode 3 |
| RUZE | ST pad buffer | not_x3 | POFY | |

The loop dynamics: during the AVAP pulse, POME is forced low and POFY
initialises to NOT(PAHO) = 1 (PAHO was reset throughout Mode 2). During
Mode 3, PAHO tracks XYDO on each CLKPIPE cycle — PX-bit-3-aligned
transitions — and the first PAHO=1 event with POFY=1 drops POFY to 0,
where the loop self-holds until the next scanline's AVAP.

```admonish info "Measured: the ST pulse shape"
ST is **one high pulse per scanline: [AVAP+0.008, AVAP+14.493] dots, width
14.485 dots** at SCX=0 — POFY arms at AVAP+0.006, the first PAHO=1 capture
(PX bit 3 first high, ≈PX 8–9 through ROXO) drops it at +14.489, and the
pin follows each edge ~490 ps later. Zero variance across sampled
scanlines (dmg-sim measurement). The rising edge is AVAP-anchored; the
falling edge tracks the first PX-bit-3 capture and therefore shifts with
fine scroll.
```

**LCD-off and release.** During LCD-off, TOFU=1 forces POFY=0 and the pad
is static; PAHO is additionally held by XYMU. At the LCD-on edge TOFU
drops combinationally, the loop stays quiescent, and the first AVAP
produces an ST edge on the first scanline — HSYNC is normal from the
outset. Measured: first `st_pad`↑ inside that scanline's Mode 3, the
second exactly one 454-dot first-scanline period later, and per-scanline
cadence thereafter — with the second scanline onward at the steady 456-dot period
(dmg-sim measurement; the 2-dot first-scanline shortening sits in Mode 0 and
shifts the cadence once without altering pulse shape).

```admonish question "Open question: HSYNC at the glass"
The pulse shape above is measured; what remains open is the
interpretation at the glass — pulse-width expectations and the row-driver
response rest on community pin-role references, not measurement. See
[Appendix C](open-questions.md).
```

## Pin inventory

Eight pads connect the PPU to the glass. Each pad is driven through a
single inverting driver stage into an inverting pad cell (`pad = !o_n`) —
two inversions, so **each pin tracks its source signal in phase** (measured:
the st pin follows POFY with ~490 ps of buffer delay):

| Pad | Driver cell | Source | Primary description |
|-----|-------------|--------|---------------------|
| `s` (VSYNC) | MURE (not_x1) | MEDA | [Line counters](line-counters.md) (the LY=0 capture) |
| `st` (HSYNC) | RUZE (not_x3) | POFY | this chapter |
| `cp` (pixel clock) | RYPO (not_x1) | SEMU = OR2(TOBA, POVA) | this chapter |
| `cpg` (clock pulse gate) | POGU (not_x1) | RYNO = OR2(SYGU, VCLK); VCLK = RUTU.Q | per-scanline gate |
| `cpl` | KYMO (not_x1) | KAHE (AO22 on LCDC.7) | this chapter |
| `fr` | KOFO (not_x1) | KUPA (AO22 on LCDC.7) | this chapter |
| `ld0` | REMY | PERO | pixel plane 0 |
| `ld1` | RAVO | PATY | pixel plane 1 |

**The VSYNC path.** MEDA (the NYPE-clocked LY=0 capture) drives `s_pad`
through the MURE inverter — one gate delay end to end. MEDA's first 0→1
transition after LCD-on lands at the LY 153→0 boundary *ending* the first
frame, so **the first frame emits no VSYNC pulse** — part of the LCD-on startup
transient ([LCD-on power-up](scanline-frame-timing/lcd-on.md)).
Community pin-role documentation describes pin S as the input to the LCD's
Y-driver row shift register (one row per pulse); that interpretation is
not netlist-derivable.

**The pads are stateless.** The LD0/LD1 pad cells have no clock and no
stateful element; their propagation is fixed at the netlist level,
independent of pixel index or upstream pipeline state. Measured across
both branches of a palette-write iteration family: the CUPA-relative
offsets at the chip pin are picosecond-identical within each branch, and a
known one-M-cycle upstream differential reproduces at the pin without
compression (dmg-sim measurement). Whatever produces the LCD-interface
overlay family above, it is not a stateful pad driver on the die.
