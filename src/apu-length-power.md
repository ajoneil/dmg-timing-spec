# Length Counters and Power Cycling

The length counters are the APU's third counter family — and the one with
**no reset path at all**. That single fact, plus the polarity of their
256 Hz gate, generates the DMG power-cycle behaviour and the NRx4-write
extra-clock quirk.

```admonish abstract "At a glance"
- The counter cells (`tffnl`) have **no reset pin** — power-off
  preserves the value and merely stops the clocking.
- The counter advances on `bufy_256hz`'s **falling** edge (frame-sequencer
  even steps).
- Enabling length via NRx4 is itself a clock opportunity: **extra clock ⟺
  CARU.q = 0 ∧ not stopped** — a plain level test at the enable edge.
```

## The counter cells

CH2's counter (CH1/CH3/CH4 mirror cell-for-cell; CH3's is 8-bit) is a
6-bit ripple up-counter loaded from NR21[5:0] on an NRx1 write and
counting on the gated 256 Hz tick:

| Bit | Cell | Load value | Load enable | Count clock | Notes |
|:---:|------|------------|-------------|-------------|-------|
| 0 | ERYC | NR21[0] | `bymo` | DYRO (= NOT(DEME), the gated 256 Hz tick) | First stage |
| 1 | CERA | NR21[1] | `bymo` | ERYC ripple | |
| 2 | CONU | NR21[2] | `bymo` | CERA ripple | |
| 3 | CAME | NR21[3] | `bymo` | CONU ripple | |
| 4 | BUVA | NR21[4] | `aget` | CAME ripple (via BUKO) | Upper bits load via `aget` — a fan-out split of the same NRx1 strobe |
| 5 | AKYD | NR21[5] | `aget` | BUVA ripple | `q_n` = the channel's length-stop flag |

`tffnl` has no reset pin. Compare the duty counters
([CH1/CH2](apu-ch1-ch2.md)), which are `dffr` cells reset by `apu_reset`:
the two counter families sit on opposite sides of the reset tree, and
that is the entire DMG-vs-power-cycle story below.

## The length clock and its gate

The tick is `bufy_256hz` — the CARU tap of the frame-sequencer ripple
([APU clocks](apu-clocks.md)) — gated per channel by a NOR3:

> CH2: `deme` = NOR(`cyre`, `bufy_256hz`, NOT(NR24[6]))

(CH1/CH3/CH4: CAPY/DODA/GEPY with their NRx4 bits.) The counter's clock
pulses only with length-enable set and `bufy_256hz` low.

```admonish info "Measured: polarity"
`bufy_256hz` equals CARU.q exactly — verified at 1,266/1,266 sampled
points (dmg-sim measurement) — and the counter advances on
`bufy_256hz`'s **falling** edge (CARU 1→0, i.e. the frame-sequencer
ripple reaching an even step: the conventional "length clocks on even
steps"). To keep the two edge roles straight: the *rising* edge is the
re-lock step marker in the power-cycle closed form; the *falling* edge is
the counting edge.
```

## The NRx4-write extra length clock

Because the gate is a plain NOR, **enabling length is itself a clock
opportunity**: an NRx4 write taking bit 6 from 0→1 drives the gate's
enable input low, and the gate output rises — one extra count, identical
to a 256 Hz tick — **iff `bufy_256hz` is LOW at that instant** (and the
channel isn't already length-stopped).

```admonish tip "Rule: the extra clock is a level test"
extra clock ⟺ CARU.q = 0 (frame-sequencer step even) ∧ not stopped —
evaluated at the enable edge, nothing more.
```

After an in-game power-cycle the ripple holds its reset value until the
first 512 Hz edge, so an enabling write landing in that window fires (or
not) according to the power-on sub-step — the `a` bit of the re-lock
closed form. Measured on the hardware-pinned gambatte
`div_write_reset_length_counter_timing` ROM (dmg-sim measurement): power-on
at sub-step 14 leaves CARU high, the enabling NR24 write produces **no**
extra clock, the counter holds its value across the write, and the ROM's
NR52 expectation follows — with the opposite sub-step case predicted to
fire by the same level test.

## Across an NR52 power cycle (DMG)

Power-off does two things to a length counter, **neither of which is a
reset**:

1. **No clear** — the cells have no reset path; the value holds.
2. **No clocking** — power-off clears the register bank, so length-enable
   drops and the gate pins the clock; the counter cannot advance.

Preserved and frozen — exactly the behaviour blargg's
`08-len ctr during power` pins for DMG ("unaffected, and not clocked").
The *phase* of the length clock after power-on re-locks per the
frame-sequencer rule in [APU clocks](apu-clocks.md); the *value* is
governed entirely by the two points above.
