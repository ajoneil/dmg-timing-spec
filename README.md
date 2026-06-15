# DMG Timing Specification

A gate- and signal-level description of the timing behaviour of the original Game Boy's SoC,
the **DMG-CPU B**: the clock tree, the PPU's rendering pipeline and mode machinery, the CPU's
view of every register boundary, the timer and interrupt-dispatch circuits, and the APU's
per-channel counters.

**Read it here:** https://ajoneil.github.io/dmg-timing-spec/

Every claim is tied to a named gate from the public DMG-CPU B netlist, so it can be looked up
and checked independently.

## What this is

This book works at the gate and signal level: which flip-flop captures which signal on which
clock edge. Existing references describe the console above that layer: [gb-ctr] in the most
detail, [Pan Docs] at register granularity, pixel-FIFO write-ups as an algorithm. This is the
level beneath, where questions like *on which clock edge does Mode 3 actually end, and what
does the CPU read from STAT during the M-cycle that straddles it?* get answered, across each
subsystem the book covers.

It is not a general Game Boy reference and doesn't duplicate what [gb-ctr] already covers.
DMG-CPU B only: later revisions and the Game Boy Color have no public netlist and are out of
scope.

## Why it exists

The gate-level understanding here was developed while building [Missingno], a high-accuracy
DMG emulator. Getting a core that accurate meant working out exactly which circuit produces
which behaviour on which edge, and that work was polished and refined into this spec.

## How this book was written

This book was written with substantial AI assistance. I directed the work: methodology,
standards, scope, and verification discipline. Claude (Opus 4.6 through 4.8), working mostly
through a Claude Code harness, drafted the prose and ran the research and measurement.

The methodology is netlist-authoritative throughout. Every gate name is a cell in
[the public netlist][dmg-schematics], every dynamic figure is reproducible from
[msinger's public simulation][dmg-sim] at the calibration constant the book pins, and the
combinational-depth and race figures come from [static analysis of that netlist][propagation].
No emulator was used as a source of hardware truth, and behaviour not settled by
primary-source evidence is marked as an open question rather than guessed (Appendix C). Any
claim can be checked against these sources, all linked below.

## How to read it

Few readers need the whole book. The introduction lays out three routes: an
emulator-implementor path, a boundary-behaviour path for a failing timing test, and a
self-contained APU path. New to gate-level material? Read *Notation and Conventions* first;
the rest assumes it.

## Corrections

Corrections and suggestions are welcome. Please open an issue. Every claim is tied to a named
gate, so a correction can point straight at the cell or waveform it concerns, which makes it
easy to pin down and fix.

## Acknowledgements

This book builds on a lot of other people's reverse engineering. Without it, none of this
would be possible.

- **msinger and rgalland**: the DMG-CPU B die reverse engineering this book stands on. The
  [netlist][dmg-schematics] supplies its ground truth, and the [gate-level
  simulation][dmg-sim] produces every dynamic measurement.
- **Furrtek**: the die photography and tracing the gate-level reverse engineering descends
  from.
- **gekkio**: [gb-ctr], the behavioural reference this book cross-checks against throughout,
  and the Mooneye test suite.
- **The test-ROM authors** whose work the book drew on for scenarios and cross-checks:
  aappleby (gbmicrotest), LIJI32 (SameSuite), c-sp (AGE test ROMs), blargg, Hacktix, sinamas (gambatte hardware-test suite), wilbertpol (Mooneye fork), and mattcurrie (Mealybug Tearoom).

The complete source list, with how each was used, is in Appendix E.

Any errors are my own.

## License

This book is dedicated to the public domain under
[CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/).

<!-- links -->
[gb-ctr]: https://gekkio.fi/files/gb-docs/gbctr.pdf
[Pan Docs]: https://gbdev.io/pandocs/
[Missingno]: https://github.com/ajoneil/missingno
[dmg-schematics]: https://github.com/msinger/dmg-schematics
[dmg-sim]: https://github.com/msinger/dmg-sim
[propagation]: https://github.com/ajoneil/gb-propagation-delay-analysis
