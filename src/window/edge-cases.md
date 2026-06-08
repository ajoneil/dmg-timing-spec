# Window Edge Cases

Three edge cases of the window trigger model ([window control](../window.md)): the inert right-edge zone, multiple activations on one scanline, and the NUKO/PANY one-dot slip.

## The right edge

**WX = 167 never fires**: PX terminates at 167 on the same dot XUGU fires,
and TADY resets PX before any further advance — no NUKO rise is possible.

**WX ∈ [~162, 166]: the cascade fires but is observationally inert.**
Measured at WX=166 (dmg-sim measurement, patched gbmicrotest `win0_a` ROM;
zero variance
across 368 firing scanlines, all values sim-dots from AVAP↑):

| Stage | Δ (dots) |
|-------|---------:|
| NUKO↑ (PX = 166) | +172.038 |
| PYCO↑ / NUNU↑ / PYNU↑ | +172.481 / +172.982 / +172.985 |
| NUNY↑ / **MOSU↑** | +172.988 / **+172.989** |
| NUKO↓ / XUGU↓ (PX = 167) | +173.040 / +173.042 |
| WODU↑ | +173.044 |
| **XYMU↑ (Mode 3 ends)** | **+173.481** |
| NOPA captures → MOSU↓ | +173.482 |

Mode 3 length is *identical* to the no-window baseline on the same trace.
The MOSU-driven NYXU pulse executes (BG counter resets, steps 1–7 of the
sequence all fire, NOPA captures), but XYMU sets ~0.5 dots after MOSU↑ and
the combinational POKY-clear switches the pipeline off before any SACU edge
can advance the fresh fetch — structurally executed, observationally inert.
The +6-dot penalty term simply does not apply once MOSU lands inside the
[WODU↑, XYMU↑] closing window.

**Right-edge window + sprite collision** (WX=166, sprite at OAM X=167,
gambatte `m2int_wxA6_spxA7` ROM, dmg-sim measurement): on steady-state
firing scanlines the window cascade adds nothing (inert as above) and the
sprite adds exactly +6.000 dots — the right-edge collapse of the
single-sprite penalty ([sprite pipeline](../sprite-pipeline/penalty-model.md)). The one-off
*first* firing scanline (LCDC.5 just enabled, NOPA still 0) measures +17.0
dots — the standard 11-dot first-sprite penalty plus the 6-dot core, induced
by the MOSU↑ NYXU pulse re-firing the fetch on the cascade's maiden
activation only.

## Multi-activation within one scanline

A mid-Mode-3 LCDC.5 1→0 write raises XOFO and clears PYNU on the same dot;
NOPA captures the cleared PYNU one ALET edge later, re-opening NUNY's gate.
PYCO and NUNU are *not* reset by XOFO — they keep propagating any NUKO pulse
regardless of LCDC.5. PYNU's nor_latch inputs are level-sensitive, so what
happens at a subsequent LCDC.5 0→1 restore depends on NUNU's state at the
XOFO↓ edge:

1. **NUNU=0 at restore** — the match fired long before the disable and has
   drained. PYNU stays 0; a new activation needs a *fresh* NUKO rise, which
   under fixed WX is impossible same-line (PX is monotonic) — the CPU must
   rewrite WX ahead of PX. This is the structurally-new "multi-fire" case:
   Mealybug `m3_lcdc_win_en_change_multiple` (WX rewritten 24 → 120 between
   the toggles) produces two independent MOSU pulses per scanline (dmg-sim
   measurement: second activation at NUKO↑ +132.013, PYNU↑ +132.996).
2. **NUNU=1 at restore** — the match fired *inside* the disabled window
   and the restore lands within the ~1-dot NUNU-high pulse
   (≈ [NUKO↑+1.0, NUKO↑+2.0]). PYNU sets
   combinationally at XOFO↓ — a **deferred completion** of the
   first activation, not a new one. Mealybug
   `m3_lcdc_win_en_change_multiple_wx` exercises exactly this: on LY=16 and
   LY=44 the restore lands inside the NUNU pulse and MOSU fires within a
   gate delay of XOFO↓; on LY=22 the match falls between the off-pulses and the standard
   sequence runs untouched (dmg-sim measurement).

**The window line counter** (VYNO → VUJO ripple) clocks on PYNU *falling*
edges. A scanline with N intra-line LCDC.5 1→0 transitions advances it N+1
times (once per disable, once at the end-of-line ATEJ). Under WX-only
rewrites with LCDC.5 held high it advances exactly once per scanline: PYNU's
set input saturates (already 1) and NUNY stays closed (NOPA holds), so a
second NUNY/MOSU pulse is impossible without a PYNU fall.

## The NUKO/PANY slip

NUKO has exactly two netlist consumers: PYCO (the capture chain above) and
**PANY** — the BG drain-detector input ([BG pipeline](../bg-pipeline/fetcher.md)). The
second consumer produces the subtlest window behaviour on the DMG.

PANY pulses high for ~1 dot at every tile-boundary drain. A NUKO pulse
landing **inside** that window splits it across RYFA's capture edge: PANY is
forced low at NUKO↑, released at NUKO↓, and then falls naturally one dot
late — so RYFA captures the late half, and the entire SEKO → TEVO → NYXU
chain slips by one dot for that boundary. The slip delays both the
BG-shifter parallel load (unconditionally — this branch has no window
gating) and the window tile-X increment (only when `in_window`=1).

- **WX-rewrite form** (Mealybug `m3_wx_4_change` family): a mid-Mode-3 WX
  rewrite that lands PX==WX inside a PANY window produces the documented
  pair-shuffle pattern at recurring tile boundaries. Measured: three
  perturbed scanlines per batch with NUKO arriving in the last sub-dot of
  the PANY window; pulses outside the window do nothing; and the simulated
  render is bit-identical to the hardware reference — the mechanism is fully
  on-die (dmg-sim measurement).
- **Armed-but-disabled form** (no rewrite needed): NUKO is gated by REJO but
  **not by LCDC.5**. With the window armed then disabled (REJO=1, LCDC.5=0)
  and WX ≡ 7 (mod 8) placing PX==WX at a tile boundary, the natural sweep
  fires the same slip — the BG shifts one pixel right from that column to
  the line's end, every scanline until VBlank clears REJO, while the window
  itself never renders (PYNU is held reset, so the `wx_clk` branch stays
  silent). Measured end to end: the boundary's PANY/RYFA/SEKO/TEVO/NYXU all
  slip exactly +1.000 dot (dmg-sim measurement).

```admonish warning "Pitfall: the slip needs no rendered window"
A model that keys the slip on "window active" misses the
armed-but-disabled form entirely — REJO alone arms NUKO. Disabling LCDC.5
does not disarm the coupling; only VBlank's REPU does.
```

