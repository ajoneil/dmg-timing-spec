# Read Locks

While the PPU owns OAM or VRAM, a CPU read of the locked region returns
**0xFF** — not the stored byte. The lock disables the array's bus drivers,
so the read cycle still fires but nothing drives the bus; it settles to its
keeper value. The 0xFF is a floated bus, not a driven constant.

```admonish abstract "At a glance"
- **OAM reads return 0xFF in Mode 2 and Mode 3**; **VRAM reads return 0xFF
  in Mode 3**. Mode 0 and VBlank read the stored byte.
- The lock acts on the **address / data tri-state drivers, not the read
  strobe** — `asam` (OAM) and `ropy` = NOT(`mode3`) (VRAM) gate the drivers
  off while the CPU read decode still pulses every cycle.
- The VRAM lock engages exactly at the **Mode 2→3 boundary** (`mode3`↑).
```

## What a locked read returns

| Read | Mode 0 / VBlank | Mode 2 | Mode 3 |
|------|:---:|:---:|:---:|
| OAM (`$FE00`–`$FE9F`) | stored byte | **0xFF** | **0xFF** |
| VRAM (`$8000`–`$9FFF`) | stored byte | stored byte | **0xFF** |

Measured on the full-SRAM model (dmg-sim, purpose-built `read_lock` ROM:
OAM pre-filled 0x42, VRAM 0x24). Every Mode-2 and Mode-3 OAM read and every
Mode-3 VRAM read returned 0xFF — never the pre-fill — while a Mode-2 VRAM
read returned 0x24. The VRAM byte flips 0x24→0xFF across the single read
step straddling `mode3`↑: the lock engages on the Mode-2→3 edge. Mode-0 and
VBlank reads return the stored byte because the same gates go inactive
there (below).

## Disabled drivers, not a blocked strobe

The CPU read strobe fires on every read regardless of mode (`cpu_oam_rd_n`
= NAND3(`~clk_t4`, FEXX, `ppu_rd`); the VRAM read-enable toggles each
`clk_t4`). What the lock removes is the **drive onto the bus**:

| Gate | Role | Type | Logic |
|------|------|------|-------|
| ASAM | OAM CPU-address disable | or3 | `oam_addr_cpu_n` = OR(mode2, mode3, `dma_run`) — high disables the eight OAM-address tri-state drivers |
| WAME | OAM output-enable inverter | not_x2 | `oam_oe_n` = NOT(oam_oe); OAM's output enable is steered to the PPU parser during Mode 2/3 |
| ROPY | Mode 3 inverter for VRAM enables | not_x1 | = NOT(`mode3`); high outside Mode 3, low in Mode 3 — closes the VRAM read latch via SERE → SEBY |
| SEBY | VRAM read-latch enable | not_x1 | = AND(SERE, `cpu_vram_oam_rd`), SERE = AND(`tole`, ROPY); gated low in Mode 3, so the VRAM byte is never latched onto the internal bus |

With the OAM-address drivers disabled (ASAM high) the CPU address never
reaches the OAM array; with the VRAM read latch closed in Mode 3 (ROPY low →
SEBY low) the VRAM byte never reaches the CPU bus. Either way the bus holds
its keeper value and the CPU reads 0xFF. In Mode 0 and VBlank `asam` goes
inactive and `ropy` goes high, the drivers turn back on, and the read
returns the stored byte — the mirror of the write lock, where the same
windows block the strobe ([lock boundaries](lock-boundaries.md)).
