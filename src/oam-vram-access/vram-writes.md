# VRAM Writes

VRAM is off-chip; the CPU's address, data, and write-strobe outputs reach the
VRAM SRAM through dedicated pads (`ma_pad[12:0]`, `md_pad[7:0]`, `mwr_n_pad`,
`mrd_n_pad`, `mcs_n_pad`) whose tri-state enables are gated on mode 3. There
is no on-chip "VRAM write enable latch" analogous to OAM's WYJA; the Mode 3
lock is enforced by tri-stating the address bus and write strobe so the
off-chip SRAM never sees a valid write cycle.

| Gate | Role | Type | Inputs | Notes |
|------|------|------|--------|-------|
| ROPY | Mode 3 inverter for VRAM enables | not_x1 | mode3 | ROPY = NOT(mode3); high outside Mode 3 |
| XANE | VRAM address tri-state-disable source | nor2 | `vram_to_oam`, mode3 | XANE=1 when neither VRAM-sourced DMA nor Mode 3 holds — the CPU may drive the address pads |
| XEDU | VRAM address tri-state enable | not_x2 | XANE | XEDU = NOT(XANE); drives the `not_if0` ena_n of the `ma_pad[*]` address pads |
| TOLE | CPU data-bus enable stage 1 | combinational | CPU bus enables | Carries CPU-side write-window timing |
| SERE | CPU data-bus enable stage 2 | and2 | TOLE, ROPY | Active only when CPU drive is enabled AND outside Mode 3 |
| ROCY / RAHU / SAZO / REVO | Per-bit data-pad tri-state enable chain | combinational | SERE + per-bit fan-out | Drive `not_if0` ena_n on `md_pad[*]` |
| SOHY | Write-strobe driver | nand2 | TYJY (CPU bus-phase write enable), SERE | Drives `mwr_n_pad` low when both arms hold |
| TAXY / SYSY / RACO / TUTO | CPU-bus-phase strobe assembly | combinational | T1T2_n / SOTO_n / TYJY chain | Generate the CPU-side write window feeding SOHY |
| `mwr_n_pad` | Off-chip write strobe | tri-state pad output | SOHY | Active-low; visible to the off-chip VRAM SRAM |
| `ma_pad[12:0]` | Off-chip address pads | tri-state pad outputs | XEDU enable; CPU bus address | Driven when XANE=1 |
| `md_pad[7:0]` | Off-chip data pads | tri-state pad outputs | SERE chain enable; CPU bus data | Driven when SERE=1 |

**Lock-window behaviour.**

- **Outside Mode 3** — ROPY=1, XANE=1, XEDU=1 → address pads driven from
  the CPU bus; SERE=1 (when CPU drive is active) → data pads driven and
  SOHY may pulse `mwr_n_pad` low. The off-chip SRAM sees a valid write
  cycle.
- **During Mode 3** — ROPY=0 → SERE=0 (data pads tri-stated) and SOHY held
  high (no strobe); independently XANE=0 → XEDU=0 (address pads
  tri-stated). The SRAM never sees the write: both address and strobe are
  absent, so there is no "missed write" latch to need — the strobe simply
  never asserts.

**CUPA is not involved here.** Unlike the OAM path — whose write window is
the CUPA pulse via `ppu_wr` — the VRAM-write window is set by the SM83
bus-phase signals (T1T2_n, SOTO_n, TYJY) feeding TAXY/SYSY/SOHY. The mode-3-gated tri-state enables are
the lock. If they are stable across the CPU's write window, the strobe
propagates cleanly; if mode 3 transitions *during* the window, the address
pads and strobe tri-state mid-cycle, truncating the write at the pins.
What the external SRAM does with a truncated cycle is off-die and out of
scope.
