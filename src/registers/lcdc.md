# LCDC Structure

LCDC's 8 bits are stored as `drlatch_ee` cells sharing the CUPA write
strobe ([writes](writes.md)). Each bit has a distinct consumer path in a
distinct mode; this page is the index. LCDC *storage* lives here —
consumer-path detail lives with the mode that consumes the bit.

```admonish abstract "At a glance"
- Every LCDC consumer path is **combinational from the latch** — there is
  no clocked stage between an LCDC cell and its consumer.
- The static race analysis classifies every LCDC cell as **per-frame
  domain** (diff = 59.6 ge from the CPU data bus through the enable
  chain): stable absent CPU writes, no per-dot races.
```

| LCDC bit | Gate / wire name | Consumer | Chapter |
|----------|------------------|----------|---------|
| 0 (BG_AND_WIN_EN) | VYXE / `LCDC_BGENp` | BG/window pixel plane gate (RAJY/TADE); mid-Mode-3 toggle behaviour | [LCD output](../lcd-output.md) |
| 1 (OBJ_EN) | XYLO / `LCDC_SPENp` | Sprite trigger (AROR → per-sprite NAND3 decode) | [Sprite pipeline](../sprite-pipeline/lcdc-paths.md) |
| 1 (OBJ_EN) | XYLO / `LCDC_SPENp` | Pixel MUX gates (XULA, WOXA) | [Sprite pipeline](../sprite-pipeline/lcdc-paths.md) |
| 2 (OBJ_SIZE) | XYMO / `LCDC_SPSIZEp` | Y-compare decoder (GOVU) — Mode 2 | [OAM scan](../oam-scan.md) |
| 2 (OBJ_SIZE) | XYMO / `LCDC_SPSIZEp` | Sprite tile VRAM ma4 (GEJY) — Mode 3 | [Sprite pipeline](../sprite-pipeline/lcdc-paths.md) |
| 3 (BG_MAP) | XAFO / `LCDC_BGMAPp` | BG tilemap address during fetch counter=0 | [BG pipeline](../bg-pipeline/fetcher.md) |
| 4 (TILE_SEL) | WEXU / `LCDC_BGTILEp` | BG tile-data address at counter=2/4 | [BG pipeline](../bg-pipeline/fetcher.md) |
| 5 (WIN_EN) | WYMO / `LCDC_WINENp` | Window enable (XOFO → PYNU.r) | [Window control](../window.md) |
| 6 (WIN_MAP) | WOKY / `LCDC_WINMAPp` | Window tilemap address at counter=0 | [BG pipeline](../bg-pipeline/fetcher.md) |
| 7 (LCDEN) | XONA / `LCDC_LCDENp` | VID_RST gate (LCDC.7 → XONA → XODO) | [Register writes](writes.md) |
| 7 (LCDEN) | XONA / `LCDC_LCDENp` | CPL/FR audio/PPU-clock mux (KAHE, KUPA AO22) — all modes including LCD-off | [LCD output](../lcd-output.md) |

A CUPA write mid-rendering transitions the latch output within the CUPA
pulse window (1.493 dots spanning T3–T4), and consumer effects
propagate combinationally. Per-bit sampling cadence within a fetch cycle,
and mid-Mode-3 write behaviour, are detailed in the consumer chapters.

LCDC.7 has two distinct consumer paths: the VID_RST gate (reset-and-enable
for the PPU's internal machinery) and the CPL/FR mux selector — the AO22 mux
that keeps the LCD's column-alternator pins toggling on APU clocks while the
LCD is off, protecting the glass from DC bias.
