# Appendix B: Timing Constants

Quick reference for the numeric constants used throughout the book. Each value is
developed in context in the linked chapter; this table is a lookup aid, not a
substitute for the surrounding mechanism.

## Clock and timing

| Unit | Value | Definition |
|---|---|---|
| Dot / T-cycle | ≈238.4 ns (dmg-sim: 244,000 ps) | One full master clock cycle (rise + fall) — see [Methodology](methodology.md#how-to-read-the-numbers) |
| M-cycle | 4 dots | CPU machine cycle |
| ATAL half-cycle | 0.5 dots | Interval between consecutive master-clock edges |

## Mode 3 structure

Baseline: SCX=0, no sprites, no window. Developed in
[BG pipeline](bg-pipeline/startup-fine-scroll.md) and [Mode transitions](mode-transitions.md).

| Interval | Dots |
|---|---|
| AVAP → first SACU (startup pipeline) | 7.026 |
| AVAP → WODU (Mode 3 core) | 173.045 |
| WODU → XYMU (VOGA → WEGO → XYMU) | 0.436 |
| AVAP → XYMU (Mode 3 total) | 173.481 |
| SACU rising edges per Mode 3 | 167 (invariant with SCX) |

## Fine scroll

Developed in [BG pipeline](bg-pipeline/startup-fine-scroll.md).

| Constant | Value |
|---|---|
| Cost per `SCX & 7` increment | Exactly 1 dot (ROXY suppression at startup) |
| First SACU at SCX=N | 7 + (N & 7) + 0.026 dots after AVAP |
| Mode 3 core duration at SCX=N | 173 + (N & 7) + 0.045 dots |
| SACU count | 167, invariant with SCX |

## NYXU and the BG fetch counter

NYXU = NOR3(AVAP, MOSU, TEVO). The BG fetch counter counts 0→5 per 8-dot tile;
TEVO resets it on every tile boundary. Developed in
[BG pipeline](bg-pipeline/fetcher.md) and [Window control](window.md).

| Event | Value |
|---|---|
| AVAP → NYXU pulse width | 0.492 dots |
| TEVO → NYXU pulse width | 0.495 dots |
| MOSU → NYXU pulse width | 0.503 dots |
| TEVO firings per scanline (normal BG) | 21 at SCX=0; 22 at SCX&7 ∈ {1..7} (the tail TEVO closing tile 21) |
| NYXU assertions per scanline | 22 at SCX=0, 23 at SCX&7 ∈ {1..7} (1 AVAP + TEVO), +1 MOSU if window |

## Sprite fetch

Trigger chain: TEKY → SOBU (1 DFF) → RYCE → TAKA (combinational, ~800 ps from
SOBU). SUDA is a separate DFF on the next ALET falling edge. The sprite fetch
counter (TOXE/TULY/TESE) counts 0→5 over 5 dots. Developed in
[Sprite pipeline](sprite-pipeline/fetch-machine.md).

| Event | Value |
|---|---|
| TEKY↑ → TAKA↓ (fetch freeze) | 6.010 dots, zero variance (gate-resolved) |
| Net sprite penalty | exactly 6 dots (6 suppressed SACU edges) |
| SACU resume after TAKA↓ | +1.024 dots (next ALET falling edge) |

## Clock-tree phase relationships

Developed in [Mode transitions](mode-transitions.md) and
[Register writes](registers/writes.md).

| Relationship | Value |
|---|---|
| VOGA capture after WODU↑ | 0.435 dots (same-dot ALET rising edge) |
| WODU → VOGA → WEGO → XYMU chain | 0.436 dots |
| CUPA pulse width | 1.493 dots (T3–T4) |
| VID_RST (XODO) deassertion after CUPA↑ on LCDC.7=1 write | 3,173 ps |

All PPU dividers (WUVU, VENA, TALU, XUPY, BOGA) are held at 0 while VID_RST is
asserted; counting starts from zero on deassertion. WOSU is a sampler, not a
counted divider; it tracks WUVU with a half-dot delay.

## STAT interrupt

Developed in [STAT interrupts](stat-interrupts.md).

| Relationship | Value |
|---|---|
| WODU → LALU propagation | 2,637 ps (0.011 dots, combinational) |
| STAT Mode 0 interrupt fires before the Mode 3→0 transition (XYMU clear) | 0.425 dots |

## Palette writes

Developed in [Palette latches](registers/palette-latches.md).

| Event | Value |
|---|---|
| Palette dlatch update after the write-dot SACU falling | ~15.6 ns (data-bus settling — structural) |
| LCD output pins update after SACU falling | 1,019–1,711 ps |

On the write dot, the shifted pixel uses the OLD palette value; the new value
first affects the next SACU rising edge.

## XYMU polarity

- XYMU = **0** during Mode 3 (active rendering)
- XYMU = **1** during Modes 0, 1, and 2 ("not rendering" — active-high meaning)

## Post-boot state

Full table in [Post-boot state](post-boot.md). Headline: LY=0, LX=98 (mid-Mode-0
of the first scanline, 15 M-cycles before LINE_END), XYMU=1, BESU=0, scan counter=39.
Scan counter bit order: YFEL (LSB) → WEWY → GOSO → ELYN → FAHA → FONY (MSB).
