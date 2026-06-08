# Scanline and Frame Timing

This chapter frames the per-subsystem behaviour at scanline and frame
level: one scanline's dot structure, and the physical ordering of events
inside a single dot. The complete power-up story — the shortened first
scanline, the startup transient at the pins, and the LCD-on → first-WODU
composition — has its own pages.

```admonish abstract "At a glance"
- One scanline = **456 dots = 114 M-cycles**; the Mode 2 / Mode 3 / Mode 0
  split is set by the pixel pipeline, not a fixed schedule.
- Inside a dot, events unfold by **propagation depth**: registered outputs
  first (0–8 ge), ALET captures mid (8–22), CLKPIPE last (63.8).
```

## The scanline

One scanline = 456 dots = 114 M-cycles (LX counts 0–113):

| Phase | Dots | Duration | What happens |
|-------|------|----------|--------------|
| Mode 2 (OAM scan) | 0–79 | 80 dots | 40 OAM entries, 2 dots each; sprite store builds |
| Mode 3 (pixel transfer) | 80 – ~253.5+ | 173.481 + (SCX & 7) + penalties | Fetch and shift; CLKPIPE drives output |
| Mode 0 (H-Blank) | remainder | 456 − the rest | CPU may access VRAM/OAM |

The Mode 3 budget, consolidated (all developed in their own chapters):

- **Baseline 173.481 dots**: 7.026 startup + 166.019 dots through the pipe
  (167 SACU edges, 166 steps after the first) + the 0.436-dot VOGA tail.
- **Fine scroll: exactly 1 dot per `SCX & 7` step**, applied at startup via
  ROXY; the SACU count never changes.
- **Sprites: 6-dot core + 0–5 dots alignment per sprite.** Single sprite at
  SPRITEX 8/9 → +11/+10 dots; four sprites step ~4 dots per +1 SPRITEX over
  [8, 13]; tile-boundary anomalies at {14, 15}; stacked same-X sprites add
  exactly 6 dots each ([sprite pipeline](sprite-pipeline/penalty-model.md)).
- **Window: +6.000 dots, invariant across WX** — one fetch-pipeline restart
  ([window control](window.md)).
- The 21 TEVO tile boundaries are internal to the fetcher and cost nothing.

## Inside one dot

By propagation depth, each master-clock edge unfolds in three bands:

- **Early (0–8 ge)** — registered outputs and the VRAM bus settle.
- **Mid (8–22 ge)** — ALET arrives and ALET-clocked DFFs capture;
  MYVO/LEBO arrive on the opposite edge.
- **Late (22–64 ge)** — sprite matches, FEPO, WODU, VYBO, and finally
  CLKPIPE at 63.8 ge.

The physical ordering on silicon:

**ALET rising**: the ALET-clocked DFFs capture (NYKA, LYZU, PYGO, RENE,
DOBA, NOPA, VOGA); the sprite fetch clock SABE rises; the NOR/NAND latches
(XYMU, ROXY, POKY, LONY, BESU) respond combinationally; addresses and
enables settle.

**ALET falling** (= MYVO rising): PORY captures NYKA's fresh output; the BG
fetch clock LEBO rises and the LAXU→MESU→NYVA ripple advances; CUPA is high
in this window once per M-cycle, making the CPU-written register latches
transparent.

**Delayed, same falling edge**: SACU's rising edge *is* the ALET falling
edge, arriving 41.6 ge later through VYBO/TYFA/SEGU — the delay is what
guarantees DFF captures and counter advances settle before the pixel pipe
shifts. CPU register *reads* are not ordered events at all — the read path
is continuously-enabled combinational logic, sampled on the CPU's own
schedule.

At the 1 MHz tier, TALU/SONO distribute LX counting and LINE_END capture
across M-cycle boundaries ([line counters](line-counters.md)).

## The LCD-on story

The power-up sequence has two pages:

| Page | Covers |
|------|--------|
| [LCD-on power-up](scanline-frame-timing/lcd-on.md) | The 454-dot first scanline, the pin-level startup transient, and the reset-domain taxonomy of every PPU DFF |
| [LCD-on → first WODU](scanline-frame-timing/lcd-on-to-wodu.md) | The end-to-end path from the LCD-enabling write to the first H-Blank, and what the CPU observes |
