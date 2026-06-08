# AVAP and the 80 Dots

At scan completion, a rising-edge detector formed by BYBA and DOBA produces
AVAP — a ~0.483-dot pulse that triggers the Mode 2→3 transition.

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| AVAP | Mode 2→3 trigger | not_x2 | Input: BEBU | Wire name `stop_oam_parsing`. Primary here; consumed in [Mode transitions](../mode-transitions.md) |
| BEBU | Scan-complete aggregate | or3 | Inputs: DOBA, BALU, `byba_n` | The third input is BYBA's **Q_n** pin, not Q |
| BYBA | Scan-done flag | dffr | XUPY rising | D: FETO. Reset: BAGY (active-low) |
| DOBA | ALET-clocked scan-done-prev | dffr | ALET rising | D: BYBA.Q. Reset: BAGY |
| BALU | OAM-parse reset (active-high) | not_x1 | Input: ANOM | |
| BAGY | BYBA/DOBA reset driver | not_x1 | Input: BALU | ANOM buffered through two inverters |

AVAP = NOT(BEBU) fires when BYBA has just captured scan-done (`byba_n`=0) AND
DOBA hasn't yet caught up (DOBA=0) AND the parse reset is released (BALU=0).
BYBA captures on XUPY rising (= ALET falling); DOBA captures BYBA half a dot
later on ALET rising. The AVAP pulse runs 0.483 dots — just under the
0.487-dot BYBA.Q↑→DOBA.Q↑ capture span it tracks (the small skew is the
BEBU/AVAP gate delay).

**BALU is structurally dominated.** When BALU rises, BAGY falls and forces
BYBA into reset, driving `byba_n` to 1 within 864 ps (dmg-sim measurement).
BALU is therefore never the sole BEBU contributor, and the OR3 reduces
observationally to **AVAP = BYBA AND NOT(DOBA)** (given BYBA's clearing at
the scanline-boundary reset).

## Measured cascade: counter=39 → AVAP

From the XUPY edge that advances the counter 38→39 (dmg-sim measurement,
gbmicrotest `int_hblank_nops_scx0`, zero variance across 1,721 scanlines):

| Stage | Δ (ps) | Dots | Type | Mechanism |
|-------|--------|------|------|-----------|
| Counter 38→39 (XUPY rising N) | 0 | 0.000 | Clocked | Only YFEL toggles; upper bits hold |
| YFEL.Q settled | +296 | 0.001 | Combinational | DFF Q delay |
| FETO↑ | +2,602 | 0.011 | Combinational | AND4 of settled ripple Qs |
| XUPY rising N+1 | +488,000 | 2.000 | Clocked | Next XUPY edge |
| **BYBA.Q↑** | +488,902 | 2.004 | Clocked | Captures FETO=1, 902 ps after the edge |
| BEBU↓ | +489,558 | 2.006 | Combinational | `byba_n`↓ collapses the OR3 |
| **AVAP↑** | +490,679 | 2.011 | Combinational | Mode 2→3 trigger |
| BESU↓ | +491,385 | 2.014 | Latch reset | ASEN clears the scan-active latch |
| ALET rising | +606,906 | 2.487 | Clocked | First ALET↑ after BYBA↑ |
| DOBA.Q↑ | +607,790 | 2.491 | Clocked | Captures BYBA=1 |
| BEBU↑ | +608,036 | 2.492 | Combinational | DOBA restores BEBU |
| AVAP↓ | +608,613 | 2.494 | Combinational | Pulse ends — width 117,934 ps = 0.483 dots |

The one-XUPY-cycle latency from counter=39 to BYBA's capture is structural
(BYBA is a clocked DFF, not a combinational path): FETO settles 2,602 ps
after edge N — a comfortable margin against the 488,000 ps period — and the
static race analysis tags the whole BESU/CATU/BYBA/YFEL/FETO family
`static` accordingly.

Supporting pulse measurements (same scenario, zero variance): BALU pulse
1.008 dots per scanline at an exactly-456-dot period; BYBA Q delay 902 ps
after XUPY; DOBA Q delay 884 ps after ALET.

## Where the 80 dots come from

Mode 2's duration (BESU=1, CATU↑ to AVAP↑) decomposes exactly:

> 2 dots (CATU↑ → first counter tick) + 76 dots (counter 1→39 over 38 XUPY
> cycles) + 2 dots (FETO → BYBA capture) = **80 dots**

The preceding RUTU↑ → CATU↑ capture latency is one dot — measured at
243,059 ps (0.996 dots), picosecond-identical across 431 steady-state
boundaries (dmg-sim measurement) — and sits *before* Mode 2:
total scanline boundary (RUTU↑) to AVAP↑ is 81 dots.
