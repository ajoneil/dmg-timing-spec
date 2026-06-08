# Engage, Release, and HALT

A transfer has three boundaries: the **engage** after an FF46 store, the
**release** back to the scan when the last byte commits, and the **pause**
across HALT. Each interacts with the Mode-2 scan differently — engage is
edge-immediate, release waits for the scan cadence, and HALT freezes the
byte counter without surrendering the bus.

```admonish abstract "At a glance"
- FF46-store → `dma_run`↑ is a fixed engine constant: **1.495 M-cycles
  ≈ 6 dots**, M-cycle-phase quantised by MATU's clock.
- **Engage is immediate** — the lock takes the bus and suppresses captures
  on the same master-clock edge as `dma_run`↑.
- **Release waits for the scan cadence** — the first post-DMA capture
  fires within 2 dots, and only if BESU is still set (before dot 80).
- The dot-80 boundary is sharp: DMA ending after AVAP defers OAM
  visibility one full line.
- **HALT pauses DMA structurally** — `dma_phi` is pinned, the byte counter
  freezes, but `dma_run` keeps owning the buses for the whole span.
```

## DMA-start during Mode 2: engage cadence

A `dma_run`↑ inside Mode 2 freezes the scan partway through the line. Two
things fix where it cuts off: the FF46-store → `dma_run`↑ latency, and the
engage cascade's alignment to the scan cadence.

**FF46-store → `dma_run`↑ latency.** The arm chain:

> **LAVY → NOLO → MYTE → LARA/LOKY → MATU**
>
> LAVY = AND2(`ppu_wr`, ff46) detects the store; MYTE captures it on
> `dma_phi_n`; the LARA/LOKY latch pair arms; MATU (clocked by `dma_phi`)
> raises `dma_run`.

Because MATU is clocked by `dma_phi`, `dma_run`↑ is M-cycle-phase
quantised: measured LAVY↑ → `dma_run`↑ is a fixed engine constant of
**1,459,029 ps = 1.495 M-cycles ≈ 6 dots**, identical across all 1,154
DMAs in the capture, always landing at `dma_phi` = 1. A one-M-cycle shift
of the FF46 store shifts `dma_run`↑ by exactly one M-cycle.

**Engage cascade.** Opposite polarity to the release chain (next
section), and — unlike release — with no XOCE wait:

| Stage | Δ from `matu.q`↑ | Source |
|-------|---|---|
| BOGE↓ | +2,741 ps | NOT(`dma_run`) |
| mode2↓ | +3,688 ps total | AND2(BOGE, `oam_parsing`) → 0 |
| `oam_addr_parse_n`↑ | **+4,507 ps** | APAR — parse driver tri-states; DMA takes the bus |
| AJEP | already high — no edge | NAND2(mode2, XOCE); the quantised engage always lands at XOCE=0 |
| ODL held low | while AJEP = 1 | BODE |

The +4,507 ps is constant across every mode2=1 engage in the stream
(dmg-sim measurement, gambatte `late_sp00x_2`). The lock takes the bus and
suppresses captures on the **same master-clock edge** as `matu.q`↑ —
engage is immediate, unlike release.

**The last captured entry.** The scan captures one byte-pair per
XOCE↑ with mode2=1; entry 0 is captured at BESU↑ + 0.49 dots. The scan
tick at dot 2k captures sprite k, so with `dma_run`↑ at Mode-2 dot D,
sprites 0..⌊D/2⌋ are captured this line and ⌊D/2⌋+1..39 are blocked,
holding the pre-engage Stage-1 pair. A 4-dot shift of the FF46 store moves
the boundary by ±2 sprites — the structure the `late_sp*` partner ROMs
bracket.

**Sprite 0 is always captured — the early-engage sub-case is unreachable.**
The engage instant is phase-locked twice over (dmg-sim measurement):

- `dma_run`↑ sits at a fixed phase of the CPU clock grid (+973,143 ps past
  `clk_1mhz`↑; single population, n=1,154);
- the line boundary CATU↑ sits at its own fixed phase (+610,111 ps,
  n=5,468), and the PPU grid's phase against the CPU clock is itself
  pinned by the M-cycle-quantised LCD-on write.

The only reachable engage positions are therefore Mode-2 dots
**1.488 + 4k**: no program can place a lock inside the first ~0.5 dots of
Mode 2, and the entry-0 capture (BESU↑ + 0.49 dots) always completes a
full dot before the earliest possible lock.

## DMA-end during Mode 2: release cadence

A `matu.q`↓ (DMA end) inside Mode 2 re-enables the scan capture — but not
on the same edge; the next ODL pulse waits on the scan cadence:

| Stage | Δ from `matu.q`↓ | Source |
|-------|---|---|
| BOGE↑ | +1,941 ps | NOT(`dma_run`) |
| mode2↑ | +3,723 ps total | AND2(BOGE, `oam_parsing`) — requires BESU.q=1 |
| next XOCE↑ edge | 0–2 dots | free-running scan cadence |
| AJEP↓ | +105 ps after that XOCE↑ | NAND2(mode2, XOCE) |
| ODL↑ | +2,547 ps after AJEP↓ | BODE |

Every stage is a few nanoseconds; the only multi-dot wait is for the next
XOCE edge. **Upper bound: the first post-DMA capture fires within 2 dots
of DMA end.**

The interlock that matters is BESU: the release requires BESU.q = 1 at the
`matu.q`↓ edge. BESU's set (CATU, at the scanline boundary) is independent
of DMA, and its reset (AVAP) fires at scan completion — the line's dot
80 — regardless of DMA, because the scan counter free-runs through the
overlap. So:

- **DMA ends before dot 80**: the chain re-enables at the next XOCE↑, and
  the *remaining* sprites of this line's scan capture from the
  freshly-written OAM. Earlier entries hold the pre-DMA Stage-1 pair.
- **DMA ends after dot 80 (post-AVAP)**: BESU is already clear, mode2
  stays 0, and no captures occur for the rest of this line — new OAM
  contents first become visible at the *next* line's Mode 2.

The boundary is sharp at dot 80, with at most one XOCE period (2 dots) of
resolution. Both regimes are directly captured in the DMA re-trigger
stream (dmg-sim measurement, gambatte `late_sp00x_2`): a release with
BESU.q=1 resumes captures within ~1 dot; a release ~280 dots after AVAP
produces zero captures for the remainder of the line.

```admonish note "For implementors"
This boundary is exactly what the gambatte `late_sp*` partner-test pairs
bracket: variants whose DMA ends ≤ dot 80 extend Mode 3 via the
newly-captured sprites; the 4-dot-later partners miss the window and defer
visibility one full line, landing the test's STAT sample in a different
mode. Emulators that defer OAM visibility to "the next line" unconditionally
— or fail to advance the scan counter during DMA — get the in-time partner
wrong; switching the PPU's OAM view atomically at DMA end reproduces both.
```

## DMA pause during HALT

The DMA byte counter (MATU) is clocked by `dma_phi` = NOT(`data_phase`),
and `data_phase` is held LOW throughout HALT
([HALT and EI](../halt-ei.md)). The consequence is structural: **while
halted, `dma_phi` is held HIGH, MATU has no clock edges, and the byte
counter does not advance.** Any DMA in flight at HALT entry pauses on the
byte being fetched and resumes on the first `dma_phi`↑ after HALT exit
(~+0.5 dots after the wake CLK9↑). The same holds for any state that pins
`data_phase` low (reset, STOP).

`dma_run` does **not** fall during the pause — both its set and reset
transitions require `dma_phi` clocks — so all `dma_run`-gated arbitration
(OAM bus and source bus alike) continues to assert "DMA owns the bus" for
the entire HALT span, even though no bytes move.

```admonish info "Measured: a 17 ms HALT contributes zero bytes"
End to end (dmg-sim measurement, gambatte
`oamdmasrc80_halt_lycirq_read8000` ROM): a VRAM-source DMA advances 13
bytes, HALT entry freezes `dma_a` at 0x0D, the counter stays frozen across
a ~17 ms HALT (≈17,900 M-cycles — zero advance), and on wake it resumes at
one byte per M-cycle. The post-HALT read of the source page lands at byte
index 0x1F = source[$801F] = $81, matching the hardware-verified `out81`
expectation. The HALT span contributes exactly zero to the byte index —
which is why the `_lycirq` (17 ms HALT) and `_m2irq` (~1 ms HALT) variants
share the same expected value.
```
