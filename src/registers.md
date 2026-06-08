# Registers

```admonish abstract "In this part"
Three chapters cover the CPU's view of the PPU: this one (the register
file — how reads and writes reach it, the palette latches, and LCDC's
consumer fan-out), [OAM and VRAM access](oam-vram-access.md) (the lock
logic), and [OAM DMA](dma.md) (bus arbitration and conflicts).
```

The PPU's CPU-facing registers (LCDC, STAT, SCX, SCY, LY, LYC, BGP, OBP0,
OBP1, WX, WY) are accessed via distinct paths for reads and writes:

- **Reads**: purely combinational — address decode plus the ungated
  `ppu_rd` enables tri-state drivers from the underlying DFF Q outputs
  onto the bus. There is no read strobe; the CPU latches the bus on its
  own schedule.
- **Writes**: PPU-side level-sensitive latches, transparent during the
  CUPA strobe — which fires once per M-cycle at a deterministic position,
  making every writable register transparent at the same time.

The asymmetry reflects which side is the receiver. Reads just drive the
bus combinationally while the CPU requests the address (the CPU is the
receiver); writes must strobe the PPU-side register latches closed on
specific edges (the PPU is the receiver).

## What's where

| Page | Covers |
|------|--------|
| [Reads](registers/reads.md) | The combinational read path, the full per-register driver map, and the STAT driver-family split |
| [Writes](registers/writes.md) | The CUPA strobe, the phase generator behind it, and what a mid-rendering write affects |
| [Palette latches](registers/palette-latches.md) | BGP/OBP0/OBP1's latch cells, the mid-Mode-3 write race, and the BGP OR-overlap |
| [LCDC structure](registers/lcdc.md) | LCDC's storage cells and the per-bit consumer index |
