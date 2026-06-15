# Races

This chapter consolidates the static race analysis: which signal pairs race,
which orderings are structural, and — just as importantly — where
propagation delay does *not* matter. The source is graph analysis over the
netlist (depths in gate equivalents from the master clock; see
[Methodology](methodology.md)).

```admonish abstract "At a glance"
- Only **13 per-dot races** exist, and none is a data contention: 5 are
  clock-vs-settled-data skew, 8 are the BG temp-latch *late enable* — which
  is what makes the capture correct.
- The **sprite store** is the deepest per-line path (71.2 ge, negative
  slack at worst case) — the source of the first-entry staleness window.
- The palette/SACU pair is **not a race**: a mid-Mode-3 write's latch
  update lands ~15.6 ns after the write-dot SACU↓ (data-bus settling).
- Most register paths are **per-frame**: stable absent CPU writes.
```

## Same-clock synchronous paths

DFFs sharing a clock capture on the same edge with the full period as
settling budget — race analysis does not apply:

| Clock | DFFs |
|-------|------|
| `alet` | NYKA, LYZU, PYGO, RENE, DOBA, NOPA, VOGA |
| `myvo` | PORY |
| `sacu` | PX bits 0–3 + 16 BG shifter + 16 sprite shifter + 16 attribute pipe (52 total) |
| `talu` | SAXO, NYPE |
| `sono` | RUTU, SYGU |

## Complementary-clock pipelines

A DFF on clock A feeding a DFF on NOT(A) forms a structural half-period
pipeline:

- **ALET → MYVO (NYKA → PORY)**: PORY always reflects what NYKA captured on
  the most recent ALET edge — a half-dot lag, not a full dot.
- **TALU → SONO (LX → LINE_END)**: the LX counter increments on TALU rising;
  RUTU captures the SANU decode on SONO rising — RUTU sees the LX value
  from one TALU edge ago.
- **SACU → TOCA (PX lower → upper nibble)**: PX bits 4–7 clock on
  TOCA = NOT(XYDO), one propagation delay after bit 3 — a small skew inside
  the dot that the XUGU terminal decode (which reads bits from both groups)
  absorbs.

## Cross-domain paths: the 13 per-dot races

**PPU Cycles group (5 races).** Clock-vs-registered-data skew, not data
contention: the data inputs are DFF Q outputs (depth 0) that settled on the
previous edge, so data is always stable before the clock arrives.

| DFF(s) | Clock | Clock depth | Data inputs |
|---|---|---|---|
| PORY | MYVO | 22.2 ge (tightest) | NYKA, NAFY (6.2 ge) |
| LYZU, PYGO, RENE | ALET | 16.3 ge | LAXU, PORY, RYFA |
| NYZE | MOXE | 16.7 ge | PUXA |

**BG FIFO data-latch group (8 races, all diff = 16.2 ge).** The temp-latch
enable propagates from fetch-counter bit 2 through
NYVA → NOFU → NYDY → METE → LOMA → LUNA, arriving 16.2 ge after the counter
changes — *after* the VRAM bus has stabilised with the current fetch's
data. The hardware ordering is therefore: bus stable → counter advances →
enable fires late → already-stable data captured. The late enable is what
makes the capture correct.

**Palette/SACU ordering (3 registers).** A mid-Mode-3 palette dlatch
update lands ~15.6 ns after the write-dot SACU↓
([palette latches](registers/palette-latches.md)) — the latch enable opens early
but the captured value follows the CPU data bus, which settles past the
pixel edge. The old-palette-on-the-write-dot outcome is structural: a
~15 ns ordering, not a marginal race.

## Per-line races

**Sprite store (100 races, diff 70.5–71.2 ge).** The deepest per-line path
in the PPU: MUWY (LY bit 0) → the Y-compare carry chain → slot decode →
EBEB (the store write enable) at 71.2 ge, against OAM data arriving at
depth 0. At the worst-case gate-delay estimate the path *misses* its
1-M-cycle deadline — negative slack. The hardware effect: on the very
first OAM entry after LY increments, the Y-comparator result can be one
cycle stale; subsequent entries settle normally. This is the
first-entry-staleness window noted in [OAM scan](oam-scan.md) (CENO's
one-XUPY-cycle lag is the other half of that story).

**Sprite FIFO / X match (44 races, diff 30.7–63.8 ge).** Sourced from
CENO. These settle during Mode 2's 80 dots, long before CLKPIPE starts
toggling — ample slack for the rest of the line.

**Sprite Y-compare captures (TOBU/VONU/SEBA, diff 25.8–27.3 ge).**
Technically per-dot but only fire during the scan's 2-dot-per-entry
window, which comfortably covers them.

## Where propagation delay does not matter

"Per-frame" below means the race's critical source only changes on CPU
writes — the writes themselves land mid-frame at the dot the
[register-write chapter](registers/writes.md) describes.

| Area | Domain | Notes |
|------|--------|-------|
| Palette registers | Per-frame | Stable absent writes; the mid-Mode-3 write ordering is the structural case above |
| SCX | Per-frame | Mid-scanline writes propagate through the VRAM address adder (66.2 ge). SCX feeds **only the counter=0 tilemap stage** (the map column) — the tile-data stages use the cached tile index, and SCX's fine bits drive only the startup fine scroll (pixel-clock suppression, [startup and fine scroll](bg-pipeline/startup-fine-scroll.md)), never a tile-data address, so there is **no LCDC.4-style hybrid-fetch path for SCX**: a write landing between counter=2 and counter=4 cannot mix bitplanes. Writes before/after counter=0 split exactly like LCDC.3/.6 sampling ([BG pipeline](bg-pipeline/fetcher.md)) |
| SCY | Per-frame | Same adder, but SCY feeds **two** fetch stages: the map **row** at counter=0 (once, like BG_MAP) **and** the fine-**Y** tile-data row offset at counter=2/4 (twice, once per bitplane — like TILE_SEL). So SCY **does** have an LCDC.4-style hybrid path: a write landing between counter=2 and counter=4 fetches the low bitplane from the old fine-Y row and the high bitplane from the new one ([BG pipeline](bg-pipeline/fetcher.md)) |
| WX, WY | Per-frame | WY compared once per line; WX per-dot but the compare is shallow |
| LCDC | Per-frame | All consumer sampling is combinational from the latches; the interesting cases are the per-bit mid-write behaviours covered in their consumer chapters |
| STAT enables | Per-frame | CUPA-strobed; the FF41 bus-settle transient is in [STAT interrupts](stat-interrupts.md) |
| LY == LYC | Per-frame | Full line to settle (boundary races excepted — [STAT interrupts](stat-interrupts.md)) |
| LCD output path | Static | Pixel data is combinational from shifter MSBs |
| External cartridge address pads | Per-line | CPU-vs-DMA contention on `a0..a10`, uniformly resolved in DMA's favour by the address MUX; the observable conflict is on the data side ([DMA](dma/source-bus.md)) |
| Internal OAM address bus | Per-line | Four mutually-exclusive tri-state enables — exactly one driver in every (mode, DMA) combination; not a deadline race at all, but it shapes what the scan latches during DMA ([DMA](dma/oam-bus.md)) |
