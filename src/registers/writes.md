# Writes

Reads drive the bus combinationally ([reads](reads.md));
writes work the other way around — the PPU is the receiver, so the write
must strobe the PPU-side register latches closed on specific edges. Every
writable PPU register (LCDC, STAT, SCX, SCY, LYC, BGP, OBP0, OBP1, WX, WY)
is a level-sensitive latch, transparent during the CUPA strobe — which
fires once per M-cycle at a deterministic position, making every writable
register transparent at the same time.

```admonish abstract "At a glance"
- **One shared strobe**: every writable PPU register is a level-sensitive
  latch enabled by AND(address decode, CUPA).
- CUPA rises at the start of **T3** and falls mid-**T4** — 1.493
  dots wide; the latch is transparent for the whole pulse.
- What a mid-rendering write affects is decided by the **bus-settle
  instant** (~17 ns into the pulse), not by the window edges.
- An LCDC.7=1 write releases VID_RST **3,173 ps after CUPA↑** — purely
  combinational, no synchroniser — and the dividers count cleanly from
  zero, which is why the first scanline runs ≈454 dots.
```

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| CUPA | Shared PPU register write strobe | combinational (derived from phase gen) | — | Primary here; fires once per M-cycle, 1.493 dots wide. Its output wire is named `ppu_wr` in the simulation — `ppu_wr` elsewhere in this book is this strobe |
| DYKY | CUPA driver stage 1 | not | Input: TAPU | |
| TAPU | CUPA driver stage 2 | not | Input: UBAL | |
| UBAL | CUPA driver stage 3 | muxi | Test gate | |
| APOV | CUPA driver stage 4 | not | Input: AREV | |
| AREV | CUPA driver stage 5 | nand2 | Inputs: `cpu_write`, AFAS | |
| AFAS | Ring decode driving CUPA | nor2 | Inputs: ADAR, ATYP | High three ATAL half-cycles per M-cycle (1.504 dots measured) |
| AFUR | Phase generator ring stage 1 | dff | Driven by ATAL/ADEH | 4-stage ring (AFUR/ALEF/APUK/ADYK) |
| ALEF | Phase generator ring stage 2 | dff | Driven by ATAL/ADEH | |
| APUK | Phase generator ring stage 3 | dff | Driven by ATAL/ADEH | |
| ADYK | Phase generator ring stage 4 | dff | Driven by ATAL/ADEH | |
| WARU | LCDC write enable | and2 | Inputs: VOCA, CUPA | Enables XUBO/XURE |
| SEPA | STAT write enable | and2 | Inputs: VARY, CUPA | Enables RYVE/PUPU |
| VELY | BGP write enable | and2 | Inputs: WERA, CUPA | Enables TEPO/LYFA ([palette latches](palette-latches.md)) |
| ARUR | SCX write enable | and2 | Inputs: XAVY, CUPA | — |
| BEDY | SCY write enable | and2 | Inputs: XARO, CUPA | — |
| XODO | VID_RST signal | combinational | Input: XONA | Deasserts 3,173 ps after CUPA↑ on an LCDC.7=1 write |
| XONA | LCDC.7 storage latch | drlatch_ee | d: d7; ena: XURE/XUBO | q (`ff40_d7`) drives XODO/VID_RST on an LCDC.7 write |

## The shared CUPA strobe

All CPU-writable PPU registers are level-sensitive latches (`drlatch_ee` or
`dlatch_ee`) whose enable is AND(address decode, CUPA):

- LCDC: WARU = AND2(VOCA, CUPA) → XUBO/XURE
- STAT: SEPA = AND2(VARY, CUPA) → RYVE/PUPU
- BGP: VELY = AND2(WERA, CUPA) → TEPO/LYFA
- SCX: ARUR = AND2(XAVY, CUPA)
- SCY: BEDY = AND2(XARO, CUPA)
- …and so on: every writable register shares CUPA, with per-register address
  decode.

## Phase generator derivation

> **ADAR/ATYP ring decode → AFAS → AREV (with
> `cpu_write`) → APOV → UBAL → TAPU → DYKY → CUPA**

The phase generator is a 4-stage ring (AFUR/ALEF/APUK/ADYK) driven by
ATAL/ADEH, the complementary 4 MHz enables from the
[clock tree](../clock-tree.md). Each stage output is high for four consecutive
ATAL half-cycles — 2 dots — per M-cycle (measured: AFUR 487,713 ps, ADYK
488,188 ps). The AFAS decode is high for **three half-cycles per M-cycle**
— 367,065 ps = 1.504 dots, from the ALET rising edge of dot 2 to the ALET
falling edge of dot 3 (dmg-sim measurement) — and CUPA follows it through
the five-stage drive chain at 1.493 dots.

## CUPA pulse position

Measured across three independent CUPA pulses with identical alignment
(dmg-sim measurement):

- **Rises at the start of T3**, 4,848 ps after T3's ALET rising edge.
- **Falls mid-T4**, 1,716 ps after its ALET falling edge.
- **Width: 364,298 ps = 1.493 dots.**

The register latch is transparent for the entire pulse; bus data propagates
through combinationally.

**Alignment to TALU/SONO.** One TALU cycle = one M-cycle = 4 dots. CUPA
fires at the start of T3; the [LX counter](../line-counters.md) increments on TALU
rising. That alignment decides which LX count a write lands on.

## Behaviour on the write dot

While CUPA is high (T3–T4): the latch is transparent — combinational
consumers and sampling DFFs see the new value as soon as the bus carries
it. When CUPA falls (T4's ALET falling edge): the latch closes and
holds.

**The bus, not the strobe, sets the commit instant.** The latch is open for
the whole 1.493-dot pulse, but the captured value follows the CPU data
bus — which settles ~17 ns in, just past the write-dot SACU↓. The palette
write shows the consequence: the BGP dlatch updates ~15.6 ns *after* that
pixel edge, so the pixel shifted on the write dot uses the OLD palette
value ([palette latches](palette-latches.md)). What a mid-rendering
write affects is decided by the bus-settle instant, not by the window
edges. (`clk_t4`, the CPU's bus-drive clock, is the
BAPY = NOR3(`clk_ena_n`, AROV, ATYP) decode — the same ring vocabulary as
CUPA.)

## VID_RST deassertion

After a CPU write sets LCDC.7=1, the VID_RST signal (XODO) deasserts through a
purely combinational chain (dmg-sim measurement):

- CUPA↑ → XONA↑ (+1,003 ps; the LCDC.7 bit captures) → XODO↓ (+2,170 ps further)
- Total CUPA↑ to XODO↓: **3,173 ps** (~0.013 dots)

At the instant XODO deasserts, the divider toggle DFFs are in reset (q=0):
WUVU = VENA = BOGA = 0, with ATAL = ALET = 1. The combinational complements
settle within gate delays of the just-reset q_n outputs: TALU = NOT(`vena_n`)
= 0, XUPY = NOT(`wuvu_n`) = 0, SONO = NOT(TALU) = 1.

The dividers then count cleanly from zero (dmg-sim measurement):

| Event | Time after XODO↓ | Dots |
|---|---|---|
| WUVU.q↑ (toggle DFF) | +114,938 ps | +0.471 |
| XUPY↑ (combinational from `wuvu_n`, +2,135 ps after WUVU.q↑) | +117,073 ps | +0.480 |
| VENA.q↑ (toggle DFF, captures `wuvu_n`'s first rise) | +360,224 ps | +1.476 |
| TALU↑ (combinational from `vena_n` through the not_x4, +1,628 ps after VENA.q↑) | +361,852 ps | +1.483 |

VID_RST therefore clears within a single dot of the CPU's write; there is no
multi-cycle synchroniser delay.

### First-scanline consequence

The write's T3 landing plus the divider startup phase make the post-LCD-on
first scanline run **≈454 dots** instead of 456, the deficit entirely in its
Mode 0 — Mode 2 and Mode 3 match steady state. The decomposition, the
direct Mode-0-span measurement, and the phase-conservation proof live in
[LCD-on power-up](../scanline-frame-timing/lcd-on.md).
