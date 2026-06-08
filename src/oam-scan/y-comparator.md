# The Y Comparator

During the scan, the hardware compares each sprite's Y position against LY.
The comparison is an 8-stage full-adder carry chain with a NAND6 decoder, and
the match result is **combinational** — there is no DFF capture of the match
during the scan. `sprite_y_match` gates the per-slot sprite-store
write-enables directly via the CARE chain.

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| GESE | `sprite_y_match` driver | not_x1 | combinational (input: WOTA) | `sprite_y_match` = NOT(WOTA) |
| WOTA | Y-compare decoder | nand6 | combinational | Inputs: GACE, GUVU, GYDA, GEWY, `wuhu_c`, GOVU. Low when the Y-in-range condition holds |
| GYKY | Carry-chain bit 3 | full_add | combinational | See the chain table |
| GOVU | LCDC.2 mask | or2 | combinational | GOVU = OR2(`ff40_d2`, GYKY) — masks the bit-3 comparison for 8×16 sprites |
| XUSO | OAM Y-offset bit 0 latch (Mode 2) / tile-index bit 0 latch (Mode 3) | dlatch_ee | Enable: COTA | Data: YDYV (Stage 1). `xuso_n` feeds carry-chain bit 0 operand B in Mode 2 and the sprite tile VRAM row-bit derivation in Mode 3 ([Sprite pipeline](../sprite-pipeline/fetch-machine.md)) |
| Y-side Stage-1 register | 8 × dlatch on the bank-B bus | dlatch | Enable: `oam_data_latch` (shared) | YDYV, YCEB, ZUCA, WONE, ZAXE, XAFU, YSES, ZECA — capture the even byte of the addressed OAM byte-pair ([DMA](../dma/oam-bus.md) for the byte-pair architecture) |

The chain computes LY − spriteY in two's-complement form:

- **Operand A** — LY, complemented: each a-input is a `not_x1` on a
  V-counter bit (EBOS…GUSU = NOT(`v0`…`v7`)).
- **Operand B** — the OAM Y byte, complemented: the Stage-2 latches'
  q̄ outputs (XUSO and its seven siblings, each holding the Stage-1 byte
  through the COTA-gated rank).
- **Stage-0 carry-in** — constant 0.

| Stage | Gate | a input (NOT LY bit) | b input (OAM-Y latch q̄) | cin | cout | sum |
|-------|------|---------|----------------------|-----|------|-----|
| 0 | ERUC | EBOS | `xuso_n` | const0 | `eruc_c` | ERUC |
| 1 | ENEF | DASA | `xegu_n` | `eruc_c` | `enef_c` | ENEF |
| 2 | FECO | FUKY | `yjex_n` | `enef_c` | `feco_c` | FECO |
| 3 | GYKY | FUVE | `xyju_n` | `feco_c` | `gyky_c` | GYKY |
| 4 | GOPU | FEPU | `ybog_n` | `gyky_c` | `gopu_c` | GOPU |
| 5 | FUWA | FOFA | `wyso_n` | `gopu_c` | `fuwa_c` | FUWA |
| 6 | GOJU | FEMO | `xote_n` | `fuwa_c` | `goju_c` | GOJU |
| 7 | WUHU | GUSU | `yzab_n` | `goju_c` | `wuhu_c` | WUHU |

The decoder is WOTA = NAND6(GACE, GUVU, GYDA, GEWY, `wuhu_c`, GOVU). Its
six inputs:

- **GACE, GUVU, GYDA, GEWY** = NOT(GOPU, FUWA, GOJU, WUHU) — the
  upper-nibble sums, inverted (`not_x1` cells, netlist-derived);
- **`wuhu_c`** — the top-stage carry-out, the sign of the difference;
- **GOVU** — the LCDC.2 size mask (below).

WOTA asserts (low) exactly when sum bits 4–7 are all zero, the sign carry
is set, and the size mask holds — i.e. the difference lies in the
sprite-height range.

**LCDC.2 (OBJ_SIZE) in the comparator.** GOVU = OR2(XYMO, GYKY): with
XYMO=1 (8×16 sprites) the OR masks one bit of the range comparison,
accepting a 16-row match; with XYMO=0 it passes GYKY unchanged for 8 rows.
Because the compare is combinational, a mid-scan CUPA write to LCDC.2
splits the scan — entries before the write use the old range, entries
after use the new — so hybrid sprite-store populations are possible.
XYMO's other consumer (the sprite tile VRAM address bit, Mode 3) lives in
the [sprite pipeline](../sprite-pipeline/lcdc-paths.md).

**Capture durability.** XUSO's enable (COTA) is high through most of Mode 3
steady-state, so XUSO is *transparent* there — its held Mode-2 value survives
only because the Stage-1 latch upstream (YDYV) holds when `oam_data_latch`
stops pulsing. On scanlines **with** Mode-3 sprite fetches, each fetch pulses
`oam_data_latch` once and overwrites the Stage-1 byte-pair with the fetched
sprite's (tile index, attribute) — the Mode-2 Y bits do not survive past the
first fetch. This breaks nothing: the Y-compare chain is only consumed during
Mode 2.

