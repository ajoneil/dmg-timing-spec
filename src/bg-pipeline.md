# Mode 3: The BG Pipeline

Mode 3 is the pixel-transfer phase — typically 173 dots plus fine-scroll,
sprite, and window penalties. Two machines run in parallel: the tile fetcher
pulls BG/window tile data from VRAM on an 8-dot cadence, and the pixel clock
(CLKPIPE / SACU) shifts pixels out of the BG shift registers toward the LCD.
This chapter covers the BG side across four pages; the
[sprite pipeline](sprite-pipeline.md), [window control](window.md), and
[LCD output](lcd-output.md) have their own chapters.

```admonish abstract "At a glance"
- Mode 3 baseline is **173.481 dots** = a 173.045-dot AVAP→WODU span (a
  7.026-dot startup cascade, then 167 pixel-clock edges) + a 0.436-dot
  WODU→XYMU handoff tail — before fine-scroll, sprite, and window
  penalties.
- **167 SACU edges per Mode 3, invariant with SCX** — fine scroll is clock
  suppression (1.000 dot per `SCX & 7` step), never push-and-discard.
- Tile fetches run an **8-dot cadence**: 6 counting dots plus a 2-dot
  drain; the fetch-counter reset NYXU fires 22 times per scanline at
  SCX=0, 23 at SCX&7 ∈ {1..7}.
- The pixel clock is the **deepest routinely-toggling signal** in the PPU
  (63.8 ge) — which is why shifter data is always stable before the edge
  that shifts it.
- LCDC.3/4/6 reach VRAM addressing through **level-sensitive paths**;
  TILE_SEL is sampled twice per fetch, enabling hybrid fetches.
```

## What's where

| Page | Covers |
|------|--------|
| [The tile fetcher](bg-pipeline/fetcher.md) | The 3-bit fetch counter, the 8-dot tile cycle and drain cascade, VRAM addressing, and the live LCDC.3/4/6 fetch-address sampling |
| [The pixel clock](bg-pipeline/pixel-clock.md) | The PX counter, terminal count 167, the CLKPIPE/SACU gating tree, and the four halt conditions |
| [Startup and fine scroll](bg-pipeline/startup-fine-scroll.md) | The AVAP→first-SACU cascade, the fine-scroll counter and discard, late SCX writes, the tail-of-Mode-3 drain, and the invariance measurements |
| [The BG shift registers](bg-pipeline/shifters.md) | The shifter planes, temp latches, parallel load via NYXU, and the two startup loads |
