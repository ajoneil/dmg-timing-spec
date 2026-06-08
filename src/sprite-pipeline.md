# Mode 3: The Sprite Pipeline

Sprite rendering interleaves with BG fetching: when a sprite's X position
matches the pixel counter, a trigger chain freezes the pixel clock while the
sprite fetcher reads OAM attributes and VRAM tile data, then merges the
fetched pixels into dedicated shift registers with hardware-level priority.
This chapter covers that machinery across four pages.

```admonish abstract "At a glance"
- The fetch core costs **6 dots** (the TEKY↑ → TAKA↓ freeze — 6.010 dots at
  gate resolution); the documented
  "6–11 dots per sprite" is this fixed core plus 0–5 dots of BG-fetcher
  alignment overhead.
- Sprite-to-sprite priority is implemented **in the shift register**: the
  parallel load is transparency-gated per stage, so the first-fetched
  (lowest-OAM-index) sprite's opaque pixels survive.
- N sprites stacked at one X cost the first sprite's penalty plus
  **(N − 1) × 6.000 dots** — the inter-fetch gap is combinational
  (gate delays).
- The CLKPIPE freeze is **FEPO-driven, not TAKA-driven**: a mid-fetch
  `LCDC.1 ← 0` write resumes the pixel clock early while the fetch machine
  runs on.
- The fetcher has no dedicated OAM DFFs — it **reuses Mode 2's capture
  chain**; under DMA it captures the byte-pair at DMA's current address.
```

## What's where

| Page | Covers |
|------|--------|
| [Sprite storage and X matching](sprite-pipeline/storage-matching.md) | The sprite shift registers, the transparency-gated load, the per-slot store and X comparators, FEPO, and the attribute pipes |
| [The fetch state machine](sprite-pipeline/fetch-machine.md) | The TEKY→SOBU→RYCE→TAKA trigger chain, the fetch counter, the BG-cascade freeze, OAM byte capture (including under DMA), and TAKA's line-end behaviour |
| [The penalty model](sprite-pipeline/penalty-model.md) | Measured penalties by SPRITEX, the (K + SCX) mod 8 plateau, stacked-sprite composition, the K=167 right edge, and sprites in the prelude |
| [LCDC.1 and LCDC.2 paths](sprite-pipeline/lcdc-paths.md) | OBJ_EN's trigger-vs-output split, mid-fetch disable writes, the OFF single-pixel transient, and OBJ_SIZE's live VRAM row-bit path |
