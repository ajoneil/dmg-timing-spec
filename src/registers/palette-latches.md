# Palette Latches

BGP, OBP0, and OBP1 commit through the same CUPA strobe as every PPU
register ([writes](writes.md)) — but a palette write landing mid-Mode-3
interacts with the pixel clock, and repeated writes additionally exhibit
a below-netlist overlay at the glass (measured on BGP). This page holds
the latch cells, the measured write race, and the OR-overlap rule.

```admonish abstract "At a glance"
- A mid-Mode-3 palette write reaches the latches **after** the write-dot
  SACU↓ (the data bus settles ~17 ns into the CUPA pulse): the pixel
  shifted on the write dot uses the **old** palette value.
- Second-or-later mid-Mode-3 BGP writes fire a one-column
  **`new | old` OR-overlap** at the glass — a below-netlist behaviour with
  a BESU-reset recovery state and a visible-emission clause.
- The OR overlap is **measured on BGP**; no available OBP transition can
  discriminate, so its scope across the other palettes is open.
```

## The latch cells

24 registered cells (8 per palette: BGP, OBP0, OBP1) hold the palette
mappings. Each is a `dlatch_ee` (level-sensitive latch) that captures CPU
data-bus writes via the shared CUPA strobe
([register writes](writes.md)); the write enable is a
per-register AND of address decode and CUPA — for BGP, VELY = AND2(WERA,
CUPA). During normal rendering with no active CPU write, palette values are
stable and contribute no per-dot races.

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| BGP (8 × dlatch_ee) | BG palette | dlatch_ee | Enable: VELY = AND2(WERA, CUPA) | Per-bit cells not individually named here |
| OBP0 (8 × dlatch_ee) | Sprite palette 0 | dlatch_ee | Enable: per-register AND with CUPA | — |
| OBP1 (8 × dlatch_ee) | Sprite palette 1 | dlatch_ee | Enable: per-register AND with CUPA | — |
| TEPO | BGP write enable sub-signal | combinational | Inputs: VELY chain | Fires during a CUPA write to BGP |
| PERO | Palette-output multiplexer signal | combinational | Inputs: palette outputs | Feeds the LCD-output path ([LCD output](../lcd-output.md)) |

## The mid-Mode-3 palette-write race

Measured across 7,621 mid-Mode-3 BGP writes (dmg-sim measurement,
purpose-built `bgp_steady` ROM; zero variance per write direction). Event ordering relative to the CUPA rising edge (the
dot's ALET rising edge plus 4,848 ps of phase-generator propagation):

| Δ from CUPA↑ (ps) | Event |
|---|---|
| +0 | CUPA↑ (bus write strobe begins) |
| +1,306 | TEPO↓ (BGP write enable opens the latches) |
| +1,320 | SACU↓ (the write dot's pixel clock falls) |
| +16,870 / +17,060 | BGP dlatch output changes (per write direction) — the CPU data bus settles here, past the pixel edge |
| +18,510 | PERO changes |
| +19,464 | REMY (LD0) changes (RAVO/LD1 likewise when the transition flips it) |
| +129,254 | Next SACU↑ |

The dlatch output lands **~15.6 ns after the write-dot SACU↓** — not a
marginal race: the latch opens early (TEPO↓), but the captured value
follows the CPU data bus, which settles in the second half of the CUPA
window. **The pixel being shifted out on the write dot therefore uses the
OLD palette value**, structurally; the new value first affects the pixel
shifted on the next SACU rising edge, +129,254 ps after CUPA↑.

The LCD-side pixel-emission edge that captures each pixel — `cp_pad`↑,
which lags SACU↑ by 1,019 ps combinationally
([LCD output](../lcd-output.md)) — frames the same fact from the glass side:
the OLD-palette pixel emits at the `cp_pad`↑ 0.466 dots *before* the write
CUPA↑, and the NEW-palette pixel emits at the `cp_pad`↑ 0.534 dots *after*
it, within the same ALET cycle as the write. No further latching exists
between PERO/RAVO/REMY and the pad.

Outside Mode 3 (LCD off, H-Blank, VBlank, Mode 2), SACU is static and palette
changes propagate combinationally with no pixel-clock interaction — the write
is immediately visible on the next pixel when rendering resumes.

## The BGP OR-overlap rule

The single-write race above is the netlist-level story, and the gate-level
simulation reproduces it exactly for isolated mid-Mode-3 writes. At the LCD
interface, however, BGP writes additionally exhibit an **OR-overlap**
behaviour that is *below the netlist's modelling scope* — it was
characterised against hardware reference photographs, not by gate-level
measurement. The behavioural rule, with its empirically-tightened scope:

> When a BGP CUPA↑ satisfies BOTH (a) it is **not the first BGP CUPA since the
> most recent BESU↑** AND (b) **at least one visible `cp_pad`↑ has elapsed
> since that prior BGP CUPA**, the first `cp_pad`↑ after the new CUPA (the
> same-dot late-half emission edge at +0.534 dots after CUPA↑) samples a
> palette value equivalent to **`BGP_new | BGP_old`** rather than `BGP_new`
> alone. All subsequent `cp_pad`↑ events sample `BGP_new` directly. When
> either clause fails — the CUPA is the first BGP write since BESU↑, or no
> visible `cp_pad`↑ has occurred since the prior write — the first `cp_pad`↑
> after the new CUPA samples `BGP_new` cleanly.
>
> A **visible `cp_pad`↑** fires during ST_pad=0 — it shifts a real LCD
> column. The POVA-driven pulse at AVAP+6.5 dots and the fine-scroll
> startup pulses fire under ST_pad=1 ([LCD output](../lcd-output.md));
> neither engages the rule.
>
> The prior-write tracking resets at each **BESU↑** (Mode 2 entry,
> [OAM scan](../oam-scan.md)) and persists through H-Blank and VBlank —
> a late-frame write stays primed until the next frame's first Mode 2.

When `NEW ⊇ OLD` bitwise, `OR = NEW` and the rule is invisible. When the
write clears bits, the first post-write column emits with OLD's cleared bits
re-added — a one-column transient before the new palette takes effect.

### What the rule predicts

The rule reproduces every hardware reference image across three suites —
DMG photographs from the Mealybug Tearoom suite, Gambatte's
`dmgpalette_during_m3` family, and AGE's `m3-bg-bgp` (the last
hardware-verified on a DMG-CPU C):

```admonish info title="Per-test validation" collapsible=true
| Test | Write structure | Outcome under the rule |
|------|-----------------|------------------------|
| Mealybug `m3_bgp_change`, LY ≥ 1 | W1 in Mode 2; W2…W7 in Mode 3 | W1 clean; W2…W7 fire OR (both clauses hold). Matches the reference's 13-column transients: W3/W5/W7 clear bits — visible; W2/W4/W6 OR ≡ NEW — invisible |
| Gambatte `dmgpalette_during_m3_{3,4,5}` | Setup writes pre-LY=0 (no BESU↑ between), then two Mode-3 writes per scanline | LY=0's BESU↑ resets the state: each scanline's first write is clean (clause (a) fails), the second fires OR — discriminating both the every-CUPA and the no-BESU-reset scopes |
| AGE `m3-bg-bgp`, LY=9 (SCX=1) | W1 pre-ST_pad↓ (AVAP+3.50); W2 at AVAP+15.50 — 707 ps (0.003 dots) after ST_pad↓, the column-0 emission dot | W1 clean. W2 satisfies (a) but **fails (b)**: zero visible `cp_pad`↑ between W1 and W2 (the whole window lies in ST_pad=1). W2 samples NEW; reference column 0 = shade 0 |
| AGE `m3-bg-bgp`, LY=21 (SCX=5) | Same, shifted +4 dots (one extra NOP; SCX=5 adds 4 dots of fine-scroll gating) | Same: W2 fails (b); column 0 = shade 0 |
| AGE `m3-bg-bgp`, LY ∈ {8, 10–20, 22–79} | W1 pre-column-0; W2 after the first visible `cp_pad`↑ | W1 clean; W2 fires OR post-column-0. Reference: column 0 = shade 3 (bare W1), then the one-column transient |
| Mealybug `m3_obp0_change` | Single OBP0 write, OR ≡ NEW ($04 → $0C) | Cannot discriminate — OR ≡ NEW either way; the OBP scope question stays open (below) |
```

### Evidence

```admonish info title="Primary evidence" collapsible=true
- **dmg-sim measurement** pins the netlist-level invariants: 12.000-dot
  CUPA-to-CUPA spacing (Mealybug `m3_bgp_change`), write-invariant
  CUPA→dlatch latency (16,870/17,060 ps — the bus-settle instant, not the
  enable), uniform +0.534-dot CUPA→`cp_pad`↑ latency, and no per-dot race in
  the BGP-to-`cp_pad` chain (all 24 palette dlatch races are per-frame in the
  static analysis). Isolated-write tests match at LCD-column resolution. The
  OR overlap is **structurally absent from the netlist** at every timing
  calibration — which is what places the mechanism beyond the die model.
- **The reset is BESU↑** (Mode 2 entry), not Mode-3 entry (AVAP) or Mode-3
  exit (WODU): Mealybug `m3_bgp_change` LY ≥ 1 fires the OR overlap on a
  W1→W2 pair that spans Mode 2 → Mode 3 (an AVAP reset would miss it), and
  the Gambatte `_3/_4/_5` setup writes persist through VBlank into LY=0's
  first Mode-3 write (a WODU reset would clear them). BESU↑ satisfies both.
- **The overlap also requires a visible `cp_pad`↑ between the two writes**
  (AGE `m3-bg-bgp`): the per-scanline SCX=LY−X handler shifts W2 by one dot
  per SCX step, and at LY=9 and LY=21 — where W2 lands on the column-0
  emission dot with no visible `cp_pad`↑ between W1 and W2 — the hardware
  reference shows clean NEW (so the condition is the intervening `cp_pad`↑,
  not merely "second-or-later since BESU↑"). Edge timing: W2 CUPA↑ at
  AVAP+15.4962 dots, ST_pad↓ at +15.4933, first visible `cp_pad`↑ at
  +16.0444, `ld_pad[1:0]` = 0b00 (shade 0) at the column-0 sample edge.
```

### Mechanism

```admonish question "Open question: the OR-overlap mechanism"
The BESU↑ reset — not every CUPA, not Mode 3 exit — extends the recovery
duration to roughly a scanline including H-Blank. That is too long for
intra-cell driver-vs-keeper contention at the BGP dlatch, which would settle
within nanoseconds. The state is therefore best placed *downstream of the
die*, at the LCD glass's column shift register: while the pipe actively
shifts a `cp_pad`↑ each dot, the downstream state stays primed for
OR-overlap; H-Blank silences the shift activity; the next BESU↑ marks
resumption with fresh state.

The AGE evidence sharpens this: the primed state engages on **actual
sampling of a value into a visible LCD column**, not on the CUPA write
itself — `cp_pad`↑ events during ST_pad=1 do not drive the glass and do not
engage it. The LD0/LD1 pad drivers are ruled out as the home of the state:
the pad path is combinational and measured stateless. Discriminating the
mechanism fully needs hardware capture of LD0/LD1 and CP across BGP writes
spanning H-Blank; the behavioural rule above is sufficient for emulation
regardless.
```

### Per-palette scope

```admonish note "For implementors"
The OR overlap is **measured on BGP**. Whether it extends to OBP0/OBP1 is
untested: the only available mid-Mode-3 OBP transition has OR ≡ NEW, which
cannot discriminate. Apply the rule to BGP, and treat OBP0/OBP1 as
following the bare single-write race until a discriminating test exists —
noting that the working mechanism hypothesis (glass-side state, above)
contains nothing BGP-specific.
```
