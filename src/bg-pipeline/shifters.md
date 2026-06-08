# The BG Shift Registers

The shifters are where fetched tile data becomes a pixel stream: two 8-bit
planes shifting under SACU, refilled in parallel every 8 dots from a bank of
temporary latches. The interesting timing is all in the load path — when it
fires, and why loaded data can never race the shift clock.

```admonish abstract "At a glance"
- Two 8-bit planes of dffsr cells, clocked by SACU; tile data staged
  through 8 temporary latches off the VRAM bus.
- The parallel load is **level-sensitive**, fired by NYXU via LOZE —
  independent of the shift clock.
- Load enable settles at 16.2 ge against SACU's 63.8 — loaded data is
  always stable before any shift edge.
- Startup fires **two loads**: stale content at AVAP, fresh at +5.996
  dots; the stale load never reaches the glass.
```

## The cells

Two 8-bit shift registers (planes A and B), each an 8-stage chain of dffsr
cells clocked by SACU. Tile data arrives via an 8-bit temporary latch group
whose enable chain hangs off fetch-counter bit 2.

| Gate family | Role | Type | Clock / Trigger | Count | Notes |
|-------------|------|------|-----------------|-------|-------|
| BG shifter plane A | Tile data plane 0 | dffsr | SACU | 8 | MYDE–PYBO range |
| BG shifter plane B | Tile data plane 1 | dffsr | SACU | 8 | TOMY–SOHU range |
| Temp data latches | VRAM bus capture | dlatch_ee_q | Enable: LUNA/LOMA | 8 | LEGU, LUZO, MEGU, MUKU, MYJY, NASA, NEFO, NUDU |
| LUNA / LOMA / METE / NYDY / NOFU | Temp-latch enable chain | not / nand3 | from NYVA | 5 | Depth 16.2 ge at LUNA |

The 16 shifter cells: MACU, MODU, MOJU, MYDE, NEDA, NEPO, NOZO, PYBO, RALU,
RYSA, SADY, SETU, SOBO, SOHU, TACA, TOMY — chain order determined by dffsr
`d`-pin connectivity.

## Parallel load via NYXU

Each cell has a NAND2 pair driving its `s_n` / `r_n`, gated by LOZE =
NOT(NYXU): `s_n` = NAND2(LOZE, temp bit), `r_n` = NAND2(LOZE, temp bit
complement). When NYXU pulses low, LOZE pulses high and every cell loads its
temp-latch bit; when NYXU is idle, both NANDs output 1 and the cells shift
normally under SACU. The load is level-sensitive and independent of the
shift clock — and since the load enable settles at 16.2 ge against SACU's
63.8, loaded data is stable before any shift edge.

## Two parallel loads per Mode 3 startup

NYXU fires twice during the
[startup cascade](startup-fine-scroll.md#the-startup-pipeline-avap-to-first-sacu):

1. **Load #1 at AVAP (dot 0).** The temp latches still hold pre-Mode-3
   content — the previous scanline's last tile, or power-on state on the
   very first frame. The shifters load this **stale** content.
2. **Load #2 at the first TEVO (dot 5.996).** By now the fetch counter has
   run 0–5 and the temp latches hold this scanline's first tile. The
   shifters load the **fresh** content, overwriting load #1.

The first SACU edge lands at 7.026 — *after* load #2 — and CP is gated by
PX ≥ 9, so load #1's stale content never reaches the glass. This matters
cross-frame: on the first frame vs a steady-state frame the stale content differs, but load #2
overwrites it before any CP sample either way, so LD output is identical.
At mid-Mode-3 tile boundaries only the TEVO-driven load fires — 20 more per
scanline, 21 at SCX&7 ≥ 1
([the tail drain](startup-fine-scroll.md#the-tail-of-mode-3-drain)).
