# Notation and Conventions

This chapter defines the units, naming rules, and recurring table formats used
throughout the book. It is reference material — skim it once, then return when
a notation needs pinning down.

## Timing units

| Unit | Definition |
|---|---|
| **dot** | One full master-clock cycle: one rising plus one falling edge of the 4.194304 MHz crystal clock. ≈238.4 ns on hardware. Synonymous with **T-cycle**. |
| **M-cycle** | One CPU machine cycle = 4 dots. |
| **ATAL half-cycle** | Half a dot — the interval between two consecutive master-clock edges. Named for ATAL, the buffered 4 MHz clock at the root of the PPU clock tree. |
| **ge** | Gate equivalent — the unit of combinational depth used by the static netlist analysis (see [races](races.md)). |
| **ps** | Simulation picoseconds — a model unit, not hardware time: one dot = 244,000 ps in dmg-sim's default timing mode. See [Sources and Methodology](methodology.md#how-to-read-the-numbers). |

There is no inherent "start" edge within a dot: the master clock is a continuous
alternation of edges, and circuits trigger on whichever edge they are wired to.
Statements like "X happens 0.436 dots after Y" locate X's edge relative to Y's
edge; the prose names both.

**Writing positions.** Dots measure *length*; cycle labels name *place*:

| To express | Convention | Example |
|---|---|---|
| A duration or offset | dots, fractional where needed | "0.436 dots after WODU↑" |
| A segment of an M-cycle | T-cycles **T1–T4** | "CUPA spans T3–T4" |
| An exact point in an M-cycle | + dots from the M-cycle's start | "+3.995 dots — the tail of T4" |
| An instruction's machine cycles | **M1, M2, …** (M1 = fetch) | "dispatch M1–M4" |
| A counter position | the literal register value, from 0 | "LY 0–153", "PX terminal count 167" |

T-cycle and machine-cycle numbering follow
[gb-ctr](https://gekkio.fi/files/gb-docs/gbctr.pdf). Pan Docs uses dots
purely as durations, with the same zero-based counter values.

## Signal names

- Gate names come from the DMG-CPU B netlist and are the book's canonical
  identifiers, UPPERCASE in prose: **WODU**, **XYMU**, **AVAP**. The names
  are the reverse-engineering effort's, not the designers' — but they are
  stable and globally unique, so every claim is greppable.
- Lowercase backtick forms refer to netlist cells or simulation wires where the
  distinction matters: `` `wodu` `` (the cell), `` `byba_n` `` (a specific output
  pin).
- A gate's Boolean function is written in netlist style:
  `WODU = AND2(XENA, XANO)`, `NYXU = NOR3(AVAP, MOSU, TEVO)`.
- A few signals are simulation wire names rather than netlist cells (e.g.
  `` `ck1_ck2` ``, the crystal input); these stay lowercase throughout.

**Roles vs names.** Each gate has a one-phrase semantic role ("scan-done flag",
"Mode 3 ending condition") introduced at first mention in a chapter; after that,
the gate name alone is used. The role phrasing is identical everywhere a
gate appears, and [Appendix A](concordance.md) holds the canonical phrasing.

**Polarity.** Many signals are active-low, and several latches hold inverted
semantics — the most important being XYMU, the rendering-mode latch, which reads
**0 during Mode 3** (its role phrase is "not rendering"). Polarity quirks are
stated explicitly wherever they bite. DFF complementary outputs are written
`q`/`q_n`; when a consumer takes the `q_n` pin, the text says so.

## Edges

- `↑` = rising edge, `↓` = falling edge: **WODU↑** is WODU rising.
- The clock tree produces named edge vocabulary used everywhere: "captures on
  ALET rising", "XUPY falling". Complementary clock pairs (ALET/MYVO, TALU/SONO,
  XOTA/XYFY) mean every half-dot boundary has two equivalent names — "MYVO
  rising" *is* "ALET falling" — and chapters use whichever matches the
  flip-flop's clock pin.

## Signal paths

Cascades are written as an arrow chain in a blockquote, followed by prose:

> **WODU → TARU → SUKO → TUVA → VOTY → LALU**
>
> The Mode 0 condition (WODU) combines with the STAT enable bit through TARU,
> joins the other STAT sources at the combining gate (SUKO), and propagates
> through two inverters to clock the STAT interrupt latch (LALU).

The chain names every stage in order, with no skipped intermediates.

## Reference blocks

Chapters that discuss individual gates open with a **reference block** — a table
of the gates the chapter covers:

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| BYBA | Scan-done flag | dffr | XUPY (rising) | Captures FETO; feeds AVAP edge detector |

- **Type** is the netlist cell type (`dffr`, `nor_latch`, `and2`, `dlatch_ee`…).
- **Clock / Trigger** gives clock-and-edge for registered cells, the input list
  for combinational gates, and set/reset inputs for latches.
- Arrays of identically-clocked cells (shift-register planes, the sprite store)
  use a family row with a **Count** column; individual cell names follow in
  prose.
- A signal whose full description lives in another chapter gets a brief row
  pointing there. One chapter is *primary* for every signal;
  [Appendix A](concordance.md) records which.

## Callout boxes

Seven kinds of callout carry the book's recurring content types:

| Callout | Carries |
|---------|---------|
| **At a glance** | A chapter's headline facts — the numbers and the one rule to remember |
| **In this part** | The first chapter of each part sketches its siblings |
| **Rule** | A behavioural rule an emulator must honour, stated in its sharpest form |
| **Measured** | A self-contained measured result worth pulling out of the prose |
| **For implementors** | Modelling advice — how to structure an implementation so the rule falls out |
| **Pitfall** | A known wrong model, naming trap, or test-harness trap |
| **Open question** | A behaviour not yet pinned down by primary-source evidence — collected in [Appendix C](open-questions.md) |

Inline `(dmg-sim measurement, …)` tags remain the provenance convention —
a **Measured** callout features a result but never replaces them.

Two markers flag the edges of current knowledge: the **Open question**
callout, and a **blank role cell** in [Appendix A](concordance.md) —
connectivity documented, semantic role not yet derived, never an invented
description.
