# The Pixel Clock

Two tightly coupled circuits meter every pixel of every scanline: the PX
counter tracks horizontal position, and CLKPIPE (SACU) — the pixel pump —
decides on each dot whether the pipeline shifts at all. Every Mode 3 penalty
in this book is, mechanically, a CLKPIPE halt.

```admonish abstract "At a glance"
- PX is an 8-bit hybrid synchronous/ripple counter clocked by SACU; it
  always starts at 0 — fine scroll never masks PX.
- Terminal count is **167**, decoded by XUGU from five bits; with no sprite
  match this raises WODU, the H-Blank condition.
- SACU = OR2(ROXY, SEGU) — a 63.8-ge-deep gated clock with **four halt
  conditions**: fine scroll, data-not-ready, window activation, and
  sprite-match/H-Blank.
- 167 SACU rising edges per Mode 3, invariant with SCX.
```

## The pixel counter (PX)

PX is an 8-bit hybrid synchronous/ripple counter clocked by SACU. The lower
nibble (XEHO, SAVY, XODU, XYDO) is clocked directly by SACU with XOR carry
logic — all four bits update on the same SACU rising edge. The upper nibble
(TUHU, TUKY, TAKO, SYBE) clocks on TOCA = NOT(XYDO), rippling off bit 3.

| Gate | Role | Type | Clock | Notes |
|------|------|------|-------|-------|
| XEHO | PX bit 0 | dffr | SACU | Toggle |
| SAVY | PX bit 1 | dffr | SACU | D = XOR(SAVY, XEHO) |
| XODU | PX bit 2 | dffr | SACU | D = XOR(XUKE, XODU); XUKE = AND2(SAVY, XEHO) |
| XYDO | PX bit 3 | dffr | SACU | D = XOR(XYDO, XYLE); XYLE = AND2(XUKE, XODU) |
| TOCA | Upper-nibble clock | not | XYDO | |
| TUHU / TUKY / TAKO / SYBE | PX bits 4–7 | dffr | TOCA | Toggle / XOR carry, mirroring the lower nibble |
| XUGU | Terminal count decode | nand5 | SYBE, SAVY, TUKY, XEHO, XODU | Fires at PX = 167 |
| TADY | VOGA + PX-counter reset | nor2 | TOFU, ATEJ | Shared with VOGA's reset and (via ATEJ) the scan-counter chain |

PX is held in reset during Mode 2 and video reset and released when Mode 3
begins. **PX starts at 0 regardless of fine scroll** — fine scroll is pure
clock suppression, never PX masking.

**Terminal count 167.** XUGU monitors PX bits {0, 1, 2, 5, 7}: all five high
= 0b10100111 = 167 (bits 3, 4, 6 are 0 at that count, so the partial decode
is exact at the terminal). XUGU=0 → XANO=1, and with no sprite match
(XENA=1), WODU goes high — the H-Blank condition
([STAT interrupts](../stat-interrupts.md)).

```admonish info "Measured: 167 SACU edges per Mode 3, invariant with SCX"
Fine scroll delays the *first* SACU; it never changes the count (dmg-sim
measurement). Of the 168 PX states, 160 drive visible pixels: CP is gated by
PX ≥ 9 through TOBA, so the first 8 CLKPIPE cycles shift through the
register without reaching the glass, CP fires 159 times at PX 9–167, and the
POVA path captures the 160th pixel at end-of-line
([LCD output](../lcd-output.md)).
```

## CLKPIPE / SACU

SACU — the pixel pump — is an OR2 at depth 63.8 ge with fan-out 53, clocking
52 cells across the shifters, the sprite attribute pipes, and PX. Its gate
chain (`SACU ← SEGU ← TYFA ← VYBO`) and the four halt inputs are below.

| Gate | Role | Type | Inputs | Notes |
|------|------|------|--------|-------|
| SACU | Pixel pipe shift clock (CLKPIPE) | or2 | ROXY, SEGU | Depth 63.8 ge; fan-out 53 |
| ROXY | Fine-scroll gate | nor_latch | Set: PAHA; Reset: POVA | See [Startup and fine scroll](startup-fine-scroll.md) |
| SEGU | Buffered CLKPIPE | not_x4 | TYFA | Depth 49.4 ge |
| TYFA | CLKPIPE gate | and3 | POKY, SOCY, VYBO | Depth 48.5 ge |
| POKY | Pixel-pipe data ready | nor_latch | Set: PYGO; Reset: LOBY | LOBY = NOT(mode3): clears POKY outside Mode 3 |
| SOCY | Window-halt gate | not_x1 | TOMU | SOCY = NOT(RYDY) via triple inversion (TOMU, SYLO) — drops during window activation |
| VYBO | Operational per-dot gate | nor3 | MYVO, FEPO, WODU | Depth 46.9 ge |
| FEPO | Sprite X priority aggregate | or2 | FOVE, FEFY | Depth 41.9 ge |

Four halt conditions — any one stalls CLKPIPE (SACU stuck high):

| Halt | Cause | Mechanism |
|---|---|---|
| ROXY=1 | Fine scroll active | Forces SACU=1 directly |
| POKY=0 | Data not ready (outside Mode 3, or first tile not yet fetched) | TYFA=0 → SEGU=1 |
| SOCY=0 | Window activation (RYDY=1) | TYFA=0 → SEGU=1 |
| VYBO=0 | Sprite match (FEPO=1) or H-Blank (WODU=1) | TYFA=0 → SEGU=1 |

When free-running, SACU is simply MYVO's alternation pushed through the
tree: SACU is low while ALET is high and high while ALET is low — one rising
edge per dot, on ALET falling. The shift registers advance on SACU **rising**
only.

**Why CLKPIPE arrives late.**

> Crystal → ALET (16.3 ge) → MYVO (22.2) → VYBO (46.9,
> dominated by WODU's 45.5-ge input) → TYFA (48.5) → SEGU (49.4) →
> SACU (63.8)

The pixel clock is the deepest routinely-toggling signal in the PPU —
which is exactly why data-vs-clock ordering works out: shifter data settles
at depth 0–8 ge, the parallel-load enable at 16.2 ge, and the shift clock
last of all. Data is always stable long before the edge that shifts it.

## Sprite penalty interaction

A sprite X match sets FEPO=1, halting CLKPIPE for the fetch core (TEKY↑ →
TAKA↓, 6.010 dots; zero SACU transitions; SACU resumes 1.024 dots after
TAKA clears — dmg-sim measurement; the plateau total is exactly +11.000
dots, [penalty model](../sprite-pipeline/penalty-model.md)). The resume edge
is the book's one margin-sensitive edge assignment — its exact dot is
delay-calibration-dependent to within half a dot, a model prediction
rather than a hardware fact; the totals (the +11.000 plateau, the 6-dot
core) are calibration-independent. The freeze also pins
the drain-detect cascade ([the tile fetcher](fetcher.md)), so no tile
boundary fires during a sprite fetch.

```admonish warning "Pitfall: TAKA is not on the SACU halt path"
VYBO sees FEPO, not TAKA. A mid-fetch `LCDC.1 ← 0` write drops FEPO early
via AROR (a short combinational chain), and SACU resumes mid-fetch while the
fetch state machine runs on unaffected. The Mode-3-end dot count is set by
the FEPO↓ edge, not TAKA↓ ([sprite pipeline](../sprite-pipeline/lcdc-paths.md)).
```

## H-Blank

When PX hits terminal with no sprite match, WODU=1 halts CLKPIPE the same
way; VOGA captures WODU on the same-dot ALET rising edge and XYMU sets
0.436 dots after WODU↑ ([mode transitions](../mode-transitions.md)). When
XYMU flips, the PAHA/LOBY/ROXY/POKY chain switches to its non-Mode-3 state
combinationally — POKY disarms, ROXY re-arms for the next scanline, the
Mode-3-only DFFs acquire their reset hold, and VRAM/OAM access control
releases. No clock edge is waited for.
