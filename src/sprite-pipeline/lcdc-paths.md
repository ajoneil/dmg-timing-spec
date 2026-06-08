# LCDC.1 and LCDC.2: The Sprite Consumer Paths

Two LCDC bits act directly inside the sprite pipeline. OBJ_EN (LCDC.1)
splits into a trigger path and an output path that a mid-Mode-3 write can
divorce; OBJ_SIZE (LCDC.2) feeds the VRAM row-bit mux live while sprite
visibility stays scan-latched.

```admonish abstract "At a glance"
- XYLO (OBJ_EN) has **two pipeline consumers**: AROR gates the trigger,
  XULA/WOXA gate the output mux — a mid-Mode-3 flip between fetch and
  emission splits them.
- A mid-fetch OFF write drops FEPO within gate delays of CUPA↑; **SACU resumes
  mid-fetch** and Mode 3's end is set by the FEPO↓ edge.
- On hardware, the OFF direction emits **one shade-3 column** — a
  below-netlist behaviour pinned by reference photographs.
- XYMO (OBJ_SIZE) is **live** in the VRAM row-bit path, but visibility was
  latched at scan time — mid-Mode-3 writes produce hybrid artefacts.
```

## LCDC.1 (OBJ_EN): the pixel-mux paths

XYLO has four netlist consumers: AROR (the trigger gate —
[OAM scan](../oam-scan.md)), XULA and WOXA (this section), and XERO (CPU
read-back).

| Gate | Role | Type | Inputs | Notes |
|------|------|------|--------|-------|
| XULA | Plane-B pixel-output gate | and2 | XYLO, `sprite_px_b7` (WUFY) | |
| WOXA | Plane-A pixel-output gate | and2 | XYLO, `sprite_px_a7` (VUPY) | |
| NULY | Sprite-visibility mask | nor2 | WOXA, XULA | 1 ⇔ sprite transparent or XYLO=0; drives the output-MUX combiner POKA ([LCD output](../lcd-output.md)) |

**Trigger vs output divergence.** Steady-state XYLO is redundant across the
two paths, but a mid-Mode-3 flip between a sprite's fetch and its emission
splits them: data in the shifter but gated at output, or vice versa. The
outcome of a write depends on where the CUPA window lands relative to the
trigger's capture edge:

| CUPA window | Outcome |
|---|---|
| Entirely before TEKY's ALET edge | AROR=0 kills the trigger — no fetch, no penalty |
| Entirely after SOBU's capture | Fetch completes; XULA/WOXA gate the output (sprite suppressed at the mux) |
| Straddling the edge | Sub-dot race between the level-sensitive XYLO latch and the edge-triggered SOBU — not resolvable from topology alone |

**The mid-fetch OFF write and Mode 3's end.** In the "after SOBU" row, the
same write also drops FEPO within gate delays of CUPA↑ via AROR (TAKA is *not* on
the SACU halt path — VYBO watches FEPO). SACU resumes mid-fetch from the
AROR↓ dot; the fetch machine runs on; Mode 3's end is set by the FEPO↓
edge. Measured across the four gambatte
`sprites_late_late_disable_spx{18,19,1A,1B}` ROMs (bit-identical AVAP, all
four firing the write inside the freeze window), the closed form

```
mode3_end(SPRITEX, AROR↓) = floor(AROR↓ dot) + 167 − SPRITEX + 0.436
```

predicts the measured XYMU↑ to <0.001 dots in all four cases (dmg-sim
measurement). The +3-dot step in the data between SPX=0x19 and 0x1A is a
test-harness artifact — the +1-NOP CUPA shift (+4 dots) net of the −1-dot
SPRITEX step — not a tile-boundary effect; the formula holds across it.

### The OFF-CUPA single-pixel transient

Like the BGP OR-overlap ([palette latches](../registers/palette-latches.md)),
LCDC.1 OFF transitions exhibit a behaviour **below the netlist's modelling
scope**, characterised against hardware reference photographs:

> When a CUPA write transitions LCDC.1 from 1 to 0 inside Mode 3, the first
> `cp_pad`↑ after that CUPA↑ (+0.534 dots) emits **one anomalous LCD
> column** on real DMG hardware — shade 3 in the Mealybug reference image —
> before the sprites-off state takes hold. Subsequent columns follow the
> normal mux path. The 0→1 RESTORE direction produces no transient.

The gate-level simulation shows *zero* transitions on NULY/POKA/PATY across
the affected scanlines and white output at every column — the netlist does
not produce the symptom at any available timing calibration. The reference
images show the transient walking columns x=1..7 across LY≈24..125 (the
walking reflects per-LY `cp_pad` cadence variation from sprite-stall
positions; the write's AVAP-offset is constant within each test branch),
with verified column agreement at five sampled LYs.

Two further empirical facts:

- some LYs at frame edges and a few scattered rows do not fire the
  transient at all despite ps-identical netlist state (verified across all
  144 LYs — no signal differs between firing and non-firing rows);
- multi-pixel rows in the reference carry *additional* earlier columns that
  are ordinary gate-level shifter content, not part of this rule.

```admonish note "For implementors"
Modelling the rule as an asymmetric consumption — the OBJ-mux popper reads
the pre-transition XYLO for one `cp_pad` while the trigger chain sees the
live value — reproduces the column-precise behaviour with no per-LY
exception list.
```

```admonish question "Open question: the transient's mechanism"
Whether the silicon mechanism is a popper-side sub-dot ordering, pad-driver
residue, or LCD-glass sample-and-hold requires hardware measurement; the
per-LY firing scope likewise. See [Appendix C](../open-questions.md).
```

## LCDC.2 (OBJ_SIZE): the VRAM address path

XYMO's second consumer (the first is the Y comparator —
[OAM scan](../oam-scan.md)) selects the sprite tile-data row bit during the
fetch:

| Gate | Role | Type | Inputs | Notes |
|------|------|------|--------|-------|
| FUFO | XYMO complement | not_x1 | XYMO | |
| GEJY | OBJ_SIZE mux | ao22 | `xuso_n`, FUFO, `ff40_d2`, WAGO | 8×8: passes `xuso_n` (Y-offset bit 0); 8×16: passes WAGO |
| WAGO | 8×16 half-select | xor | `sprite_y_store3`, WUKY | Which half of the 16-row sprite |
| WUKY | Y-flip term | not_x1 | YZOS | YZOS latches OAM attribute bit 6 |
| FAMU | `~ma4` tri-state driver | not_if0 | in = GEJY, ena_n = ABON | One of 6 `~ma4` drivers; the sprite-fetch arm |
| ABON | FAMU enable | not_x2 chain | from TULY/VONU/mode3 | Active (low) exactly during the tile-data stages of the sprite fetch |

**Mid-Mode-3 OBJ_SIZE writes.** Sprite *visibility* was decided at scan
time and is latched; XYMO writes cannot change it. The VRAM row-bit path,
however, is live: any in-progress or subsequent fetch uses the new XYMO
combinationally through GEJY. The result is a hybrid — visibility under the
old size, tile-data row under the new — producing the small rectangular
artefacts the Mealybug `m3_lcdc_obj_size_change` tests exercise.
