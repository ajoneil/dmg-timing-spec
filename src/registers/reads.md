# Reads

CPU reads of PPU registers use a purely combinational path from the underlying
DFF Q outputs to the CPU data bus, gated by address decode and the CPU's
ungated read signal `ppu_rd`. There is no PPU-side read strobe analogous to
CUPA ([writes](writes.md)); the CPU latches the bus on its own schedule via
the SM83-internal `data_phase` signal.

```admonish abstract "At a glance"
- Reads are **purely combinational** — no PPU-side strobe, no latch
  between a register's DFFs and the bus; the CPU samples at its own
  `data_phase`.
- Every $FF4x register reads through the same decode → AND2(`ppu_rd`,
  select) → tri-state chain, netlist-enumerated for all twelve $FF4x
  registers (the eleven PPU registers plus DMA $FF46): ten on NOT_IF0
  drivers, DMA on NOT_IF1, STAT split across both.
```

## The read path

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| `ppu_rd` | CPU read assert | not_x4 (buffered) | combinational | Buffer-chained from `cpu_rd`: `ajas` = NOT(`cpu_rd`); `ppu_rd` = NOT(`ajas`). No phase gating; asserts throughout the CPU's read cycle. |
| `ff4x` | $FF4x range high-byte decode | not_x1 | combinational | Active-high when address[15:4] matches $FF4. Shared input to every $FF4x register address decoder. |
| `wyle` | $FF44 address decode | nand5 | combinational | Inputs: `ff4x` + low-nibble inversions (`xola`, `xeno`, `walo`, `xera`). Active-low when address = $FF44. |
| `wofa` | $FF41 address decode | nand5 | combinational | Inputs: `ff4x` + low-nibble inversions (`wado`, `xeno`, `xusy`, `xera`). Active-low when address = $FF41. Orthogonal to `wyle` on address bit 0 (`wado` = a0; `xola` = NOT(a0)). |
| `ff44` | $FF44 active-high | not_x2 | combinational | = NOT(`wyle`). Drives `wafu`. |
| `ff41` | $FF41 active-high | not_x1 | combinational | = NOT(`wofa`). Drives `tobe`. |
| `wafu` | $FF44 read enable | and2 | combinational | Inputs: `ppu_rd`, `ff44`. Inverted by VARO for the drivers' active-low `ena_n`: 8 NOT_IF0 drivers from the LY DFFs (MUWY…LAFO) to CPU data bus d[0..7]. |
| `tobe` | $FF41 read enable | and2 | combinational | Inputs: `ppu_rd`, `ff41`. Enables the NOT_IF1 drivers for STAT bits 0–2; its complement VAVE gates the NOT_IF0 drivers for bits 3–6 (the family split below). |

The $FF44 (LY) read, representative of the pattern:

1. The CPU asserts `cpu_rd` with address $FF44 on its port.
2. `ppu_rd` follows `cpu_rd` through two inverter stages; no phase gating.
3. The address decoders fire: `ff4x` (high-byte match) combines with the
   low-nibble decode `wyle` to drive `ff44` active-high.
4. `wafu` = AND2(`ppu_rd`, `ff44`) enables the 8 tri-state drivers from the LY
   DFFs (through VARO, the drivers' active-low `ena_n`).
5. Each NOT_IF0 driver presents its bit's current Q output onto the CPU data
   bus d[0..7] — a single NOT gate plus tri-state driver, all combinational.
6. The CPU samples `cpu_port_d` (connected to d[] via transmission gates) at
   the SM83-internal `data_phase` — which is the phase generator's
   NOT(ATYP) window, clock-enable gated: BELU = NOR2(ATYP, `clk_ena_n`),
   inverted by BYRY and rebuffered by BEVA. It speaks the same AFUR-ring
   vocabulary as the write path's `clk_t4`
   ([writes](writes.md)); its falling edge — the
   CPU's capture point — is the ATYP rise at the M-cycle boundary.

## The STAT family split

$FF41 (STAT) splits across **both driver families** (netlist-derived): the
live status bits read through NOT_IF1 drivers (enabled by
TOBE = AND2(`ppu_rd`, ff41)), the interrupt-enable bits through NOT_IF0
drivers (enabled by VAVE = NOT(TOBE)):

| Bit | Driver | Source | Family |
|---|---|---|---|
| 0 | TEBY | SADU | NOT_IF1 (TOBE) |
| 1 | WUGA | XATY | NOT_IF1 (TOBE) |
| 2 | SEGO | `rupo_n` | NOT_IF1 (TOBE) |
| 3 | PUZO | `roxe_n` | NOT_IF0 (VAVE) |
| 4 | POFO | `rufo_n` | NOT_IF0 (VAVE) |
| 5 | SASY | `refe_n` | NOT_IF0 (VAVE) |
| 6 | POTE | `rugu_n` | NOT_IF0 (VAVE) |

The split matches the [driver-cell distinction](../cpu-visible-boundaries.md):
the transition-prone status bits carry the bus-settling x-window; the
per-frame-stable enable bits read like LY, with none.

## The full register read map

Every $FF4x PPU register follows the same chain, netlist-enumerated for
all twelve. Per register: a nand5
address decode (`ff4x` plus the four low-nibble address lines, true or
inverted to match the digit) feeds a not_x2 to make the active-high
select; an AND2 with `ppu_rd` forms the read enable; a not_x1 inverts it
for the drivers' active-low `ena_n`; eight NOT_IF0 drivers put the
register on the bus. Each driver's input is the latch's *inverted* output,
so the bus receives true polarity. The selects are shared with the write
path — the same cell feeds the write enable's AND2 with CUPA
([writes](writes.md)).

| Register | Decode → select | Enable | `ena_n` | Drivers d0 → d7 |
|---|---|---|---|---|
| LCDC $FF40 | WORU → VOCA | VYRE | WYCE | WYPO XERO WYJU WUKA VOKE VATO VAHA XEBU |
| STAT $FF41 | WOFA → VARY | TOBE | VAVE (bits 3–6) | TEBY WUGA SEGO PUZO POFO SASY POTE (split, no bit 7) |
| SCY $FF42 | WEBU → XARO | ANYP | BUWY | WARE GOBA GONU GODO CUSA GYZO GUNE GYZA |
| SCX $FF43 | WAVU → XAVY | AVOG | BEBA | EDOS EKOB CUGA WONY CEDU CATA DOXE CASY |
| LY $FF44 | WYLE → XOGY | WAFU | VARO | VEGA WUVA LYCO WOJY VYNE WAMA WAVO WEZE |
| LYC $FF45 | WETY → XAYU | XYLY | WEKU | RETU VOJO RAZU REDY RACE VAZU VAFE PUFY |
| DMA $FF46 | WATE → XEDA | MOLU | NYGO → PUSY (`ena`) | POLY ROFO REMA PANE PARE RALY RESU NUVY |
| BGP $FF47 | WYBO → WERA | VUSO | TEPY | RARO PABA REDO LOBE LACE LYKA LODY LARY |
| OBP0 $FF48 | WETA → XAYO | XUFY | XOZY | XARY XOKE XUNO XUBY XAJU XOBO XAXA XAWO |
| OBP1 $FF49 | VAMA → TEGO | MUMY | LOTE | LAJU LEPA LODE LYZA LUKY LUGA LEBA LELU |
| WY $FF4A | WYVO → VYGA | WAXU | VOMY | PUNU PODA PYGU LOKA MEGA PELA POLO MERA |
| WX $FF4B | WAGE → VUMY | WYZE | VYCU | LOVA MUKA MOKO LOLE MELE MUFE MULY MARA |

Two exceptions to the NOT_IF0 rule:

- **STAT** splits across both families (above).
- **DMA ($FF46)** reads through eight **NOT_IF1** drivers — the same
  active-high-enable family as STAT's live bits — via a double inversion
  (MOLU → NYGO → PUSY) and fed by `dma_a15…a8`, the DMA source-page
  register. The family membership has no observable consequence: those
  DFFs change only on an FF46 write, which cannot land inside an FF46
  read M-cycle, so no transition can straddle a read. The
  [x-window machinery](../cpu-visible-boundaries.md) stays exclusive to
  STAT bits 0–2.

## Timing implications

- **No within-dot ordering for reads.** Unlike writes — which land at a
  specific dot position gated by CUPA — reads are continuously enabled while
  `ppu_rd` is asserted and the address decoder matches. A read is not an
  ordered event in the PPU's within-dot sequence.
- **No PPU-side sample point.** What the CPU reads is the current DFF state at
  the instant its `data_phase` fires. Any update to the underlying DFFs (an LY
  reset via LAMA, a ROPO capture on TALU rising) propagates combinationally
  through the inverter and tri-state driver within gate delay — well under a
  dot.
- **CPU-side sampling phase.** `data_phase` gates the CPU's internal capture
  of `cpu_port_d`. What the CPU observes when a PPU transition lands inside
  the read M-cycle is worked through, case by case, in
  [CPU-visible timing at mode boundaries](../cpu-visible-boundaries.md).

The structural takeaway: **there are no latches or DFFs between a PPU
register's DFFs and the CPU data bus on the read side** — the PPU drives the
bus combinationally while `ppu_rd` is asserted, and the CPU latches at its own
`data_phase`.
