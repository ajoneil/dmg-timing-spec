# The Counter and BESU

The scan's bookkeeping: a 6-bit ripple counter walks the 40 OAM entries,
and the BESU latch (with its XUPY-phased copy CENO) marks the scan as
active.

## The scan counter

The 6-bit OAM scan counter is a ripple chain clocked by GAVA = OR2(XUPY,
FETO), counting 0–39 across the 40 OAM entries. Each stage's output clocks the
next (the same ~Q convention as the LX and BG fetch counters). All six cells
share ANOM as reset, driven by the CATU/ANEL scanline-boundary chain
([Mode transitions](../mode-transitions.md)).

| Bit | Signal | Type | Clocked by | Notes |
|-----|--------|------|------------|-------|
| 0 (LSB) | YFEL | dffr | GAVA | Reset: ANOM |
| 1 | WEWY | dffr | YFEL (~Q ripple) | Reset: ANOM |
| 2 | GOSO | dffr | WEWY (~Q ripple) | Reset: ANOM |
| 3 | ELYN | dffr | GOSO (~Q ripple) | Reset: ANOM |
| 4 | FAHA | dffr | ELYN (~Q ripple) | Reset: ANOM |
| 5 (MSB) | FONY | dffr | FAHA (~Q ripple) | Reset: ANOM |

The terminal value 39 is 0b100111 — YFEL=WEWY=GOSO=FONY=1, ELYN=FAHA=0
(1+2+4+32 = 39) — the value visible in the boot-ROM handoff snapshot
([Post-boot state](../post-boot.md)).

### Clocking and reset

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| GAVA | Scan counter clock | or2 | Inputs: XUPY, FETO | Drives the YFEL→FONY ripple chain |
| ANOM | Scan counter reset | nor2 | Inputs: ATEJ, ATAR | Output is `oam_parse_reset_n`; see the CATU/ANEL chain ([Mode transitions](../mode-transitions.md)) |
| FETO | Scan-done decode | and4 | Inputs: `oam_parse_a2, a3, a4, a7` | Wire name `last_sprite`. Fires at counter = 39. Also feeds BYBA's D input |
| ASEN | BESU reset driver | or2 | Inputs: ATAR, AVAP | Rising edge clears the scan-active latch — at scan completion or video reset |
| ATAR | VID_RST active-high | not | Input: XAPO | Drives ANOM and ASEN |

XUPY = NOT(`wuvu_n`) = WUVU.q is the 2 MHz scan clock from the
[clock tree](../clock-tree.md). FETO's AND4 over `a2, a3, a4, a7` is another
partial decode that is exact within range: in [0..39], only 39 sets all four
of those bits.

**OAM bus address.** The counter drives OAM address bits `oam_parse_a2`–`a7`
(the entry index); bits a0–a1 come from a separate byte-cycle counter
advancing within each entry's 4-byte record. The address reaches the OAM SRAM
through the APAR tri-state driver — which DMA tri-states without stopping the
counter ([DMA](../dma/oam-bus.md)).

**XUPY buffer copies.** Two signals appear as OAM-scan clock sources for
per-byte-bit DFFs: CYKE = NOT(XUPY) and WUDA = NOT(CYKE) = XUPY. They are
physical buffer copies, not new signals — XUPY edges by another name.

**COTA, the composite Stage-2 enable.** COTA = NOT(BYCU) gates the Stage-2
capture latches (XUSO and siblings, below); BYCU = NAND3(CUFE, XUJY, AVER)
is a net AND3 of three condition terms:

- **AVER** = NAND2(mode2, XYSO) — scan phase;
- **CUFE** — CPU/DMA $FExx access;
- **XUJY** — sprite-fetch counter.

Its cadence therefore depends on CPU/DMA activity and mode state; during
Mode 3 steady-state with CPU and DMA idle it sits at 1 — Stage 2
transparent (dmg-sim measurement). COTA's complement WOVU drives the OAM
**bitline precharge** — the lever behind the OAM corruption bug
([OAM corruption](../oam-vram-access/corruption.md)).

## The scan-active latch (BESU) and its phased copy (CENO)

BESU tracks whether OAM parsing is in progress; CENO is BESU re-captured on
XUPY rising, giving downstream consumers an XUPY-edge-aligned copy.

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| BESU | Scan-active latch (Q=1 during Mode 2) | nor_latch | Set: CATU.Q at Mode 2 start; Reset: ASEN | Wire name `oam_parsing`. Drives the OAM-bus lock directly: mode2 = AND2(BOGE, BESU.Q) |
| CENO | XUPY-phased scan-active copy | dffr | XUPY rising | Data: BESU. Drives `oam_rendering` = AND2(`ceno_n`, mode3) and CEHA = NOT(`ceno_n`) feeding CARE |

CATU captures `RUTU AND NOT(vblank)` on the XUPY rising edge after each
scanline boundary; its Q both sets BESU and drives the scan-counter reset
chain. At scan completion, AVAP propagates through ASEN = OR2(ATAR, AVAP) and
the rising ASEN clears BESU.

**The OAM-bus lock is BESU-direct** — mode2 takes BESU.Q straight, not
through CENO. CENO's two consumers are:

1. **The Mode 3 rendering gate** — `oam_rendering` = AND2(`ceno_n`, mode3).
2. **The sprite-store save enable** — CEHA = NOT(`ceno_n`) feeds CARE =
   AND3(XOCE, CEHA, `sprite_y_match`). CENO's one-XUPY-cycle lag behind BESU
   gives the save window precise XUPY-edge alignment — and is the source of
   the first-entry staleness race catalogued in [Races](../races.md): the
   CARE-gated window has not fully opened when the very first OAM entry's
   compare result arrives.

**Reset behaviour.** BESU has no reset net of its own; it starts at 0 at
LCD-on because ASEN includes ATAR, which is 1 throughout LCD-off.

**On the first scanline of the first frame after LCD-on, BESU never sets** — the
LCD-on Mode-2-entry path bypasses CATU
([Scanline and frame timing](../scanline-frame-timing.md)). Consequences:

- the STAT Mode 2 bit reads 0 through the nominal scan window;
- the Mode 2 interrupt does not fire;
- the CARE chain never arms;
- the sprite store is not populated by the first scanline's scan.

