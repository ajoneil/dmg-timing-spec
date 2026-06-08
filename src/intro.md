# Introduction

This book describes the timing behaviour of the original Game Boy's SoC — the
**DMG-CPU B** — at the gate and signal level: the clock tree, the PPU's
rendering pipeline and mode machinery, the CPU's view of every register
boundary, the timer and interrupt-dispatch circuits, and the APU's
per-channel counters.

It exists because no current documentation reaches these circuits at this
layer:

- [gb-ctr](https://gekkio.fi/files/gb-docs/gbctr.pdf) — excellent where it
  reaches, but its coverage is not yet thorough at the time of writing;
- [Pan Docs](https://gbdev.io/pandocs/) — the console at register
  granularity;
- pixel-FIFO write-ups — the PPU as an algorithm.

None of them can answer: *on which clock edge does Mode 3 actually end?
What does the CPU read from STAT during the M-cycle that straddles that
edge? Which edge sets IF when TIMA wraps?* Emulator authors
chasing the last mile of accuracy need answers at exactly that resolution —
which flip-flop captures which signal on which edge — and that is the layer
this book documents, for every subsystem on the die.

Every claim here is tied to a named gate from the DMG-CPU B netlist. The names
(WODU, XYMU, CUPA, MOBA, CARU…) come from the reverse-engineering effort, not
the die's designers — but they are stable, globally unique, and shared by the
netlist and the simulation: anyone with either can look a cell up and check
the claim independently. The dynamic evidence is gate-level *simulation*,
corroborated by test ROMs whose expected results are hardware-verified — no
figure here comes from direct electrical measurement of a real unit.
[Sources and Methodology](methodology.md) draws that boundary precisely.
[Appendix A](concordance.md) is the lookup table — every signal named in this book,
its role, its connections, and where it is described.

## What this book is not

- **Not a general Game Boy reference.** The console's general hardware and
  programming model are covered in detail by
  [gb-ctr](https://gekkio.fi/files/gb-docs/gbctr.pdf); this book does not duplicate
  them. Where a subsystem's behavioural surface is already well documented (the
  timer is the clearest example), the relevant chapter summarises it briefly, cites
  the existing reference, and spends its pages on the gate-level structure
  underneath.
- **Not derived from emulators.** No emulator's implementation was used as a source
  of hardware truth; see [Sources and Methodology](methodology.md).
- **DMG-CPU B only.** Later revisions and the Game Boy Color have no public
  netlist; their deltas are out of scope.

## The shape of the book

- **Foundations** — the clock tree and the counters and latches that give every
  other chapter its vocabulary: dots, ALET/MYVO edges, LX/LY, the XYMU
  rendering-mode latch.
- **CPU–PPU Interface** — how CPU reads and writes actually reach PPU registers,
  OAM, and VRAM: the combinational read path, the CUPA write strobe, and OAM DMA's
  bus arbitration.
- **The Rendering Pipeline** — Mode 2 OAM scan, the Mode 3 BG/sprite/window
  machinery, LCD output, mode transitions, STAT interrupts, full scanline and frame
  timing, and what the CPU observes at every mode boundary.
- **CPU-Side Subsystems** — the timer, interrupt dispatch, HALT/EI semantics, and
  the IF register, at the same edge-level resolution.
- **Audio Processing Unit** — the APU's clock tree, frame sequencer, and
  per-channel counter cells.
- **System State** — the complete hardware state at the moment the boot ROM hands
  control to the cartridge.

The PPU chapters follow one pipeline — the path every rendering scanline runs
through Mode 3:

> **AVAP → NYXU → BG fetch counter → SACU → PX counter → WODU → VOGA → WEGO → XYMU**

OAM scan completion (AVAP) starts rendering, the SACU pixel clock pumps pixels
through the PX counter, and the terminal-count condition (WODU) propagates through
a one-DFF pipeline (VOGA) to set the "not rendering" latch (XYMU). The STAT
interrupt cascade hangs off WODU and the LY-comparison match; the CPU reaches PPU
registers through a combinational read path and the CUPA write strobe. Every stage
of that path has a chapter.

The CPU-side and APU chapters need no shared roadmap — each subsystem is
self-contained: the timer is a ripple counter and a two-M-cycle reload
sequence, dispatch is a trigger/capture pair against the IF latches, and
each APU channel is a divider plus a handful of tick counters.

## Reading paths

Few readers need this book cover to cover. Three curated routes:

**The emulator-implementor path** — building or improving a cycle-accurate
core. Read [Notation and Conventions](how-to-read.md), then
[The Clock Tree](clock-tree.md) and
[Rendering Mode Control](mode-control.md) for vocabulary, then go straight
to the subsystem you are implementing. The "For implementors" callouts and
[Appendix B's constants](constants.md) are written for you; finish with
[Post-Boot State](post-boot.md) if you skip the boot ROM.

**The boundary-behaviour path** — chasing a failing timing test. Start at
[CPU-Visible Mode Boundaries](cpu-visible-boundaries.md) for the
PRE/POST framework, then the boundary's owning chapter
([Mode transitions](mode-transitions.md),
[STAT interrupts](stat-interrupts.md),
[Interrupt dispatch](interrupt-dispatch.md), or
[HALT and EI](halt-ei.md)), then [Races](races.md) if the divergence is
sub-dot.

**The APU path** — self-contained: [APU clock tree and frame
sequencer](apu-clocks.md) first (the re-lock rule and the Δ = 3 boot seed
underpin everything), then the channel chapters in any order, then the
APU rows of [Post-Boot State](post-boot.md).

If you are new to gate-level material, read
[Notation and Conventions](how-to-read.md) first — it is short, and the rest of
the book assumes it.
