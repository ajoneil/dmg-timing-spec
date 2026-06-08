# OAM Corruption

The widely documented bug: a CPU bus cycle whose address lands in
$FE00–$FEFF during Mode 2 (classically a 16-bit INC/DEC, whose IDU cycle
drives the register pair onto the bus) corrupts the OAM row under scan.
The mechanism is a suppressed **bitline precharge**, measured end to end
on the full-SRAM simulation model.

> **FEXX → CUFE → BYCU → COTA → WOVU → `oam_bl_precharge_n`**
>
> The $FExx address decode (FEXX) forces CUFE low for the bus-phase half
> of the M-cycle; BYCU sticks high, COTA sticks low — and COTA's
> complement WOVU *is* the OAM bitline precharge. The beat that should
> separate two scan rows never fires.

| Gate | Role | Type | Logic |
|------|------|------|-------|
| FEXX | $FExx address-range decode | not_x2 | = NOT(ROPE); ROPE = NAND2(SOHA, RYCU) |
| CUFE | $FExx/DMA bus-cycle term | oai21 | = NOT((FEXX OR `dma_run`) AND `dma_phi_n`) — low for the bus-phase half-M-cycle of any $FExx access |
| BYCU | COTA driver | nand3 | Inputs: CUFE, XUJY, AVER |
| COTA | Stage-2 capture enable / bitline-precharge source | not_x1 | = NOT(BYCU); gates the Stage-2 latches ([OAM scan](../oam-scan/counter.md)) |
| WOVU | OAM bitline precharge driver | not_x2 | `oam_bl_precharge_n` = NOT(COTA) |
| WAHE / WUJY | OAM wordline-driver precharge | not_x3 / nand2 | WUJY = NAND2(WAZO, `oam_bl_precharge_n`) — **not** suppressed by the bug |

Measured timeline (dmg-sim measurement, purpose-built `oam_corrupt_sweep`
ROM on the full `dmg_generic_sram` OAM model; INC HL with HL = $FE00
landing in LY=40's Mode 2; t = 0 at the INC's bus cycle):

| t (ns) | Event |
|---:|---|
| ~5 | FEXX↑ — $FE00 decodes |
| ~106 | the in-flight precharge beat completes normally; row 11's second wordline epoch opens |
| ~347 | CUFE↓ (the `dma_phi_n` phase gate arrives) |
| ~470 / ~590 | AVER pulses for the next scan step — **BYCU and COTA do not respond**; no precharge beat fires |
| ~593 | row 12's wordline opens onto bitlines still carrying **row 11's read charge** — the cells are overwritten |
| ~835 | CUFE↑ (bus phase ends, exactly 2 dots low) |
| ~961 | precharge beats resume; row 12's second epoch is clean — the damage is already committed |

Three structural facts fall out:

- **No write strobe is involved** — `oam_a_wr2`/`oam_b_wr2` never assert.
  An open wordline with defined (un-precharged) bitline charge overwrites
  the cells; that is SRAM physics, faithfully modelled.
- **The wordline side is untouched** — WAHE keeps its cadence and exactly
  one wordline is enabled at every instant (no multi-wordline epochs).
  Only the *bitline* precharge is gated through COTA.
- **The row granularity is 8 bytes by construction**: one wordline row
  spans 4 columns in each of the two banks.

The corrupted row is the one whose first wordline epoch falls inside the
2-dot CUFE-low window; the source data is whatever the bitlines last
carried — the previous row's read. Measured across a 6-frame ladder
stepping the INC one M-cycle per frame: each frame's VBlank readback
shows **exactly one full 8-byte row copy** (row 12 ← row 11, then
13 ← 12, … 17 ← 16), with every other OAM byte untouched.

Outside Mode 2 there are no scan epochs to corrupt — AVER carries the
scan-phase term, and the same CUFE gating during DMA coincides with the
scan being disabled entirely ([DMA](../dma/oam-bus.md)).

Community documentation catalogues further corruption at other bus/scan
alignments — first-word AND/OR *mixes* rather than the clean copy above — and
the blargg `oam_bug` suite pins them on hardware. These are the same
suppressed-precharge mechanism: where a fully-settled read resolves as a clean
copy, a partially-driven bitline resolves *bitwise*. Sweeping the row
alignment, the full-SRAM model reproduces the clean copy (single-access) and
the multi-row case (two reads, e.g. POP), but never the bitwise mix — a
digital SRAM cell cannot partially retain its old value. That mix is analog
bitline physics below the netlist's reach, so the hardware-validated community
formulas remain its reference.

```admonish note "For implementors"
Model the measured case — a $FExx-addressed CPU bus cycle in Mode 2 copies the
previous row's 8 bytes over the row whose scan begins during that cycle — and
defer to the community first-word formulas for the alignment-dependent mixes,
which capture analog behaviour the digital model does not.
```
