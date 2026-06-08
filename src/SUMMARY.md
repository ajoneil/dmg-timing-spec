# Summary

[Introduction](intro.md)
[Sources and Methodology](methodology.md)
[Notation and Conventions](how-to-read.md)

# Foundations

- [The Clock Tree](clock-tree.md)
- [Line Counters](line-counters.md)
- [Rendering Mode Control](mode-control.md)

# CPU–PPU Interface

- [Registers](registers.md)
  - [Reads](registers/reads.md)
  - [Writes](registers/writes.md)
  - [Palette Latches](registers/palette-latches.md)
  - [LCDC Structure](registers/lcdc.md)
- [OAM and VRAM Access](oam-vram-access.md)
  - [OAM Writes](oam-vram-access/oam-writes.md)
  - [VRAM Writes](oam-vram-access/vram-writes.md)
  - [Read Locks](oam-vram-access/read-locks.md)
  - [Lock Boundaries](oam-vram-access/lock-boundaries.md)
  - [OAM Corruption](oam-vram-access/corruption.md)
- [OAM DMA](dma.md)
  - [The OAM Bus](dma/oam-bus.md)
  - [Engage, Release, and HALT](dma/boundaries.md)
  - [The Source Bus](dma/source-bus.md)

# The Rendering Pipeline

- [Mode 2: OAM Scan](oam-scan.md)
  - [The Counter and BESU](oam-scan/counter.md)
  - [The Y Comparator](oam-scan/y-comparator.md)
  - [The Sprite Store](oam-scan/sprite-store.md)
  - [AVAP and the 80 Dots](oam-scan/avap.md)
- [Mode 3: The BG Pipeline](bg-pipeline.md)
  - [The Tile Fetcher](bg-pipeline/fetcher.md)
  - [The Pixel Clock](bg-pipeline/pixel-clock.md)
  - [Startup and Fine Scroll](bg-pipeline/startup-fine-scroll.md)
  - [The BG Shift Registers](bg-pipeline/shifters.md)
- [Mode 3: The Sprite Pipeline](sprite-pipeline.md)
  - [Sprite Storage and X Matching](sprite-pipeline/storage-matching.md)
  - [The Fetch State Machine](sprite-pipeline/fetch-machine.md)
  - [The Penalty Model](sprite-pipeline/penalty-model.md)
  - [LCDC.1 and LCDC.2 Paths](sprite-pipeline/lcdc-paths.md)
- [Window Control](window.md)
  - [Window Edge Cases](window/edge-cases.md)
- [LCD Output](lcd-output.md)
- [Mode Transitions](mode-transitions.md)
- [STAT Interrupts](stat-interrupts.md)
- [Scanline and Frame Timing](scanline-frame-timing.md)
  - [LCD-on Power-up](scanline-frame-timing/lcd-on.md)
  - [LCD-on → First WODU](scanline-frame-timing/lcd-on-to-wodu.md)
- [CPU-Visible Mode Boundaries](cpu-visible-boundaries.md)
- [Races](races.md)

# CPU-Side Subsystems

- [The Timer](timer.md)
- [Interrupt Dispatch](interrupt-dispatch.md)
- [HALT and EI](halt-ei.md)
- [The IF Register](if-register.md)

# Audio Processing Unit

- [APU Clock Tree and Frame Sequencer](apu-clocks.md)
- [CH1/CH2: Pulse Channels](apu-ch1-ch2.md)
- [CH3: Wave Channel](apu-ch3.md)
- [Sweep and Envelope](apu-sweep-envelope.md)
- [Length Counters and Power Cycling](apu-length-power.md)

# System State

- [Post-Boot State](post-boot.md)

# Appendices

- [Appendix A: Signal Concordance](concordance.md)
- [Appendix B: Timing Constants](constants.md)
- [Appendix C: Open Questions](open-questions.md)
- [Appendix D: Glossary](glossary.md)
- [Appendix E: Sources](bibliography.md)
