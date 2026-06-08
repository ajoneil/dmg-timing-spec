# Sources and Methodology

This book was written with substantial AI assistance, under direction and with
the verification discipline described in this chapter. The full account is in
the [project's README](https://github.com/ajoneil/dmg-timing-spec).

Everything in this book derives from three primary sources, used in this order:

1. **The DMG-CPU B netlist** —
   [msinger/dmg-schematics](https://github.com/msinger/dmg-schematics), a
   transistor-level reverse engineering of the DMG-CPU B die. The netlist supplies
   the ground truth this book is written in: cell names, cell types, and
   connectivity. When this book says "WODU = AND2(XENA, XANO)", that is a netlist
   fact, checkable by opening the schematics.

2. **Gate-level simulation** —
   [msinger/dmg-sim](https://github.com/msinger/dmg-sim), a SystemVerilog model
   generated from that netlist and run under Icarus Verilog — the same logic
   as the silicon, cell by cell, with modelled propagation delays (tunable
   parameters, not values extracted from silicon). Every dynamic timing claim
   in this book was measured from its waveform captures. Test ROMs run under
   a minimal **quickboot** stub that reproduces the post-boot state; the real
   boot ROM was used where boot alignment or residual boot-time state
   matters, and to validate quickboot itself.

3. **Static propagation-delay analysis** —
   [gb-propagation-delay-analysis](https://github.com/ajoneil/gb-propagation-delay-analysis),
   graph analysis over the netlist
   (4,102 nodes, 9,458 edges): combinational depth measured in **gate equivalents
   (ge)**, race-pair identification between converging paths, and critical-path
   inventories. This is the source for the
   [races chapter](races.md) and for depth figures quoted elsewhere.

Behavioural documentation — [gb-ctr](https://gekkio.fi/files/gb-docs/gbctr.pdf),
[Pan Docs](https://gbdev.io/pandocs/), TCAGBD, the Mealybug Tearoom PPU notes —
is used for cross-checking and for high-level framing, and is cited where it
carries a section's behavioural summary. **No emulator implementation was used as
a source of hardware truth.** Hardware test ROMs with hardware-verified expected
results corroborate behavioural conclusions but never ground a gate-level claim.
Annotations are trusted but not infallible — where a ROM's expected
value disagreed with the simulation, the ROM was re-run on real hardware
and excluded if the expectation did not hold there. Every cited ROM was
measured from a verified local build.

## How measurements were made

The measurement workflow behind every "dmg-sim measurement" citation:

1. A small test ROM establishes a precisely-timed scenario — for example, a
   HALT/NOP loop synchronised to the H-Blank interrupt with SCX=0. Citations
   name the scenario and its suite (e.g. gbmicrotest `int_hblank_nops_scx0`),
   or mark it purpose-built, so a measurement can be reproduced by
   reconstructing the scenario.
2. dmg-sim runs the ROM and dumps an FST waveform of the full cell-level
   state (1 ps timescale).
3. Named-signal transitions are read from the waveform with picosecond
   timestamps, and converted to dots using the simulation's measured dot period.
4. Single observations are never trusted: values are checked for invariance
   across consecutive scanlines, across scenario variants (different instruction
   preludes, different SCX values), and against the static netlist structure.

## How to read the numbers

Two kinds of numeric values appear in this book, and they carry different weight:

- **Dot-denominated figures** ("Mode 3 lasts 173.481 dots", "the sprite penalty is
  exactly 6 dots") are discrete observations: counts of clock edges between
  two events, with no precision limit. Given comfortable timing margins, the
  count is set by the clocking structure, not the delay values; the test-ROM
  corpus corroborates the counts at the resolution it observes (typically
  the M-cycle). One dot ≈ 238.4 ns on hardware.
- **Picosecond figures** ("CATU's Q output moves 988 ps after its clock
  edge") are model propagation values — the simulation's dot period is
  244,000 ps against the real ≈238,418 ps. Read them as **edge-ordering and
  phase evidence**, not as nanosecond claims about a physical chip.

Fractional-dot figures such as "WODU→XYMU = 0.436 dots" read against the
half-dot clock grid. Events in these chains sit at small propagation offsets
from clock edges, so an interval's fraction reveals which edge its endpoint
belongs to: VOGA captures WODU on the same-dot ALET rising edge, 0.435 dots
after WODU↑, and the WEGO→XYMU gate adds the final 0.001 — so XYMU's set
lands just after that ALET rising edge. The prose always names the edges; the
fraction is how the waveform shows them.

**The evidence boundary.** Every dynamic measurement here is a *simulation*
measurement. Hardware enters only as test-ROM pass/fail expectations
validated on real units and as reference photographs — never as direct
electrical measurement of a real DMG. Real units also vary (process,
voltage, temperature), so "zero variance" describes the deterministic
model, and a small-margin ps ordering is the model's prediction of the
likely silicon ordering, not a guarantee. Where unit variation could
plausibly flip an ordering, the text says so.

**Calibration pin.** The dominant delay-tuning constant is `kR_nmos_ref`
in dmg-sim's `timing-default.sv`. Every ps and fractional-dot figure in
this book is stamped to **`kR_nmos_ref = 3.6e4`** (upstream commit
`e0c8774`, the current calibration as of June 2026). Re-tuning that
constant rescales the ps figures near-uniformly and leaves edge identities
in place — verified across a 3.6× historical re-tuning, with one
margin-sensitive exception flagged where it appears
([pixel clock](bg-pipeline/pixel-clock.md)). If a build disagrees with
this book's ps values by a uniform factor, check that constant first.
Integer dot counts are calibration-independent: dot figures are expressed
in the simulation's own dot period, so known edge counts (the 4-dot TALU
period, the 6-dot sprite fetch) read as exact integers.

## Verification

Every numeric figure was re-traced to its measurement evidence during
preparation for publication and checked for consistency against every other
place it appears; the gate-level measurement always grounds the claim.
Nothing unverified is stated silently: residual unknowns are tagged
**Open question** inline and collected in [Appendix C](open-questions.md).

Some signals are documented structurally but lack a derived semantic role;
their [concordance](concordance.md) entries leave the role cell blank rather
than invent one.
