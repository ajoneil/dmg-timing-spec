# OAM Writes

A CPU write to $FExx with `cpu_wr`=1 reaches OAM through an on-chip
arbitration chain combining the address decode `fexx`, the rendering-mode
signals (`dma_run`, mode 2, mode 3), and the DMA write-path arm POWU. The
combined enable WYJA drives per-byte write strobes — YNYC (B side) and YLYC
(A side) — which become the active-low pad outputs `oam_b_wr_n` /
`oam_a_wr_n` to the OAM SRAM cells.

| Gate | Role | Type | Inputs | Notes |
|------|------|------|--------|-------|
| FEXX | $FExx address-range decode | not_x2 | = NOT(ROPE) | Active-high while `cpu_port_a` is in $FE00–$FEFF |
| AJON | Mode 3 + non-DMA | and2 | mode3, BOGE | BOGE = NOT(`dma_run`). AJON=1 only during Mode 3 with DMA inactive |
| AJUJ | OAM-access permit | nor3 | `dma_run`, mode2, AJON | AJUJ=1 only when DMA is inactive AND neither Mode 2 nor Mode 3 is active |
| AMAB | OAM CPU-write address gate | and2 | `fexx`, AJUJ | AMAB=1 only when the CPU addresses $FExx during a permitted window |
| WYJA | OAM write enable (combined) | ao21 | AMAB, `ppu_wr`, POWU | WYJA = (AMAB AND `ppu_wr`) OR POWU — `ppu_wr` is the CUPA cell's output net, so the CPU arm's window is the CUPA pulse; the DMA arm bypasses CPU arbitration |
| YLYC / YNYC | OAM A-side / B-side write strobe drivers | per-byte selector | WYJA + per-byte address decode | Drive the active-low `oam_a_wr_n` / `oam_b_wr_n` pad outputs |

**Lock-window behaviour.** The permit AJUJ decides the outcome:

| State | Permit | Outcome |
|---|---|---|
| Outside Mode 2/3, no DMA | AJUJ=1, AMAB=1 | WYJA passes `ppu_wr` — write lands |
| Mode 2 | mode2=1 → AJUJ=0 | blocked (BESU gates OAM — [OAM scan](../oam-scan.md)) |
| Mode 3 | XYMU.q=0 → AJON=1 → AJUJ=0 | blocked |
| DMA | `dma_run`=1 → AJUJ=0 | CPU blocked; POWU drives WYJA with DMA-source data ([DMA](../dma/oam-bus.md)) |

**CUPA-vs-mode-edge alignment.** A write landing in a stable open window
(mode2 = mode3 = 0 across the M-cycle) passes `ppu_wr`'s pulse through WYJA
cleanly; one landing in a stable locked window is blocked (AJUJ=0 holds
WYJA=0). The narrower case — a mode-bit transition *inside* the write
window — is real and measured: the Mode 2→3 straddle opens the AJUJ permit
for a 6,502 ps window during the AVAP cascade, worked through in
[CPU-visible timing at mode boundaries](../cpu-visible-boundaries.md).
