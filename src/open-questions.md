# Appendix C: Open Questions

Everything in this book that is *not* settled by primary-source evidence is
tagged inline as an **Open question** and collected here. Each entry names
the residual unknown and what kind of measurement would close it. The
presence of this list is deliberate: a hardware reference that hides its
edges invites silent over-trust.

## Below-netlist LCD-interface overlays (a family)

Three measured behaviours live below the die model's scope — the netlist
reproduces none of them at any timing calibration, yet hardware reference
photographs pin all three:

1. **The BGP OR-overlap** ([palette latches](registers/palette-latches.md)) — a
   second-or-later mid-Mode-3 BGP write's first emitted column reads
   `new | old`, with a BESU-reset recovery state and a
   visible-emission-engagement clause.
2. **The LCDC.1 OFF single-pixel transient**
   ([sprite pipeline](sprite-pipeline/lcdc-paths.md)) — OFF-direction only, one
   shade-3 column, with an empirically-scoped per-scanline firing set that
   netlist state cannot distinguish.
3. **The LCDC.0 one-column overlay** ([LCD output](lcd-output.md)) —
   bidirectional, every write, first column emits with the old enable
   state.

The candidate mechanisms (LCD-glass column-driver sample-and-hold vs
pad-driver residue) predict identical column-level behaviour; the on-die
pad drivers are measured stateless, pushing the state off-die. **Closing
this needs hardware-level capture** (Slowpeek-class probing of LD0/LD1/CP
at the flex cable) across writes spanning H-Blank. The behavioural rules
are sufficient for emulation regardless.

## A hardware-annotation discrepancy

The gambatte `m2enable/late_m1disable_ly0_3` ROM expects IF[1]=0 at a read the simulated
Case-4 SUKO glitch places at 1; an equivalent-cascade ROM with the clear
on the other side is netlist-consistent. **Margin-sensitive**: the
disputed dip is 1,524 ps wide and the narrowest dip hardware demonstrably
fires is 1,802 ps — a real unit could sit between the two, making the
annotation and the simulation both correct on different silicon
([STAT interrupts](stat-interrupts.md)). Needs a hardware measurement on
multiple units.

## Smaller residuals

- **The community first-word formulas**: the suppressed-precharge mechanism
  and the clean row-copy (single-access) and multi-row (two-read) cases are
  measured ([OAM corruption](oam-vram-access/corruption.md)); the community's
  alignment-dependent first-word AND/OR mixes are analog bitline resolution
  below the digital model's reach — a digital SRAM cell cannot partially
  retain its old value, so the model copies cleanly. Like the LCD-interface
  overlays above, the behaviour is hardware-established (the blargg `oam_bug`
  suite); the community formulas remain its reference.
- **VRAM-side write straddle**: a CPU VRAM write whose strobe spans a
  Mode 3 entry/exit boundary tri-states the address pads and write strobe
  mid-cycle, so the off-chip VRAM SRAM sees a truncated write cycle. What
  that partial cycle does to the SRAM is off-die, outside the netlist's
  scope ([lock boundaries](oam-vram-access/lock-boundaries.md)).
- **HSYNC at the glass**: the ST pulse shape is measured
  ([LCD output](lcd-output.md)); only the interpretation at the glass —
  pulse-width expectations, row-driver response — still rests on community
  pin-role references. Closing it needs a hardware-level capture of the ST
  pin and the LCD's row-driver response (Slowpeek-class flex-cable probing),
  as for the other glass-side behaviours.
- **HALT-vs-running handler write ordering**: the one-M-cycle M2/m7
  ordering difference is measured, uniform, and modelled correctly. Two
  things stay open: its cause inside the SM83 M-cycle sequencer (a
  cell-level reconstruction not yet attempted), and one reference image
  whose column shift runs the opposite way, which needs real-hardware
  confirmation ([HALT and EI](halt-ei.md)).
