# The Tile Fetcher

The BG/window tile fetcher is a 3-bit counter, a drain-detect pipeline, and a
combinational VRAM address generator. It runs continuously through Mode 3,
producing one tile (two bitplane bytes) every 8 dots, in parallel with pixel
shifting.

```admonish abstract "At a glance"
- A 3-bit ripple counter steps through six states (0–5) — one tile fetch =
  3 VRAM accesses × 2 dots — then holds at 5 until the NYXU reset pulse
  restarts it.
- The full tile cadence is 8 dots: 6 counting plus 2 while the drain
  cascade (NYKA → PORY → RENE/RYFA → SEKO) empties and retriggers NYXU.
- NYXU fires 22 times per scanline at SCX=0, 23 at SCX&7 ∈ {1..7}.
- LCDC.3/4/6 reach the VRAM address bus through level-sensitive paths — no
  clocked stage — and TILE_SEL is sampled twice per fetch.
```

## The fetch counter

The counter advances through six states (0–5) to drive one tile fetch,
then holds at 5 until the NYXU reset pulse restarts it for the next tile,
window handoff, or Mode 3 startup. (The downstream drain-detect pipeline
that closes each tile is tabulated in the next section.)

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| LEBO | Fetch-counter gated clock | nand2 | Inputs: MOCE, ALET | When MOCE=0 (terminal count), LEBO freezes at 1; the counter stops |
| LAXU | Fetch counter bit 0 | dffr | LEBO | Async reset by NYXU |
| MESU | Fetch counter bit 1 | dffr | LAXU (~Q ripple) | Async reset by NYXU |
| NYVA | Fetch counter bit 2 | dffr | MESU (~Q ripple) | Async reset by NYXU |
| MOCE | Self-stop condition | nand3 | Inputs: LAXU, NYVA, NYXU | Drives LEBO low at count 5 when NYXU=1 |
| NYXU | BG fetch counter reset | nor3 | Inputs: MOSU, TEVO, AVAP | Async reset; also forces MOCE=1, re-enabling LEBO |

```
LEBO (nand2) = NAND(MOCE, ALET) — gated clock, stops when MOCE=0
├── LAXU (dffr) [clk=LEBO] — bit 0
│   ├── MESU (dffr) [clk=LAXU.~Q] — bit 1 (ripple)
│   │   └── NYVA (dffr) [clk=MESU.~Q] — bit 2 (ripple)
│   └── data to LYZU
MOCE (nand3) = NAND(LAXU, NYVA, NYXU) — self-stop at count 5 when NYXU=1
```

**Reset and restart.** NYXU = NOR3(MOSU, TEVO, AVAP) combines three triggers:
window activation (MOSU), tile boundary (TEVO), and OAM-scan completion
(AVAP). A single ~0.5-dot NYXU pulse asynchronously resets the counter DFFs
*and* forces MOCE=1, re-enabling LEBO — clear and clock-restart in one
operation. At count 5 (LAXU=1, MESU=0, NYVA=1), MOCE freezes LEBO and the
counter holds; the six states match the 6 dots of one tile fetch (3 VRAM
accesses × 2 dots).

**NYXU firings per scanline** under normal BG rendering — 22 at SCX=0, 23 at
SCX&7 ∈ {1..7}:

| Source | Count | Fires at | Role |
|---|---|---|---|
| AVAP | 1 | Mode 2→3 transition | Starts Mode 3, before any tile fetch |
| TAVE → TEVO | 1 | +5.996 dots from AVAP | Closes tile 0 (the startup fetch) |
| SEKO → TEVO | 20 | 8-dot boundaries from +13.997 dots | Closes tiles 1–20 (the 20 visible BG tiles) |
| SEKO → TEVO (tail) | 0 or 1 | +173.997 dots, fixed | Closes tile 21 at SCX&7 ∈ {1..7} only — suppressed at SCX=0 ([the tail drain](startup-fine-scroll.md#the-tail-of-mode-3-drain)) |

The 167-pixel SACU count per Mode 3 is decoupled from the TEVO firings —
SACU runs at CLKPIPE cadence, not fetch cadence.

## The 8-dot tile cycle

The counter counts 0→5 over 6 dots, then holds at 5 for 2 dots while the
cascade (NYKA → PORY → RENE/RYFA) drains. When SEKO detects the drained
state, TEVO fires and resets the counter for the next tile. The counter is a
*position within the current tile fetch*, not an absolute Mode 3 position —
fetching and pixel shifting run in parallel, and the per-tile TEVO reset is
normal fetcher operation, not a stall.

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| LYRY | "Fetch complete" combinational | not_x1 | Input: MOCE | High when the counter sits at terminal with no reset — drives NYKA's data input |
| NYKA | Fetch-complete capture | dffr | ALET | Data: LYRY |
| PORY | NYKA pipeline stage | dffr | MYVO | Half-dot-later capture of NYKA |
| LYZU | Fetch counter bit 0 sample | dffr | ALET | Data: LAXU |
| PYGO | PORY pipeline stage | dffr | ALET | Feeds POKY |
| RENE | Drain-detect stage 2 | dffr | ALET | Data: RYFA |
| RYFA | Drain-detect stage 1 | dffr | SEGU | Data: PANY |
| TEVO | Tile-boundary / restart trigger | or3 | Inputs: SEKO, SUZU, TAVE | Drives NYXU per tile, per window trigger, per startup |
| SEKO | Tile-boundary drain detector | nor2 | Inputs: RENE, RYFA | Fires when the pipeline drains (both 0) |
| SUZU | Window-activation trigger | not | Input: TUXY | See [Window control](../window.md) |
| TAVE | Startup / window-restart trigger | not | Input: SUVU | SUVU = NAND4(NYKA, PORY, ROMO, mode3) |

The 2-dot hold, stage by stage:

1. The counter reaches 5: MOCE→0 freezes LEBO, and LYRY→1 starts propagating
   down the NYKA/PORY/PYGO half-dot pipeline; POKY sets; RYFA still holds 1.
2. Hold-dot 1: RENE captures RYFA=1 — SEKO stays 0.
3. Hold-dot 2: PANY falls, RYFA captures 0, RENE follows — and once both are
   0, SEKO rises, TEVO fires, NYXU resets the counter, and LYRY's fall
   propagates back down, returning SEKO to 0.

**The cascade does not drain during a sprite fetch.** When FEPO=1 freezes
SEGU at 1 ([the pixel clock](pixel-clock.md)), RYFA is frozen at its
pre-freeze 1 and RENE keeps re-capturing 1 — SEKO never rises, so neither
TEVO nor NYXU fires during the sprite window. The counter holds at 5 by the
ordinary MOCE self-stop, but the reset that would normally come 2 dots later
simply does not arrive until the freeze lifts
([sprite pipeline](../sprite-pipeline/fetch-machine.md)).

```admonish warning "Pitfall: the PANY/NUKO slip"
PANY's `wxy_match` input (NUKO) means any NUKO=1 landing inside PANY's
natural 1-dot tile-boundary window perturbs the drain pulse, slipping the
cascade by one dot. The slip delays the BG-shifter parallel-load
**unconditionally** — this branch is *not* gated by LCDC.5 or `in_window` —
and additionally delays the window tile-X increment only when `in_window`=1.
NUKO=1 inside the window arises from a mid-Mode-3 WX rewrite or from the
natural PX==WX sweep when REJO is armed with WX≡7 (mod 8) — including the
**armed-but-disabled** state (REJO=1, LCDC.5=0), where the BG still slips
even though no window renders. Full mechanism and measurements:
[Window edge cases](../window/edge-cases.md).
```

**Outside Mode 3** the MOSU/TEVO/AVAP cascade is idle, so NYXU=1 and the
counter is *not* held in reset — it freezes via the self-stop at whatever
value was last clocked (typically 5) until the next AVAP. Reset domains:

- **LAXU/MESU/NYVA** — reset on every NYXU pulse;
- **PYGO/RENE/RYFA/LYZU** — the mode3 reset net (held at 0 outside Mode 3);
- **NYKA/PORY** — NAFY (VID_RST plus window-related events).

## The fetch cycle and VRAM addressing

Each tile fetch takes 6 dots:

| Counter | What happens | VRAM address |
|---------|--------------|--------------|
| 0 | Tilemap fetch: address set | Tilemap address |
| 1 | Tilemap data arrives | — |
| 2 | Tile data low: address set | Tile-data address (low byte) |
| 3 | Tile data low arrives | — |
| 4 | Tile data high: address set | Tile-data address (high byte) |
| 5 | Tile data high arrives; MOCE stops the clock | — |

The set-address/data-arrives pattern assumes the standard async-SRAM model —
VRAM is external, and its access time is not determined by the on-die
netlist. The address is driven combinationally from the fetch counter
through the VRAM address adder (up to 66.2 ge for the full carry chain,
though the high bits settle from the previous line in practice).

## LCDC.3/4/6 sampling during the fetch

Three LCDC bits feed the fetcher's VRAM address generation, each through a
purely combinational path gated by tri-state drivers that the fetch counter
enables at specific stages:

| LCDC bit | Cell | Wire name | Drives | Path |
|----------|------|-----------|--------|------|
| 3 (BG_MAP) | XAFO | `LCDC_BGMAPp` | `~ma10` during BG tilemap reads | XAFO → AMUV (not_if0, enable BAFY) |
| 4 (TILE_SEL) | WEXU | `LCDC_BGTILEp` | `~ma12` during tile-data reads | WEXU → VUZA (nor2 with tile-index bit 7 PYJU) → VURY (not_if1, enable `bp_cy`) |
| 6 (WIN_MAP) | WOKY | `LCDC_WINMAPp` | `~ma10` during window tilemap reads | WOKY → VEVY (not_if0, enable WUKO) |

The enables derive combinationally from the fetch counter:

- **BAFY / WUKO** (BG vs window tilemap arm) — mutually exclusive, both
  fire at counter=0.
- **`bp_cy`** (the NETA AND2 output) — active across counters 2–5
  (measured 4.000 dots continuous per fetch), so **TILE_SEL is read twice
  per fetch**, once per bitplane.
- **PYJU** (dffr_cc_q) — captures VRAM data bit 7 at the tile-index stage
  and feeds VUZA, implementing the signed/unsigned tile-index split.

The sampling cadence:

| Counter | Stage | LCDC bits sampled |
|---------|-------|-------------------|
| 0 | Tilemap address | LCDC.3 **or** LCDC.6 (BAFY/WUKO arbitration) |
| 1 | Tilemap data | — (tile index captured) |
| 2 | Tile-data low address | LCDC.4 |
| 3 | — | — |
| 4 | Tile-data high address | LCDC.4 **(second sample)** |
| 5 | — | — |

```admonish tip "Rule: the fetch-address bits are level-sensitive, not edge-captured"
There is no clocked stage between an LCDC latch and the VRAM address bus —
the address bit at any instant is whatever the latch holds, gated by the
tri-state enable. TILE_SEL is sampled twice per fetch, so a mid-fetch
change mixes bitplane data from two different tile patterns; BG_MAP and
WIN_MAP sample once per fetch, at counter=0.
```

**Mid-fetch writes.** A CUPA write landing mid-cycle takes effect by
stage: before the sampling stage — normal; during it — the fetched byte is
the one the address held when the access window *opened*; after it — next
fetch. For BG_MAP/WIN_MAP this produces tile-aligned 8-pixel divergence;
for TILE_SEL, a write landing between counter=2 and counter=4 produces a
**hybrid fetch** — low bitplane from the old tile-data block, high bitplane
from the new (the Mealybug `m3_lcdc_tile_sel_win_change` reference image).
Model it by reading each bitplane once, at its own sample stage.

```admonish info "Measured: the LCDC.4 write path"
From the Mealybug `m3_lcdc_tile_sel_win_change` ROM (dmg-sim measurement; 1,019
writes per direction, all ps-identical): the LCDC-write strobe WARU rises
at sub-dot 0.507 with a 1.493-dot pulse (the CUPA width); `ff40_d4` settles
within ~1.4 ge after WARU↑ on a 0→1 write but **~32 ge** on a 1→0 write —
a measured rise/fall asymmetry of the drlatch cell; `~ma12` follows within
a gate delay once `bp_cy` enables VURY.
```

## LCDC.5 transients and the fetcher arbitration

The window-enable consumer chain enters the tilemap-select arbitration
through PYNU only, and the chain PYNU → NOCU → PORE → `in_window` →
AXAD/XEZE → BAFY/WUKO is purely combinational (~3 ge). A mid-Mode-3 LCDC.5
1→0 write therefore switches the fetcher back to BG-tilemap arbitration on
the dot the write lands: every subsequent counter=0 stage drives `~ma10`
from XAFO.

Measured across a 65-dot LCDC.5↓ gap: all 8 counter=0 stages in the gap
drive the BG tilemap region, with VEVY tri-stated (high-Z) and AMUV active
(dmg-sim measurement, Mealybug `m3_lcdc_win_en_change_multiple` ROM) —
matching the
hardware reference image, which shows BG content in the gap.

```admonish note "For implementors"
Sample `in_window` live at counter=0 of every fetch. A latched fetcher-side
window-mode flag that ignores PYNU↓ renders window content where hardware
shows BG — there is no fetcher-side window latch in the netlist beyond the
cycle-local tile-index capture.
```
