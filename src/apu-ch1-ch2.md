# CH1/CH2: Pulse Channels

The square channels' tone generation is two counters: an 11-bit **period
divider** that overflows every `0x800 − period` ticks, and a 3-bit **duty
step counter** advanced once per overflow. The behavioural surface is Pan
Docs territory; what the netlist supplies is the load network that makes
the trigger quirks *fall out* — including the exact silicon meaning of
"the low two bits of the frequency timer are not modified".

```admonish abstract "At a glance"
- The "frequency timer" is a **13-bit object**: a free-running 2-bit
  prescaler plus the 11-bit divider. Triggers reload only the upper 11
  bits.
- Trigger and natural overflow share **one load net** — there is no
  trigger-only load path, and the load is level-sensitive.
- **A channel-enabling trigger's first overflow comes one `chN_1mhz` cycle
  late**; a retrigger of a running channel and every natural overflow run at
  the steady period (Rule below).
- The duty counter clocks on `chN_frst` **falling** and resets only on
  `apu_reset` — fast retriggers starve it rather than resetting it.
```

## The period divider

```
apu_4mhz → prescaler /2 → prescaler /2 → chN_1mhz ─(gated by chN_fdis)→ toggle clock
   (CH1: AJER → CALO;  CH2: ATEP → CEMO)
                ↓
  11-bit divider: 11 × tffnl in three ripple sub-chains (4 + 4 + 3),
  joined by single inverters; all stages share one load enable:
      CH1: epyk = NOR(ch1_frst, ch1_restart) → fume/dega/dako
      CH2: duju = NOR(ch2_frst, ch2_restart) → cogu/erog/gypa
```

| Gate (CH1 / CH2) | Role | Type | Clock / Trigger | Notes |
|------------------|------|------|-----------------|-------|
| AJER / ATEP | Prescaler /2 stage 1 | dffr | `apu_4mhz` (per-channel buffer) | Toggle; free-running 2 MHz; only reset is `apu_reset` — **never reloaded by triggers** |
| CALO / CEMO | Prescaler /2 stage 2 | dffr | stage-1 ripple | Toggle; free-running 1 MHz; output (buffered / directly) is `chN_1mhz` |
| `ch1_fdis` / `ch2_fdis` | Channel-disable latch | nand_latch | Set: DAC-off / `apu_reset`; Reset: trigger (one cycle delayed) | While set, gates the divider toggle clock low |
| FULO→GEKU / CAMA→DOCA | Divider toggle clock | nor2 + not | `chN_1mhz` gated by `chN_fdis` | Bit 0 toggles on the active-high rise |
| GAXE HYFE JYTY KYNA / DONE DYNU EZOF CYVO | Divider bits 0–3 | tffnl | bit-to-bit ripple | Load enable `fume` / `cogu` |
| JEMA HYKE FEVA EKOV / FUXO GANO GOCA GANE | Divider bits 4–7 | tffnl | ripple via inverter (KYPE / sibling) | Load enable `dega` / `erog` |
| EMUS EVAK COPU / HEVY HEPU HERO | Divider bits 8–10 | tffnl | ripple via inverter (DERU / sibling) | MSB q = `chN_ftick` — the overflow edge |
| CALA + COMY / sibling pair | Overflow detector | not + dffr (/2 toggle) | clk: NOT(`chN_ftick`) | Produces the one-cycle self-clearing pulse `chN_frst`; also async-reset by triggers |
| `ch1_restart` / `ch2_restart` | Trigger synchroniser | dffr | `chN_1mhz` | Captures the NRx4-bit-7 write; aligns it to the next `chN_1mhz`↑ |
| EPYK / DUJU | Divider load enable | nor2 | `chN_frst`, `chN_restart` | Active-low; fanned through `fume`/`dega`/`dako` / `cogu`/`erog`/`gypa` (fan-out split only — no functional difference between sub-chains) |

Each divider stage is a toggle-with-load cell: `l`=1 → `q ← d`
combinationally (level-sensitive); `l`=0 → toggles on its ripple clock.

```admonish tip "Rule: the 2-bit prescaler is untouchable"
AJER/CALO (ATEP/CEMO) have no input from the trigger or the overflow;
their only reset is `apu_reset`. **This is the silicon form of "the low
two bits of the frequency timer are NOT modified on trigger"**: the
"frequency timer" is the 13-bit concatenation of prescaler + divider, and
triggers reload only the upper 11 bits. Under fast retriggers, the
free-running prescaler keeps carrying into the divider, the duty step
eventually advances — and a model that reloads all 13 bits per trigger
stalls forever.
```

The remaining structural facts:

- **The divider counts up to 0x7FF**; the MSB's fall marks overflow. The
  one-stage detector (CALA + COMY) produces the one-`chN_1mhz`-cycle pulse
  `chN_frst` that opens the reload.
- **Trigger and overflow share the same load net.** `chN_restart` and
  `chN_frst` are the two NOR inputs — there is no trigger-only load path.
- **The reload is level-sensitive**: during the window the divider tracks
  the period source combinationally; the captured value is whatever the
  source held when the load fell.

**Period sources.** CH2 (and CH3) load directly from the latched NRx3/NRx4
bits. CH1 loads from `acc_d0..10` — the **sweep shadow accumulator** —
which has three write paths: the sweep adder's sum (clocked per sweep
fire), and async per-bit set/reset from NR13/NR14 writes. A trigger
immediately after an NR13/NR14 write therefore loads the just-written
period (the async path has settled long before the synchroniser fires).

```admonish info "Measured: the natural-overflow reload"
A natural overflow reloads the divider to exactly `{NR14[2:0], NR13}` for
one `ch1_1mhz` cycle, then resumes counting (dmg-sim measurement,
purpose-built `ch1_retrigger` ROM).
```

### The reload window, edge by edge

| Edge | What happens |
|------|--------------|
| Divider MSB falls (overflow) | `chN_ftick`↓ → CALA↑ → COMY toggles → `chN_frst`↑ |
| `chN_frst`↑ | Load enable falls → all 11 stages enter level-sensitive load (`q ←` period source) |
| Next `chN_1mhz`↑ | AND(COMY, `chN_1mhz`) asserts COMY's async reset → `chN_frst`↓ |
| `chN_frst`↓ | Load enable rises → stages return to toggle mode holding the loaded value |
| NRx4 trigger write | Async-loads the period source (CH1: `acc_d8..10`; CH2: NR24 latches); the strobe arms the synchroniser |
| Next `chN_1mhz`↑ after the write | `chN_restart`↑ — the same load path opens, held while restart is high |
| `chN_restart_dly`↑ (one cycle later) | `chN_fdis` clears — the channel becomes audible |

### Trigger-vs-overflow on the same edge

When the trigger's synchroniser edge coincides with a would-be overflow,
the race resolves at the overflow-capture DFF (COMY): its async reset
(through DYRU = NOR(`apu_reset`, `ch1_restart`, DOKA)) asserts **~2 ns
after the edge — before any overflow ripple or load-induced MSB fall can
reach its clock pin** (measured: reset at +1,970 ps, earliest possible clock
attempt at +2,830 ps). Across all five sub-cases (divider at 0x7FE/0x7FF/
lower, period MSB set or clear): **no `chN_frst` pulse, no duty advance.**
The trigger reloads the divider and opens the channel one cycle later, but
the duty step is preserved (dmg-sim measurement, 41 trigger events).

The one distinct case: a trigger landing while an overflow window is
*already active* (COMY=1 from the prior edge) forces the same `chN_frst`
fall the natural self-clear would have produced — the counter advances
once, indistinguishably from the natural path.

### Trigger-to-first-overflow: the load-settle cycle

The divider's toggle clock (`GEKU` / `DOCA`) is `chN_1mhz` gated by
`chN_fdis`, so while the channel is disabled the divider holds its value. A
**channel-enabling** trigger loads the divider (`chN_restart`↑, above) while
`chN_fdis` is still set; the loaded value is held one extra `chN_1mhz` tick —
the divider cannot count until `chN_fdis` clears. A trigger of an
already-running channel (`chN_fdis` already 0) reloads while the toggle clock
is still running and counts on the next tick — no extra hold.

```admonish tip "Rule: only the channel-enabling trigger is one cycle late"
**Channel-enabling trigger → first overflow = (0x800 − period) + 1
`chN_1mhz` cycles.** A retrigger of a running channel — like every natural
overflow — takes the steady `0x800 − period`. Measured at `0x800 − period`
= 63: an enabling trigger's first interval is +64.4 cycles, then exactly
+63; a retrigger's first interval is +63, and toggling the DAC off then on
restores the +1 on the following trigger (dmg-sim measurement).
```

```admonish note "For implementors"
Apply the two-tick hold (the loaded value held for **two** consecutive ticks
before the first toggle) only on the trigger that re-enables the channel
(`chN_fdis` 1→0). Applying it to every trigger makes a retrigger's first
interval one tick too long — the boot chime's second trigger retriggers the
running channel, and the spurious +1 leaves the post-boot CH1 divider at
0x7F8 instead of 0x7F9. Collapsing the two ticks onto one edge makes the
enabling trigger's interval one tick too short — the off-by-one behind
several hardware-verified duty/envelope race outcomes.
```

## The duty step counter

A 3-bit ripple counter clocked by **`chN_frst` falling** — the end of each
natural-overflow window. Its only reset is `apu_reset`: triggers, DAC-off,
and channel-stop never touch it.

| Gate (CH1 / CH2) | Role | Type | Clock / Trigger | Notes |
|------------------|------|------|-----------------|-------|
| DAJO / CULE | Counter clock | not_x1 | NOT(`chN_frst`) | Rising edge = `chN_frst` falling |
| ESUT / CANO | Duty counter bit 0 | dffr | DAJO / CULE | Toggle; reset only by `apu_reset` |
| EROS / CAGY | Duty counter bit 1 | dffr_cc | bit-0 ripple | |
| DAPE / DYVE | Duty counter bit 2 (MSB) | dffr_cc | bit-1 ripple | |
| COSO CAVA CEVU CAXO / DOMO DYTA DOJU DOVE | Duty-pattern selects | nor2 ×4 | — | NRx1[7:6] decode, one per duty value |
| ENEK EZOZ CODO / EGOG DYMU DARE | Counter-state decodes | and2 / not | — | counter=6 / counter ∈ {6,7} / counter ∈ {0..5} |
| `ch1_pwm` / `ch2_pwm` | Duty waveform output | ao2222 | — | 4:1 mux: OR of four (decode ∧ select) pairs |
| DUWO / DOME | PWM latch | dffr | `chN_frst`↑ | Captures the *pre-advance* step; reset to 0 by `apu_reset` |
| COWE / CYSE | PWM gated by channel-running | and2 | — | ANDed with the envelope to form the 4-bit DAC input |

The duty decode:

| NRx1[7:6] | Pattern (high at step k) | Decode |
|:---:|---|---|
| 00 | 00000010 (12.5%) | counter = 6 |
| 01 | 00000011 (25%) | counter ∈ {6,7} |
| 10 | 00001111 (50%) | counter bit 2 |
| 11 | 11111100 (75%) | counter ∈ {0..5} — the 25% decode negated |

The capture-on-rise / advance-on-fall split means each overflow plays the
*pre-advance* step.

**First-trigger gating, decoded.** "Duty clocking is disabled until the
first trigger" is implemented one stage upstream: the `chN_fdis` disable
latch (set by DAC-off or `apu_reset`, cleared one cycle after a trigger)
gates the divider's toggle clock — no divider ticks, no overflows, no duty
clocks. "The first duty step plays as if it were 0" is the PWM latch:
reset to 0 by `apu_reset` and only re-captured at the first real overflow.

**Fast retriggers** starve the counter rather than resetting it: each
trigger reloads the divider before it can overflow, removing the only
advance path. The counter freezes at its pre-loop value — Pan Docs' "duty
step never advances" with the mechanism made precise.

**Post-boot state**: the boot chime leaves CH1's duty counter at **2**,
with the DAC on (NR12 = 0xF3 retained), the running latch set, and the
period divider mid-count at 0x7F9. Only NR52 clears the duty counter, so it
persists into the handoff. CH2 stays at 0. See
[post-boot state](post-boot.md). This post-boot step is the baseline the
fast-retrigger staircase below counts up from.

### The fast-retrigger phase staircase

A fast-retrigger loop freezes the counter at its running value, and *which* value
is a **staircase in the trigger phase**: while the divider free-runs, the counter
advances once per overflow, so the pinned step is just how many overflows landed
before the loop's first reload caught it — +1 step per overflow interval
(`0x800 − period` ticks).

```admonish info "Measured: the pinned-step staircase"
Sweeping the pin trigger one M-cycle later across 80 steps (period 0x7F0, a 64-T
overflow interval) walks the pinned counter **0→1→2→3→4→5** — one step per 64 T
of added delay. The boundary is sharp to one `chN_1mhz` tick: adjacent triggers
4 T apart give a clean one-step change, at counter 0→1 and at the silent→audible
**4→5** edge of the 50% pattern (dmg-sim measurement, purpose-built
`apu_ch1_duty_phase_sweep` ROM).
```

The emitted bit is the **pre-advance** step — `duwo` captures on `chN_frst`↑, the
counter advances on the following `chN_frst`↓ — so a counter pinned at *k* ≥ 1
plays step *k* − 1 (pinned at 0, having seen no overflow, the latch still holds
its `apu_reset` 0). The flip is therefore one step past the pattern edge: for the
50% pattern (high at steps 4–7) the bit is 0 for a counter pinned at 0–4 and high
at 5 (measured `chN_out` 0 → full-scale across 4→5).

```admonish tip "Rule: the trigger phase quantum is one tick"
A retrigger write lands on an M-cycle (4-T) boundary and `chN_restart`
re-synchronises it to the next `chN_1mhz`↑, so the trigger only moves in whole
`chN_1mhz` ticks — never a sub-tick. The pinned step changes once per overflow
interval; at a step boundary a single 4-T shift carries the pin across that
overflow's `chN_frst` fall and flips the captured count — **one overflow, not a
sub-`chN_1mhz` parity effect.** Which side of the edge a boundary trigger lands on
is the overflow-capture race of *Trigger-vs-overflow on the same edge* above.
```

### The inter-trigger audible window

With trigger spacing *longer* than the overflow interval, the full
chain — overflow → PWM capture → mixer — runs between triggers. Measured
across a 7-overflow inter-trigger window (dmg-sim measurement, gambatte
`duty3_pattern_pos7` ROM): the duty counter walks 0→7, the PWM latch holds
1 through six overflows of the 75% pattern and captures 0 at the seventh
(counter 6 decodes low), producing a sustained ~375 µs audible pulse and
then silence — every digital gate operating exactly as the cell map says,
with no hidden muting.

