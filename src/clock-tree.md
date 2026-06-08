# The Clock Tree

```admonish abstract "In this part"
The three Foundations chapters carry everything else: this chapter names
the clocks and their phase relationships; [Line counters](line-counters.md)
builds the LX/LY counters and the LINE_END pulse on them; and
[Mode control](mode-control.md) derives the four PPU modes from a handful
of latches.
```

Every timing claim in this book ultimately reduces to "this cell captures on that
edge of this clock". This chapter names the clocks. It follows the master clock
from the crystal input through the buffer chain to the PPU's per-dot clock pair
(ALET/MYVO), down the video divider chain to the per-M-cycle pair (TALU/SONO),
and establishes the phase relationships the rest of the book leans on.

```admonish abstract "At a glance"
- One **dot** = one full `ck1_ck2` cycle. ALET tracks the crystal **in
  phase** (16.3 ge deep); MYVO = NOT(ALET). "ALET rising / falling" is the
  book's per-dot edge vocabulary.
- **XUPY-clocked DFFs capture on ALET falling** — the opposite edge from
  ALET-clocked DFFs. BYBA→DOBA measures 0.487 dots apart.
- **TALU/SONO** are the complementary 1 MHz line-level pair; a boundary
  TALU↑ leads the same M-cycle's CLK9↑ by ~0.496 dots.
- All PPU dividers are **held at 0 under VID_RST** and count from zero on
  release.
```

## The master oscillator

The DMG-CPU B has a single crystal oscillator at 4.194304 MHz connected through
the top-level input `ck1_ck2` — the **master clock** from which all on-chip
timing derives. Wherever this book says "the master clock" (including the
propagation-delay origin used in [Races](races.md)), the referent is `ck1_ck2`.

One **dot** covers both edges of the master clock. ALET, the PPU's main 4 MHz
clock, tracks the master clock **in phase**: ALET rising corresponds to
`ck1_ck2` rising after ~16.3 ge of buffer delay. MYVO = NOT(ALET) is the
complementary clock — DFFs clocked by ALET capture on one edge, those clocked
by MYVO on the other, both within a single dot. "One ALET half-cycle" and "one
ATAL half-cycle" both mean half a dot.

Dot counts in this book are edge counts — independent of the simulation's
delay model. Picosecond values come from the gate-level simulation's
244,000 ps dot period —
see [How to read the numbers](methodology.md#how-to-read-the-numbers).

## Crystal to PPU clocks

The crystal feeds a NAND feedback pair (ANOS/AVET) that produces ATAL (the
4 MHz enable) and its complement ADEH. From ATAL the PPU clock tree branches
through a buffer chain (AZOF → ZAXY → ZEME) to ALET (4 MHz PPU main clock) and
its complements (MYVO, MOXE). A parallel branch (XYVA → XOTA) feeds the
WUVU/VENA/WOSU video divider chain; VENA drives TALU (1 MHz LX-counter clock)
and its complement SONO. The tree diagram shows the propagation hierarchy; the
reference block catalogues each named cell.

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| `ck1_ck2` | Master clock (crystal oscillator input, 4.194304 MHz) | top-level input | — (external crystal) | Tree root. One dot = one full `ck1_ck2` cycle (one rising + one falling edge). Conventional propagation-delay origin (see [Races](races.md)). Not an on-chip cell — no netlist cell type. |
| ANOS | Crystal feedback NAND | nand2 | Input: `ck1_ck2` | Pairs with AVET |
| AVET | Crystal feedback NAND | nand2 | Input: `ck1_ck2` | Pairs with ANOS |
| ATAL | 4 MHz buffered enable | not_x2 | Input: ANOS/AVET output | Master PPU enable. Also drives the CPU phase-generator ring AFUR/ALEF/APUK/ADYK paired with ADEH ([Register writes](registers/writes.md)). ATAL's edges define the "ATAL half-cycle" timing unit. |
| ADEH | ATAL complement | not_x1 | Input: ATAL | Pairs with ATAL as the complementary 4 MHz enables driving the CPU phase-generator ring. |
| AZOF | PPU clock buffer | not_x6 | Input: ATAL | Fan-out buffer |
| ZAXY | Buffer stage | not_x2 | Input: AZOF | — |
| ZEME | Buffer stage | not_x4 | Input: ZAXY | Fan-out point: drives ALET (main PPU clock) and XYVA (parallel divider branch). |
| ALET | 4 MHz PPU main clock | not_x2 | Input: ZEME | Depth 16.3 ge from crystal. Rising and falling edges are the book's canonical per-dot timing-event vocabulary ("ALET rising" / "ALET falling"); see the pipeline pairs below and [Races](races.md). |
| MYVO | ALET complement | not_x1 | Input: ALET | Depth 22.2 ge (5.9 ge more than ALET) |
| MOXE | ALET complement (fine-scroll path) | not_x1 | Input: ALET | Same phase as MYVO, different buffer. Depth 16.7 ge. Dedicated single-consumer inverter isolates the fine-scroll path's clock loading (drives NYZE only) from MYVO's high-fan-out path. |
| XYVA | Parallel divider branch | not_x1 | Input: ZEME | Single-stage inverter feeding XOTA. Same phase as ALET (both compute `NOT(ZEME)`) on a physically separate routing path; no downstream consumers beyond XOTA. |
| XOTA | WUVU clock | not_x1 | Input: XYVA | Drives the WUVU DFF. "XOTA rising = ALET falling" is the canonical derived-edge framing used throughout the book's WUVU/XUPY divider timing descriptions. |
| WUVU | 2 MHz toggle divider | dffr | XOTA | ~Q → D feedback (toggle). Halves XOTA frequency; drives VENA (via ~Q) and feeds the XUPY inverter, WOSU, and the LCD-control-phase NORs. |
| VENA | 1 MHz toggle divider | dffr | WUVU.~Q | ~Q → D feedback. Halves WUVU.~Q frequency, producing the 1 MHz cadence that TALU inverts and drives into the LX counter / LINE_END pipeline. |
| WOSU | WUVU sampler | dffr | XYFY | Captures WUVU on XYFY rising, producing a half-dot-phase-shifted WUVU replica; feeds WOJO ([LCD output](lcd-output.md)) as the phase-differentiated input of the NOR2 LCD-control-phase decode. |
| TALU | 1 MHz LX counter clock | not_x4 | Input: VENA | Drives the [LX counter](line-counters.md) (SAXO chain), NYPE (LINE_END redistribution), ROPO (LY==LYC latch), and related scanline-level DFFs. |
| SONO | 1 MHz LINE_END capture clock (TALU complement) | not_x1 | Input: TALU | Clocks RUTU ([line counters](line-counters.md)) — captures SANU (LINE_END decode) on the SONO edge, half an M-cycle after the LX increment. |
| XYFY | XOTA complement | not_x1 | Input: XOTA | Clocks WOSU. Primary XOTA complement (WOSU is its only consumer; no fan-out alternative). |

```
ck1_ck2 (4.194 MHz crystal)
└── ANOS (nand2) / AVET (nand2) — feedback pair
    └── ATAL (not_x2) — 4 MHz buffered enable
        ├── ADEH = NOT(ATAL) — complement
        ├── [CPU phase generators: AFUR/ALEF/APUK/ADYK ring]
        └── AZOF (not_x6) = NOT(ATAL) — PPU clock buffer
            └── ZAXY (not_x2) = NOT(AZOF)
                └── ZEME (not_x4) = NOT(ZAXY)
                    ├── ALET (not_x2) = NOT(ZEME) — 4 MHz PPU main clock
                    │   ├── MYVO (not_x1) = NOT(ALET) — complementary to ALET
                    │   └── MOXE (not_x1) = NOT(ALET) — same phase as MYVO
                    └── XYVA (not_x1) = NOT(ZEME)
                        └── XOTA (not_x1) = NOT(XYVA)
                            ├── WUVU (dffr) — toggle, 2 MHz [clk=XOTA]
                            │   ├── VENA (dffr) — toggle, 1 MHz [clk=WUVU.~Q]
                            │   │   ├── TALU (not_x4) = NOT(vena_n) = VENA.q — LX counter clock, 1 MHz
                            │   │   │   └── SONO (not_x1) = NOT(TALU) = vena_n
                            │   │   └── [used as clock for per-dot DFFs]
                            │   └── WOSU (dffr) — samples WUVU on opposite edge [clk=XYFY]
                            └── XYFY (not_x1) = NOT(XOTA) — opposite edge
```

### Phase relationships

These five facts carry most of the book's edge reasoning:

- **ALET rising = `ck1_ck2` rising** (in phase, not inverted). The XTAL pad
  inverts once (`clk_in_n` = NOT(`ck1_ck2`)); the ANOS/AVET feedback pair
  restores ATAL to crystal phase;
  four more inversions through AZOF → ZAXY → ZEME → ALET land in phase at
  16.3 ge total delay. ADEH = NOT(ATAL) pairs with ATAL in the CPU
  phase-generator ring.
- **ALET and MYVO = NOT(ALET)** are complementary clocks at the 4 MHz master
  rate. ALET is 16.3 ge deep from the crystal; MYVO is 22.2 ge deep (5.9 ge
  additional from the inverter and wire routing). DFFs clocked by ALET capture
  on one edge; DFFs clocked by MYVO capture on the opposite edge. This pair
  drives most per-dot PPU timing.
- **TALU and SONO** are complementary clocks at 1 MHz (derived from VENA),
  driving line-level timing.
- **WUVU and VENA are toggle DFFs** (internal ~Q → D feedback) forming the
  video clock divider chain. WOSU is a sampler DFF (D input = WUVU) that
  captures WUVU on XYFY rising to produce a phase-differentiated replica for
  the LCD-control decode.
- **All PPU dividers are held at 0 while VID_RST is asserted** (LCD off) and
  count from zero on deassertion — see
  [Register writes](registers/writes.md) for the deassertion path.

## PPU clock domains

Every registered cell in the PPU is clocked by a signal traceable to the
crystal. The major clock domains:

| Clock signal | Source | Frequency | What it drives |
|-------------|--------|-----------|----------------|
| `sacu` (CLKPIPE) | combinational, via `alet` chain | per-dot, variable arrival | Pixel shift registers, X match, PX counter — 52 cells |
| `alet` | crystal via buffer chain | 4 MHz | BG fetch cycle DFFs (NYKA, LYZU, PYGO, RENE), sprite fetch control (DOBA), window mode (NOPA), H-Blank capture (VOGA) |
| `myvo` = NOT(alet) | crystal via buffer chain | 4 MHz (opposite edge) | PORY (fetch cycle counter), VRAM timing |
| `lebo` = NAND2(moce, alet) | gated alet | conditional per-dot | LAXU (fetch counter bit 0) — stops when MOCE goes low |
| `laxu` → `mesu` → `nyva` | ripple from LAXU | per-dot, delayed | Fetch counter bits 1, 2 (tile fetch cycle counting) |
| `talu` = NOT(vena_n) (= VENA.q) | 1 MHz from VENA | 1 MHz | LX counter (SAXO chain), NYPE, ROPO |
| `sono` = NOT(talu) (= vena_n) | complementary to TALU | 1 MHz | RUTU (LINE_END), SYGU |
| `xota` | buffered 4 MHz | 4 MHz | WUVU (video clock divider) |
| `xyfy` = NOT(xota) | opposite to XOTA | 4 MHz (opposite edge) | WOSU |
| `wuvu` | toggle at 2 MHz | 2 MHz | VENA (via ~Q) |
| `xupy` = NOT(wuvu_n) (= WUVU.q) | 2 MHz | 2 MHz | BYBA (OAM scan complete capture), CENO (OAM phase control) |
| `nype` | from RUTU (LINE_END) | per-line | POPU (VBlank), MEDA (LY==0), MYTA (FRAME_END) |
| `popu` | VBlank flag | per-frame | NAPO |
| `muwy` (LY bit 0) → ripple | per-line from RUTU | per-line | LY counter chain, sprite store clocks — 181 registers via gated paths |
| `sabe` = NAND2(lape, tame) | gated NOT(alet) | conditional per-dot | TOXE (sprite fetch counter bit 0) — advances when ALET rises, stops at count 5 |
| `toxe` → `tuly` → `tese` | ripple from TOXE | per-dot, delayed | Sprite fetch counter bits 1, 2 |
| `seba` / sprite fetch | gated per-line | per-line/per-sprite | Sprite Y comparator, store writes, priority |
| `roxo` = NOT(segu) | from CLKPIPE buffer | per-dot | PUXA (fine scroll match capture) |
| `moxe` = NOT(alet) | same phase as MYVO | 4 MHz (opposite edge) | NYZE (fine scroll match second stage) |
| `pecu` = NAND2(roze, roxo) | gated CLKPIPE inverse | conditional | RYKU (fine scroll counter bit 0) |

## Complementary pairs and pipeline relationships

A DFF clocked by A feeding a DFF clocked by NOT(A) forms a **half-period
pipeline**: the downstream cell captures, half a period later, what the
upstream cell just captured. Three pairs recur:

| Pair | Upstream captures on | Downstream captures on | Pipeline |
|------|----------------------|------------------------|----------|
| ALET / MYVO | ALET↑ — NYKA | MYVO↑ (= ALET↓) — PORY | half a dot; the book's prototypical example ([Races](races.md)) |
| TALU / SONO | TALU↑ — the LX counter (SAXO) | SONO↑ — RUTU captures SANU | half an M-cycle, LX → LINE_END |
| XOTA / XYFY | XOTA↑ — WUVU | XYFY↑ — WOSU samples WUVU | half a dot; the divider's phase-shifted replica |

Worked through for ALET/MYVO: on ALET rising, the ALET-clocked DFFs (NYKA,
LYZU, PYGO, RENE, DOBA, NOPA) capture. On ALET falling — MYVO rising —
PORY captures NYKA's *new* output. PORY therefore always holds the value
NYKA captured half a dot earlier.

**XUPY-clocked DFFs capture on ALET falling** — the opposite edge from the
ALET-clocked group. The chain: XOTA is ALET's complement; XOTA clocks
WUVU; XUPY = WUVU.q. So BYBA — the scan-done capture
([OAM scan](oam-scan.md)) — moves half a dot *before* DOBA, its
ALET-clocked partner in the AVAP edge detector.

```admonish info "Measured: the BYBA/DOBA phase split"
BYBA's Q moves 2,566 ps after an ALET falling edge; DOBA's Q moves 884 ps
after an ALET rising edge; their separation is 118,888 ps = 0.487 dots,
constant across 1,721 scanlines (dmg-sim measurement, gbmicrotest
`int_hblank_nops_scx0`).
```

### TALU / `ck1_ck2` half-edge alignment

TALU rises align with `ck1_ck2` **falling** edges (modulo divider-startup
phase): WUVU toggles on XOTA↑ = ALET↓ = `ck1_ck2`↓, VENA captures the toggle
on the same edge family, and TALU inverts VENA — so each TALU↑ traces to a
`ck1_ck2`↓ through the divider chain's buffer delays. Measured directly:
**TALU↑ lands 9,350 ps after its `ck1_ck2`↓ edge**, constant across 51,489
edges (dmg-sim measurement).

### Steady-state TALU↑ vs CLK9↑ within one M-cycle

The TALU↑ that opens an M-cycle (clocking NYPE, SAXO, and ROPO) leads the
CLK9↑ that opens the *same* M-cycle ([Interrupt
dispatch](interrupt-dispatch.md)) by **121,110 ps ≈ 0.496 dots** (dmg-sim
measurement, constant across 51,489 M-cycles). The half-dot offset is
structural: the TALU edge comes from the `ck1_ck2`↓ preceding the boundary
(+9,350 ps), the CLK9 edge from T1's `ck1_ck2`↑ (+8,440 ps):
½ dot − 9,350 + 8,440 = 121,090 ps, matching the measured value to within
the edge-asymmetry residual.

The practical reading: "TALU↑ at the M-cycle boundary" fires roughly half a
dot **before** that M-cycle's CLK9↑. The two edges open the same M-cycle but
are not co-located.
