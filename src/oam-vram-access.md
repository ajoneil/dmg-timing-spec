# OAM and VRAM Access

The [registers chapter](registers.md) covered CPU writes to PPU-internal
register latches — all sharing the CUPA strobe. CPU writes to OAM ($FE00–$FE9F) and VRAM
($8000–$9FFF) follow distinct paths, because the storage is not in the PPU
register file: OAM is an on-chip dual-port SRAM with its own bus arbitration,
and VRAM is off-chip, with address/data/strobe pads driven by the PPU through
tri-state enables. Both paths gate the write enable on the rendering-mode
signals so that CPU writes during the corresponding lock window never reach
the storage.

*Which* windows are locked is well-trodden behavioural ground
([gb-ctr](https://gekkio.fi/files/gb-docs/gbctr.pdf) covers the accessibility
rules); this chapter documents the gates that enforce them, and what those
gates imply at the window boundaries.

```admonish abstract "At a glance"
- The two locks are **structurally different**: OAM gates the write
  strobe on-chip (AJUJ → AMAB → WYJA); VRAM tri-states the off-chip
  address pads and write strobe (ROPY/XANE) — there is no missed-write
  latch on either side.
- The OAM write window **is the CUPA pulse** — the `ppu_wr` net is the
  CUPA cell's output — gated by the AJUJ/AMAB permit; only VRAM's strobe
  is assembled separately, from the SM83 bus-phase signals.
- Lock transitions are **M-cycle-quantised** against CPU writes — a write
  window is fully open or fully closed; outcomes flip exactly one M-cycle
  apart, never partially.
- A mode transition *inside* the write window is the real boundary case:
  all three OAM-side straddles are characterised
  ([CPU-visible boundaries](cpu-visible-boundaries.md)).
- Any CPU bus cycle addressing $FExx during Mode 2 suppresses one OAM
  **bitline-precharge** beat — the next row's wordline opens onto the
  previous row's charge: the **OAM corruption bug**, measured as an exact
  8-byte row copy.
```

## What's where

| Page | Covers |
|------|--------|
| [OAM writes](oam-vram-access/oam-writes.md) | The on-chip write-strobe chain (AJUJ → AMAB → WYJA), the CUPA-pulse window, and the per-byte strobes |
| [VRAM writes](oam-vram-access/vram-writes.md) | The off-chip pad path, the Mode-3 tri-state lock, and the SM83 bus-phase strobe |
| [Lock boundaries](oam-vram-access/lock-boundaries.md) | M-cycle-quantised lock transitions, and the straddle cases at a mode edge |
| [OAM corruption](oam-vram-access/corruption.md) | The suppressed-precharge bug — a $FExx access in Mode 2, measured as an exact 8-byte row copy |
