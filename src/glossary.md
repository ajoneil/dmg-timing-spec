# Appendix D: Glossary

**ALET / MYVO** — the PPU's main 4 MHz clock and its complement; the
canonical per-dot edge vocabulary. "MYVO rising" = "ALET falling".

**ATAL half-cycle** — half a dot: the interval between consecutive
master-clock edges.

**AVAP** — the scan-complete pulse that starts Mode 3; fires at dot 80 of
each rendering scanline.

**CLK9** — the CPU-side M-cycle clock: one rising edge per M-cycle,
physically the dot-0 `ck1_ck2`↑ master-clock edge (+8,440 ps of buffer;
+2,983 ps after the derived ALET↑, which it is sometimes quoted against).

**CLKPIPE / SACU** — the pixel pump: the gated, deeply-buffered clock that
shifts the pixel pipeline once per dot during active rendering.

**CUPA** — the shared PPU register write strobe; 1.493 dots wide, spanning
T3–T4 of the write M-cycle. The APU's `apu_wr` is its sibling.

**data phase** — the second half of a CPU M-cycle (dots ≈ 2–4), during
which the bus carries data; `data_phase_n` defines the IF-latch
transparency window and the CPU's read-latch point (tail of T4, +3.995 dots).

**dot** — one full master-clock cycle (rise + fall); synonymous with
T-cycle. ≈ 238.4 ns on hardware; 244,000 ps in the simulation timebase.

**drain (tile boundary)** — the two-dot interval after a tile fetch
completes in which the NYKA→PORY→RENE/RYFA cascade empties, ending in the
SEKO pulse that resets the fetcher for the next tile.

**fetch overlap** — the SM83's co-issue of the next opcode fetch in an
instruction's final M-cycle when the bus is free; instructions whose
terminal cycle is bus-busy get a dedicated post-body fetch cycle (m6/m7).

**frame sequencer** — the APU's 512/256/128/64 Hz tick source: a 512 Hz
tap of the timer divider plus a 3-bit ripple counter with its own
(`apu_reset`) reset domain.

**ge (gate equivalent)** — the unit of combinational depth in the static
netlist analysis; the master clock is the conventional origin.

**LINE_END** — the LX=113 condition (SANU) and its captured pulse (RUTU)
that terminate each 456-dot scanline.

**M-cycle** — one CPU machine cycle = 4 dots.

**Mode 3 baseline** — 173.481 dots = a 173.045-dot AVAP→WODU span (a
7.026-dot startup cascade, then 167 pixel-clock edges) + the 0.436-dot
WODU→XYMU VOGA tail, before fine-scroll/sprite/window penalties.

**prelude (Mode 3)** — the AVAP→first-SACU startup window (7.026 dots at
SCX=0) during which the first tile is fetched and fine scroll is paid.

**T-cycle** — see *dot*.

**VID_RST** — the LCDC.7-driven reset domain (XODO/XAPO and branches)
that holds most of the PPU cleared while the LCD is off.

**WODU** — the Mode 0 condition (the H-Blank condition): pixel counter at
terminal count with no
sprite match; drives both the Mode 3→0 transition (via VOGA) and the STAT
interrupt (combinationally, 0.425 dots earlier).

**XYMU** — the rendering-mode latch; **0 during Mode 3** ("not
rendering" polarity), 23 consumers.

**x-window (bus settling)** — the post-transition interval during which a
tri-state-driven CPU bus bit has not resolved; reads in the trigger
M-cycle of a mode transition latch the pre-transition value through it.
