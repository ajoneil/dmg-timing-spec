# Mode 2: OAM Scan

```admonish abstract "In this part"
The Rendering Pipeline chapters walk a scanline in machine order: the OAM
scan (this chapter), the BG fetch and pixel output
([BG pipeline](bg-pipeline.md)), sprite fetch and mixing
([sprite pipeline](sprite-pipeline.md)), [window control](window.md), the
[LCD interface](lcd-output.md), then the cross-cutting views —
[mode transitions](mode-transitions.md),
[STAT interrupts](stat-interrupts.md),
[scanline/frame timing](scanline-frame-timing.md),
[CPU-visible boundaries](cpu-visible-boundaries.md), and
[races](races.md).
```

Mode 2 is the first 80 dots (20 M-cycles) of each non-VBlank scanline. The PPU
scans the 40 OAM entries (2 dots each), comparing each sprite's Y position
against LY and capturing the X position of matching sprites into a 10-slot
sprite store for Mode 3 rendering. At scan completion, AVAP rises and triggers
the Mode 2→3 transition. This chapter follows the signal chain: scan counter →
scan-active latch → Y comparator → sprite store → AVAP generation.

```admonish abstract "At a glance"
- Mode 2 decomposes exactly: 2 dots (CATU↑ → first tick) + 76 (counter
  1→39) + 2 (FETO → BYBA capture) = **80 dots**; boundary-to-AVAP is 81.
- The Y compare is **combinational** (an 8-stage carry chain) — no DFF
  captures the match; it gates the per-slot store write-enables directly.
- The store's write enable is the PPU's **deepest per-line path**
  (71.2 ge) while its data arrives at depth 0 — the source of a family of
  races.
- AVAP is a **half-dot pulse** from the BYBA/DOBA edge detector: BYBA
  captures scan-done on XUPY (ALET falling), DOBA catches up on ALET
  rising.
- On the first scanline after LCD-on, **BESU never sets** — no scan, no Mode 2
  STAT bit, no sprite store.
```

## What's where

| Page | Covers |
|------|--------|
| [The counter and BESU](oam-scan/counter.md) | The 6-bit ripple counter, its clocking and reset, the scan-active latch and its XUPY-phased copy |
| [The Y comparator](oam-scan/y-comparator.md) | The 8-stage carry chain, the WOTA decoder, LCDC.2 masking, and capture durability |
| [The sprite store](oam-scan/sprite-store.md) | 10-of-40 priority, the slot counter and write enables, reset domains, and the LCDC.1 trigger branch |
| [AVAP and the 80 dots](oam-scan/avap.md) | The BYBA/DOBA edge detector, the measured counter-39 → AVAP cascade, and Mode 2's exact decomposition |
