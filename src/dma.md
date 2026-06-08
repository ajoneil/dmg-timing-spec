# OAM DMA

OAM DMA copies 160 bytes from a source page to OAM at one byte per M-cycle.
The behavioural surface — timing, the source-region quirks, the "CPU should
stay in HRAM" rule — is documented in
[gb-ctr](https://gekkio.fi/files/gb-docs/gbctr.pdf). What this chapter adds is
the arbitration machinery: who owns each bus during DMA, what the PPU's scan
and fetch hardware does in the meantime, what happens at the engage/release
boundaries to the dot, and what the CPU actually observes when it touches a
bus DMA is driving.

```admonish abstract "At a glance"
- During DMA the OAM address bus has **exactly one driver: DMA**. The
  PPU's scan and fetch machinery keeps running — only its bus drivers
  tri-state — which is the strikethrough mechanism.
- The scan chain's capture enable is **fully gated off** during a
  DMA + Mode-2 overlap: Stage 1 holds the pre-DMA byte-pair, producing
  phantom sprites when the held Y matches.
- DMA **pauses during HALT** (its clock is pinned) but `dma_run` keeps
  owning the buses for the whole span.
- Engage takes the bus on the **same master-clock edge** as `dma_run`↑;
  release waits at most one scan-cadence period (≤2 dots).
- A CPU access to a DMA-driven bus has its **address replaced at the
  pads**; HRAM is exempt. Sources $FE/$FF read WRAM — the echo region
  extended one page.
```

## What's where

| Page | Covers |
|------|--------|
| [The OAM bus under DMA](dma/oam-bus.md) | Four-source address arbitration, the per-T-cycle byte transfer, the strikethrough effect, the two-bank SRAM, and the scan-chain capture hold |
| [Engage, release, and HALT](dma/boundaries.md) | The FF46→`dma_run` latency, DMA-start and DMA-end cadences against Mode 2, the dot-80 visibility boundary, and the HALT pause |
| [The source bus and conflicts](dma/source-bus.md) | Address-pad MUXing, CPU read/write conflicts byte by byte, the HRAM exemption, and the $FE/$FF source fold into WRAM |
