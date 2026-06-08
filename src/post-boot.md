# Post-Boot State

At the first PC=0x0100 instruction fetch after the real DMG boot ROM, the
machine is in a single, fully-measured hardware state. This chapter pins
that state — to the *edge*, because "PC=0x0100" is ambiguous without it —
and gives skip-boot emulators the exact values to adopt.

```admonish abstract "At a glance"
- The anchor is the CLK9 edge **opening the overlap M-cycle** of the boot
  ROM's terminating LDH — anchoring at the write instead is 3 dots early.
- The handoff lands at **LX = 98 of the first scanline's Mode 0**; adopt 98
  directly — the shortening is baked in.
- `reg_div16` = **0xEAF3** — FF04 alone under-constrains the timer phase.
- **IME is disabled**; IF = 0xE1 with the VBlank bit set.
- The frame-sequencer ripple seed is **Δ = 3** — not derivable from the
  divider; CH1's duty counter holds **2** with the DAC on.
```

**The anchor, precisely.** Under SM83 fetch overlap, the boot ROM's
terminating `LDH (FF50),A` performs its post-body fetch in the same
physical M-cycle as the cartridge instruction's first fetch. The anchor is
the CLK9 edge that **opens** that M-cycle: `cpu_port_a` transitions to
0x0100, `reg_pc` holds 0x0100, the FF50 write committed in the *prior*
M-cycle, and the M-cycle counter reads m7 (the post-body fetch state —
[interrupt dispatch](interrupt-dispatch.md)).

Anchoring at LDH's mid-write instead places the CPU 3 dots early and
propagates a CPU/PPU phase error through everything downstream (dmg-sim
measurement, picosecond-identical across three captures).

The handoff lands **deep in the first scanline's Mode 0** — LX = 98 of 113, 15
M-cycles before LINE_END — with the OAM scan long complete (scan counter
at its terminal 39).

## PPU state

| Signal | Value | Notes |
|--------|-------|-------|
| WUVU / VENA / TALU / XUPY | 0 / 1 / 1 / 0 | Divider phases; VENA=1 with WUVU=0 follows from the toggle timing since LCD-on |
| BOGA / ALET | 1 / 1 | T1 of the M-cycle; ALET rising phase |
| XYMU | 1 | Not in Mode 3 |
| BESU / AVAP | 0 / 0 | Scan complete; no transition in flight |
| LY | 0 | |
| **LX** | **98** | FST-measured per bit: (SAXO…TYRY) = 0b1100010. 98 × 4 = 392 dots into the 456-dot line. The first-scanline shortening and divider startup are already baked in — **adopt 98 directly**; do not synthesise a pre-shortening value ([LCD-on power-up](scanline-frame-timing/lcd-on.md)) |
| Scan counter | 39 | Terminal (0b100111) |
| MEDA | 1 | Mid-frame-pulse (fires at each LY 153→0 wrap, holds through LY=0) |
| POFY / PAHO | 0 / 0 | ST loop settled; PX-bit-3 capture in its XYMU-held reset |
| BG fetch counter | 5 (frozen) | Terminal value held by the MOCE self-stop since the last Mode 3 |
| Fetch cascade DFFs (NYKA, PORY, LYZU, PYGO, RENE, RYFA) | all 0 | Mode-3-only / NAFY reset domains |
| Fine-scroll counter + match pipeline | all 0; ROXY = 1 | PASO-held; SACU gated high awaiting the next Mode 3 |
| CLKPIPE tree | SACU=1, SEGU=1, TYFA=0, VYBO=0, SOCY=1 | All derivable from the non-Mode-3 state; WODU=1 residually (PX still holds the terminal-count bit pattern from the boot ROM's last Mode 3) |
| No-reset shifters/temp latches | boot-ROM residue | Never observable: the two-parallel-loads mechanism overwrites BG content before any pixel samples, and sprite content is mux-gated ([BG pipeline](bg-pipeline/shifters.md)) |

## CPU-visible registers

| Register | Value | Source |
|----------|-------|--------|
| LCDC | 0x91 | boot ROM |
| STAT | 0x85 | Mode 1 bits + **LYC match set** (LY=LYC=0) + bit 7 (dmg-sim post-boot readback; the mode register reads 01 at the boot→cart handoff, settling to 0 within a few lines) |
| SCY / SCX / WY / WX / LYC | 0x00 | power-on defaults |
| BGP | 0xFC | boot ROM |
| OBP0 / OBP1 | undefined; use 0xFF | the palette latches have **no reset path** — true silicon power-on state is undefined; real DMGs consistently read 0xFF |
| DIV (FF04) | 0xAB | see the divider block below |
| TIMA / TMA | 0x00 / 0x00 | |
| TAC | 0xF8 | |
| IF | 0xE1 | bit 0 (VBlank) set — captured at the boot ROM's last line-144 entry; benign (IE=0) but visible to cartridge code |
| IE | 0x00 | |

```admonish tip "Rule: the LYC-match consistency check"
LYC=0 with LY=0 **requires** STAT bit 2 = 1 at handoff. Pairing LYC=0
with a clear match bit means the LYC pipeline state contradicts the
register state.
```

## CPU dispatch chain

Every dispatch-chain cell is idle (measured, picosecond-identical across
three captures): no dispatch in flight, nothing latched, the per-bit IF&IE
latches all reading "not pending", and — the one that matters —
**IME is DISABLED** (`ime_n` = 1). Skip-boot initialisers that set IME=1
contradict the measured handoff.

## The divider, exactly

`reg_div16` = **0xEAF3** at the anchor (settling from 0xEAF2 just after
the boundary edge — bit 0's normal toggle, not a cascade).
DIV = bits [13:6] = 0xAB.

Three subtleties:

1. **FF04 alone under-constrains the state.** The low six bits (0x33) are
   software-observable through TIMA-increment phase; an emulator matching
   FF04 but not `reg_div16` diverges on the timer-phase test family.
   Pre-load the full 0xEAF3.
2. **The simulation's bit 15 is a harness artifact.** The boot-ROM
   execution measurement shows `reg_div16 = (0x8000 + M-cycles) mod 2^16`
   because the sim deasserts the divider reset ~32,768 M-cycles before the
   CPU starts; real hardware would read 0x6AF3. The difference is
   invisible to all software (no bit-15 consumer), so either value is
   correct — but know which convention you are matching.

For boot-ROM-executing emulators: the total is **5,860,082 M-cycles** from
reset deassert to the anchor (dmg-sim measurement with the real boot ROM),
with a per-PC checkpoint table dominated by the VRAM clear (~57k), the
logo scroll (~4.25M), and the jingle wait (~1.4M). One sampling caveat
carries over: several checkpoint PCs are byte-fetch transients of
multi-byte instructions, so comparisons must use literal per-byte PC
advance, not instruction-boundary semantics.

## APU state

The boot ROM powers the APU, configures CH1, triggers it twice for the
chime, and lets the envelope decay to silence. Measured at the anchor with
the real boot ROM (dmg-sim measurement):

**CH1** — the load-bearing block for the `init_pos` test family:

| Item | Value |
|------|-------|
| Duty step counter | **2** |
| Period divider | **0x7F9** (counting toward 0x7FF — six ticks to the next overflow) |
| Prescaler | 1 |
| Reload value (`acc_d` = {NR14[2:0], NR13}) | 0x7C1 |
| Volume | 0 — **but the DAC is ON** (NR12 = 0xF3 retained) and the running latch is SET |
| Envelope | saturated-stopped (`ch1_eg_stop` = 1); counter holds ~pace = 0b100 |
| Sweep | inactive (NR10 never written; pace 0 holds the fire latch in reset); the sweep counter cells are no-reset — power-on residue |

A skip-boot emulator that zeroes CH1's internals diverges from hardware on
the whole retrigger test family: the first cartridge trigger reloads the
divider but preserves the duty counter and the running latch, so
audibility depends on duty step 2 advancing (or not) during the test's
padding.

**CH2/CH3/CH4** — untouched: counters zero (a simulation convention; the
no-reset divider cells are undefined on silicon but unreachable before a
trigger), DACs off, disable latches set. CH4's LFSR holds its 0x7FFF reset
pattern. **Wave RAM has no reset** — the all-zero reading is a simulator
initialisation, *not* a hardware claim; real post-boot wave RAM is
implementation-defined.

```admonish warning "Pitfall: the frame-sequencer seed"
The single most commonly mis-modelled value: the step counter is the
`apu_reset`-reset ripple, measured directly at handoff as
`(CARU, BYLU, JYNA) = (0, 1, 0)` → **Δ = 3** in the re-lock formula
([APU clocks](apu-clocks.md)). **Do not derive the step from `reg_div16`
bits** — that places envelope/length/sweep ticks ~3 steps off the real
phase, which is exactly the failure mode of the phase-sensitive envelope
tests ([sweep and envelope](apu-sweep-envelope.md)). The ripple's advance
rate is divider-locked; only its seed carries the boot offset.
```
