# Line Counters

LX and LY are the PPU's scanline-position and scanline-number counters. Both run
on every scanline regardless of mode and are consumed everywhere: Mode 2's
sprite Y compare uses LY, Mode 3's window WY/WX compares use LY and the pixel
pipeline, and the mode-transition machinery keys off both (LX=113 triggers
LINE_END; LY≥144 enters VBlank). LINE_END's distribution via NYPE splits across
two TALU edges, driving POPU (Mode 1 capture), MYTA (FRAME_END), and MEDA (LCD
vertical sync).

```admonish abstract "At a glance"
- LX counts 0–113 on TALU (114 M-cycles = 456 dots per scanline); the
  LX=113 decode (SANU) is captured into **RUTU** on the complementary SONO
  edge.
- The RUTU pulse is **one full TALU cycle (4 dots)**; the TALU edge inside
  it does not advance LX — 114 TALU edges per line, 113 increments.
- LY clocks on RUTU, with three decodes: XYVO (VBlank, ≥144), NOKO
  (FRAME_END, =153), NERU (=0 — LCD vertical sync only).
- NYPE distributes LINE_END across **two complementary TALU edges**: POPU
  first; MYTA and MEDA one TALU period later.
```

## The LX counter

LX is a 7-bit ripple counter clocked by TALU (1 MHz, = VENA.q), counting 0–113:
114 M-cycles × 4 dots = 456 dots per scanline. Each stage's clock comes from the
previous stage's ~Q output (the same convention as the
[BG fetch counter](bg-pipeline/fetcher.md)). LINE_END is detected combinationally (SANU)
and captured into RUTU on the complementary SONO clock edge.

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| SAXO | LX bit 0 | dffr | TALU | Reset by MUDE |
| TYPO | LX bit 1 | dffr | SAXO (~Q ripple) | Reset by MUDE |
| VYZO | LX bit 2 | dffr | TYPO (~Q ripple) | Reset by MUDE |
| TELU | LX bit 3 | dffr | VYZO (~Q ripple) | Reset by MUDE |
| SUDE | LX bit 4 | dffr | TELU (~Q ripple) | Reset by MUDE |
| TAHA | LX bit 5 | dffr | SUDE (~Q ripple) | Reset by MUDE |
| TYRY | LX bit 6 | dffr | TAHA (~Q ripple) | Reset by MUDE |
| MUDE | LX reset | nor2 | Inputs: LYHA, RUTU | LINE_END or video reset |
| SANU | LINE_END decode | and4 | Inputs: SAXO, SUDE, TAHA, TYRY | Fires at LX=113 (0b1110001) |
| RUTU | LINE_END capture | dffr | SONO | Data: SANU; half an M-cycle later than SANU settle |

```
TALU (1 MHz)
├── SAXO (dffr) [clk=TALU] — LX bit 0
│   └── TYPO (dffr) [clk=SAXO.~Q] — LX bit 1 (ripple)
│       └── VYZO (dffr) [clk=TYPO.~Q] — LX bit 2
│           └── TELU (dffr) [clk=VYZO.~Q] — LX bit 3
│               └── SUDE (dffr) [clk=TELU.~Q] — LX bit 4
│                   └── TAHA (dffr) [clk=SUDE.~Q] — LX bit 5
│                       └── TYRY (dffr) [clk=TAHA.~Q] — LX bit 6
```

Reset: MUDE = NOR2(LYHA, RUTU) — LINE_END (RUTU=1) or video reset (LYHA) drives
MUDE low, resetting the counter. LX counts from 0 to 113, where LINE_END fires.

### LINE_END detection

> **LX[0..6] → SANU → RUTU**

SANU = AND4(SAXO, SUDE, TAHA, TYRY) = AND(LX[0], LX[4], LX[5], LX[6]) decodes
LX = 113 (binary 1110001). The decode is partial — it ignores bits 1, 2, 3 —
but exact within the counter's range: 113 is the *smallest* value with bits
0, 4, 5, 6 all set, so no earlier count can fire it.

RUTU (dffr, clk=SONO) captures SANU on the SONO edge (= TALU falling). The
complementary pair does the settling work: LX increments on TALU rising,
RUTU samples on SONO rising — half an M-cycle for SANU to settle
([complementary pairs](clock-tree.md#complementary-pairs-and-pipeline-relationships)).

### The RUTU pulse

RUTU.Q is high for **one full TALU cycle = 4 dots**: it captures SANU=1 on a
SONO rising edge, MUDE = NOR2(LYHA, RUTU) holds the LX counter at 0 (keeping
SANU=0), and the next SONO rising edge captures SANU=0, ending the pulse. The
pulse is bounded by successive TALU-falling edges.

Measured across 36 consecutive scanline boundaries: pulse width **4.000
dots** (one TALU cycle) and inter-rise interval **456 dots** (the scanline
period), both zero-variance (dmg-sim measurement).

One consequence: the TALU rising edge inside the RUTU pulse does **not**
advance LX (the counter is held at 0), so a steady-state scanline sees 114
TALU rising edges of which 113 increment LX. This matters for the post-LCD-on
the first scanline, which has no leading RUTU pulse — see
[LCD-on power-up](scanline-frame-timing/lcd-on.md).

## The LY counter

LY is an 8-bit ripple counter clocked by RUTU. Three combinational decodes
detect scanline phase transitions: XYVO (VBlank, LY ≥ 144), NOKO (FRAME_END,
LY = 153), and NERU (LY = 0). Each decode is captured by a DFF clocked by NYPE
(see [LINE_END distribution](#line_end-distribution-via-nype) below).

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| MUWY | LY bit 0 | dffr | RUTU | Reset by LAMA |
| MYRO | LY bit 1 | dffr | MUWY (~Q ripple) | Reset by LAMA |
| LEXA | LY bit 2 | dffr | MYRO (~Q ripple) | Reset by LAMA |
| LYDO | LY bit 3 | dffr | LEXA (~Q ripple) | Reset by LAMA |
| LOVU | LY bit 4 | dffr | LYDO (~Q ripple) | Reset by LAMA |
| LEMA | LY bit 5 | dffr | LOVU (~Q ripple) | Reset by LAMA |
| MATO | LY bit 6 | dffr | LEMA (~Q ripple) | Reset by LAMA |
| LAFO | LY bit 7 | dffr | MATO (~Q ripple) | Reset by LAMA |
| LAMA | LY reset | nor2 | Inputs: LYHA, MYTA | FRAME_END or video reset |
| XYVO | VBlank decode | and2 | Inputs: LOVU, LAFO | Fires at LY ≥ 144 |
| POPU | VBlank capture | dffr | `nype` (NYPE rising) | Data: XYVO |
| NOKO | FRAME_END decode | and4 | Inputs: LYDO, LOVU, LAFO, MUWY | Fires at LY = 153 |
| MYTA | FRAME_END capture | dffr | `nype_n` (NYPE falling; one TALU period after POPU) | Data: NOKO; drives LY reset via LAMA |
| NERU | LY=0 decode | nor8 | Inputs: all LY bits | Fires when LY = 0 |
| MEDA | LCD vertical-sync source | dffr | `nype_n` (NYPE falling) | Data: NERU. Drives the LCD `s_pad` via the `mure` inverter ([LCD output](lcd-output.md)); not an OAM-scan consumer. |

```
RUTU (LINE_END)
├── MUWY (dffr) [clk=RUTU] — LY bit 0
│   └── MYRO (dffr) [clk=MUWY.~Q] — LY bit 1 (ripple)
│       └── LEXA (dffr) [clk=MYRO.~Q] — LY bit 2
│           └── LYDO (dffr) [clk=LEXA.~Q] — LY bit 3
│               └── LOVU (dffr) [clk=LYDO.~Q] — LY bit 4
│                   └── LEMA (dffr) [clk=LOVU.~Q] — LY bit 5
│                       └── MATO (dffr) [clk=LEMA.~Q] — LY bit 6
│                           └── LAFO (dffr) [clk=MATO.~Q] — LY bit 7
```

Reset: LAMA = NOR2(LYHA, MYTA) — FRAME_END or video reset triggers the LY reset.

The three decodes, side by side:

| Decode | Function | Fires at | Captured by | On | Consequence |
|--------|----------|----------|-------------|----|-------------|
| XYVO | AND(LY[4], LY[7]) | LY ≥ 144 | POPU | `nype`↑ | Mode 1 (VBlank) begins |
| NOKO | AND(LY[0], LY[3], LY[4], LY[7]) | LY = 153 | MYTA | `nype_n`↑ — one TALU period after POPU | LAMA goes low; LY resets to 0 |
| NERU | NOR8(all LY bits) | LY = 0 | MEDA | `nype_n`↑ | drives only the LCD vertical sync (`s_pad`, via `mure`) |

The two AND decodes use SANU's partial-decode trick: 144 and 153 are the
smallest values with their bit sets, so the decode is exact within range.
MEDA touches no scan or mode-control state — see
[LCD output](lcd-output.md) for the pad driver and
[LCD-on power-up](scanline-frame-timing/lcd-on.md) for why the first frame
after LCD-on emits no pulse.

## LINE_END distribution via NYPE

RUTU drives NYPE (dffr, clk=TALU), and NYPE's two outputs split the
distribution across two complementary TALU edges: Q clocks POPU; Q_n clocks
MYTA and MEDA one TALU period later (the capture table above).

The stagger carries two consequences:

- **The LY 153→0 race window.** The one-TALU-period gap between POPU and
  MYTA is when ROPO captures the pre-reset LYC match at the 153→0
  boundary — [STAT interrupts](stat-interrupts.md).
- **A modelling constraint** — the pitfall below.

```admonish warning "Pitfall: collapsing NYPE to one edge"
For implementations, collapsing the NYPE distribution to a single edge-hook
shifts MYTA's timing by one TALU period relative to hardware — and with it
the LY 153→0 boundary behaviour.
```
