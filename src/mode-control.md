# Rendering Mode Control

XYMU is the PPU's single mode-state handle — a NOR latch whose polarity (Q=0
during Mode 3, Q=1 elsewhere) drives 23 downstream consumers across fetch
control, VRAM/OAM access, STAT mode bits, sprite comparators, and fine-scroll
gating. This chapter describes XYMU's structure and the mode lifecycle;
[Mode transitions](mode-transitions.md) describes what drives XYMU's
transitions, edge by edge.

```admonish abstract "At a glance"
- XYMU is the PPU's single mode-state handle: a NOR latch with **Q=0
  during Mode 3** ("not rendering" polarity) and 23 consumers.
- Mode 3 **starts** when AVAP resets XYMU directly; it **ends** through
  WODU → VOGA (same-dot ALET↑ capture) → WEGO → XYMU — 0.436 dots total.
- The mode-control web splits into **async latches** (XYMU, BESU, ROXY,
  POKY, LONY — combinational delay only) and **clocked DFFs** (VOGA, RUTU,
  POPU, MYTA) — most one-dot timing subtleties come from that split.
```

## Mode lifecycle

The PPU mode is determined by combinational logic from several registered
signals:

| Mode | Condition | Duration |
|------|-----------|----------|
| Mode 2 (OAM Scan) | XYMU=1, CENO active, OAM scan in progress | 80 dots (40 OAM entries × 2 dots) |
| Mode 3 (Pixel Transfer) | XYMU=0 (rendering active) | 173 + (SCX & 7) + sprite penalty + window penalty dots |
| Mode 0 (H-Blank) | XYMU=1, not VBlank | remainder of the 456-dot line |
| Mode 1 (VBlank) | POPU set (LY ≥ 144) | 10 lines × 456 dots |

This table is the behavioural surface that register-level documentation
already describes; everything below is the machinery underneath it.

## The XYMU latch

XYMU is the PPU's "not rendering" signal. The transition out of Mode 3
propagates through a WODU → VOGA → WEGO → XYMU chain
([Mode transitions](mode-transitions.md) carries the full chain timing table;
this section introduces the signals).

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| XYMU | Rendering-mode latch (active-low Mode 3 indicator) | nor_latch | Set: WEGO; Reset: AVAP | Primary here; 23 downstream consumers |
| AVAP | Mode 2→3 trigger | not_x2 | Input: BEBU | See [OAM scan](oam-scan.md) for the detection chain |
| WODU | Mode 0 condition | and2 | Inputs: XENA, XANO | See [STAT interrupts](stat-interrupts.md) for the full breakdown |
| VOGA | H-Blank capture DFF | dffr | ALET | Data: WODU; reset: TADY |
| WEGO | XYMU set driver | or2 | Inputs: TOFU, VOGA | Drives XYMU's set input |
| TOFU | Video reset | not | Input: XAPO | Reset from LCDC disable / system reset |
| TADY | VOGA + PX-counter reset | nor2 | Inputs: TOFU, ATEJ | One gate drives both the VOGA H-Blank-capture reset and the PX counter reset; ATEJ shared with the scan counter reset chain ([Mode transitions](mode-transitions.md)) |

**Transitions:**

- **XYMU↓ (Mode 3 starts) at AVAP↑.** AVAP drives XYMU's reset input.
- **XYMU↑ (Mode 3 ends) at WEGO↑.** WEGO drives XYMU's set input.

> Signal path (Mode 3 end): **WODU → VOGA → WEGO → XYMU**

The WODU condition propagates through VOGA (an ALET-clocked DFF capturing
WODU), WEGO (an OR2 combining VOGA with the TOFU video reset), and onto XYMU's
set pin — 0.436 dots from WODU↑ to XYMU↑ (dmg-sim measurement): VOGA
captures at 0.435, and the WEGO → XYMU gate adds the final 0.001.

Two chain members carry timing weight:

- **WODU** = AND2(XENA, XANO) — no sprite match active AND the pixel counter
  at terminal count: the H-Blank condition.
- **VOGA** captures WODU on the **same-dot** ALET rising edge: WODU rises
  during the ALET-low phase, and the rising edge within that same dot latches
  it (0.435 dots after WODU↑). Mode 3 therefore ends within the dot WODU
  fires.

WEGO's TOFU arm means an LCD disable also sets XYMU; TADY's ATEJ arm resets
VOGA at every scanline boundary.

### XYMU's 23 consumers

XYMU is an asynchronous NOR latch — no clock; its consumers see transitions
after combinational delay only. The complete consumer set (netlist out-edges):

- **Fetch cycle resets** — LYZU, PYGO, RENE, PAHO, PUXA, RYFA (XYMU as reset
  input).
- **Fine-scroll match reset** — NYZE (XYMU as reset input).
- **Rendering-gated paths** — LOBY, PAHA (both =1 outside Mode 3; see
  [BG pipeline](bg-pipeline/startup-fine-scroll.md) for the PAHA polarity convention).
- **VRAM access control** — ROPY, XANE.
- **OAM access control** — AJON, ASAM, AZEM, BUZA.
- **STAT mode bits** — SADU, XATY.
- **Sprite comparator resets** — SEBA, TOBU, VONU.
- **Fetch control** — LURY, SUVU.
- **Sprite Y-compare enable** — TEPA.

## Signal types in the mode-control web

The mode-control web mixes asynchronous latches with synchronous DFFs, and the
distinction carries timing weight:

| Signal | Type | Async? | Notes |
|--------|------|--------|-------|
| XYMU | NOR latch | Yes | Reset by AVAP (Mode 3 starts, XYMU↓); set by WEGO (Mode 3 ends, XYMU↑). |
| BESU (scan-active) | NOR latch | Yes | Set by `start_oam_parsing` at Mode 2 start; reset by ASEN (= OR2(ATAR, AVAP)) at Mode 2 end or video reset. Q=1 during Mode 2. |
| ROXY (fine scroll gate) | NOR latch | Yes | Set (suppress CLKPIPE) by PAHA; reset (enable CLKPIPE) by POVA |
| POKY (pixel pipe data ready) | NOR latch | Yes | Set (data ready) by PYGO; reset by LOBY |
| LONY (tile fetching) | NAND latch | Yes | Set by LURY, reset by NYXU |
| VOGA (H-Blank) | DFF (clk=ALET) | No | Synchronous capture of WODU on the **same-dot** ALET rising edge |
| RUTU (LINE_END) | DFF (clk=SONO) | No | Synchronous capture of SANU |
| POPU (VBlank) | DFF (clk=NYPE) | No | Synchronous capture of XYVO |
| MYTA (FRAME_END) | DFF (clk=NYPE falling) | No | Synchronous capture of NOKO |

The latches respond to their inputs with combinational delay only — they do
not wait for a clock edge. The DFFs capture on specific edges, introducing
pipeline delay. Most one-dot timing subtleties in the mode-transition chapters
come down to exactly this split.
