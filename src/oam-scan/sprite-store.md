# The Sprite Store

The store captures up to 10 sprites' X positions (plus per-slot state)
into dedicated latch arrays — the first 10 Y-matching sprites in OAM-index
order, the widely documented DMG behaviour
([gb-ctr](https://gekkio.fi/files/gb-docs/gbctr.pdf) covers the rule; this
page covers the machinery). Each slot has its own write enable
(`save_sprite_numN`, N ∈ [0..9]): a 4-bit slot counter and a 10-way
one-hot decoder make exactly one enable fire per matching sprite.

| Gate family | Role | Type | Clock / Trigger | Count | Notes |
|-------------|------|------|-----------------|-------|-------|
| Sprite store slots | Per-slot X-position latches | drlatch_ee | Enable: `save_sprite_numN` | 80 (10 slots × 8) | Data: OAM bus (depth 0) |
| EBEB | Slot-3 write-enable inverter | not_x1 | combinational | 1 | drives `save_sprite_num3` (logic NOR2(`sprite_save_en_n`, FOCO)). Depth 71.2 ge from MUWY via the Y-compare chain — the per-line critical path |
| CARE | Global sprite-save enable | and3 | combinational | 1 | Inputs: XOCE, CEHA, `sprite_y_match`; `sprite_save_en_n` = NOT(CARE) |
| FOCO | Slot-3 selector | nand4 | combinational | 1 | Decodes slot counter = 3; one of 10 per-slot selectors |
| BESE / CUXY / BEGO / DYBE | Slot counter bits 0–3 | dffr (toggle) | CAKE, then ~Q ripple | 4 | Reset: AZYB |
| AZYB | Slot counter reset driver | not_x1 | combinational | 1 | AZYB = NOT(ATEJ) — same start-of-line reset as the scan counter |

The priority mechanism:

1. `sprite_y_match` is combinationally established per OAM entry.
2. When it holds AND XOCE=1 AND CEHA=1, CARE=1 releases the write enables.
3. The slot counter (matched sprites so far) selects which slot's selector
   fires — FOCO decodes counter = 0b0011, via FYCU = BESE, FONE = CUXY,
   CAPE = NOT(BEGO), CAXU = NOT(DYBE).
4. `save_sprite_numN` = NOR2(`sprite_save_en_n`, selector_N) fires only when
   enabled AND targeted.
5. CAKE pulses once per stored sprite, advancing the counter.
6. At counter = 10 no decoder matches; remaining matching sprites pass
   through without a write.

**The late write enable.** The store's data input is the OAM bus (depth 0);
its write enable arrives through the deepest per-line combinational path in
the PPU (71.2 ge from MUWY). The enable arrives very late relative to data —
the family of races this creates is catalogued in [Races](../races.md).

**X-side Stage-1 register.** The per-slot latches store
`sprite_x0_n`..`x7_n`, derived combinationally (via YLOR/ZAGO) from the
X-side Stage-1 capture register: XYKY, YRUM, YSEX, YVEL, WYNO, CYRA, ZUVE,
ECED on the bank-A bus, sharing the `oam_data_latch` enable with the Y
side. [DMA](../dma/oam-bus.md) covers the byte-pair semantics and the hold
under DMA.

**Reset domains.** Each of the store's 80 X-position latches resets via
its slot's `sprite_rstN` = NOR2(ABAK, fetched flag):

- **ABAK = OR2(ATEJ, AMYG)** — AMYG carries LCD-off (every slot held at 0
  while the LCD is off); ATEJ pulses at each scanline boundary, including
  every VBlank line (RUTU keeps firing through LY 144–153).
- **The per-slot fetched flag** (EBOJ/CEDY/FONO family, clocked by WUTY)
  holds a fetched slot in reset until the next line's ATEJ.

**Cross-frame state at the first scanline.** From the second frame on, the
first scanline's store is cleared by the VBlank ATEJ pulses and populated by
its own scan. On the first frame (LCD-on) it is cleared but **never
populated**, twice over:
BESU never sets (CARE never arms, the slot counter never clocks), and
`oam_data_latch` is mode2-gated (the comparator operands are never
refreshed from OAM).

## LCDC.1 (OBJ_EN): the sprite-trigger branch

LCDC bit 1 (XYLO) drives four netlist consumers, all combinational (the
full set is in [LCDC.1/.2 paths](../sprite-pipeline/lcdc-paths.md)). The
trigger-side branch:

> **XYLO → AROR → per-slot match NAND3s → FEFY/FOVE → FEPO**
>
> AROR = AND2(XYLO, AZEM), with AZEM = AND2(mode3, BYJO); the ten match
> NAND3s reduce through the two NAND5s to FEPO, the "sprite match active"
> aggregate.

With XYLO=0, AROR=0: every NAND3 settles high, FEPO stays 0, TEKY never
fires — **no sprite fetch starts at all**, SACU is never frozen, and no
penalty dots are added. The pixel-mux consumer paths (what happens to
already-fetched sprite pixels) live in the
[sprite pipeline](../sprite-pipeline/lcdc-paths.md).

