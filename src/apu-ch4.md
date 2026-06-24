# CH4: Noise Channel

The noise channel's tone generator is a **15-bit LFSR** clocked by a
two-stage frequency timer: a **3-bit divisor prescaler** (reloaded on every
trigger) feeding a **14-bit shift divider** (never reset by a trigger). What
the netlist supplies is the load and gating network behind the trigger
quirks — the silicon reason a re-trigger's first sample lands at a different
phase than a cold trigger's, and the reason a channel-enabling trigger on a
non-zero divisor code is one cycle late.

```admonish abstract "At a glance"
- The "frequency timer" is **two reset domains**: a 3-bit divisor prescaler
  that a trigger reloads, and a 14-bit shift divider that a trigger **never
  resets** (only `apu_reset`). This is the *opposite* split from the pulse
  channels, where the prescaler free-runs and the divider reloads.
- Because the shift divider is never reset, a **re-trigger** (or any non-
  power-on re-enable) starts its first LFSR clock from the divider's retained
  phase, not from zero — a phase-dependent offset of up to one sample, at
  **every** shift (half-sample steps at shift 0, finer above).
- The trigger is also **synchronised to `hama_512khz`** (the 512 kHz APU clock):
  a NR44 write takes effect on the next rise, so the write→first-clock latency
  carries that 8 T clock's phase too — a second, divider-independent source of
  cold-vs-re-trigger variation.
- A **channel-enabling trigger on divisor code ≥ 1 is one `hama_512khz` period
  late**: the prescaler count is frozen while `ch4_fdis` is set. Divisor
  code 0 pre-loads the prescaler to its terminal count, so the freeze is a
  no-op (measured below).
```

## The frequency timer

```
ch4_1mhz ─→ jeso_512k ÷2 ─→ kanu ─(gated by ch4_fdis)→ divisor prescaler ─→ hyno ─→ gary
  (4 T)    (= hama_512khz, 8 T)                         jyco→jyre→jyfu          (enable)
                                                        (loads NR43[2:0])          │ gates
                                                                                   ▼
ch4_1mhz ───────────────→ noise_counter_clk = ch4_1mhz & gary ──→ 14-bit shift divider
                                                                  cexo…esep ─tap(shift)→
                                                                  ch4_lfsr_clk → LFSR shift
```

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| `ch4_1mhz` (BAVU) | Noise base clock | not | apu 1 MHz | ~1.048 MHz (4 T); divider and prescaler source |
| JYCO JYRE JYFU | Divisor prescaler bits 0–2 | tffnl | toggle clk `kanu` (bit 0), ripple | Loaded from NR43[2:0] (`ff22_d0..2_n`) via `l(huce)`; **reloaded on trigger** |
| HYNO | Prescaler terminal-count | and3 | — | `AND(jyfu,jyre,jyco)` — all-ones = terminal |
| GARY | Divider-clock enable latch | dffr | `gyba` (= NOT `ch4_1mhz`) | `d = hyno`; reset `guny = NOR(apu_reset, ch4_restart)` — **trigger clears it**; gates the shift divider and reloads the prescaler |
| HUCE | Prescaler load enable | not | — | `huce = NOT gofu = ch4_restart OR gary` — reloads JYCO/JYRE/JYFU on a trigger and on every terminal count |
| CARY | Shift-divider clock | and2 | — | `noise_counter_clk = ch4_1mhz AND gary` |
| CEXO … ESEP | 14-bit shift divider | dffr (ripple ÷2) | `noise_counter_clk` (bit 0), ripple | **Reset only `apu_reset4_n` — never reloaded or reset by a trigger** |
| ETYR / ERYF (+ DARY, ELYX) | Divider tap muxes | ao / or | — | NR43 shift code (`ff22_d4..d7`) one-hot-selects one divider bit |
| FEME | LFSR shift-clock select | mux | `sel = ff22_d7` | `ch4_lfsr_clk1 = ff22_d7 ? etyr : eryf` (high vs low tap bank) |
| `ch4_fdis` (JERY) | Channel-disable latch | nand_latch | Set: DAC-off / `apu_reset`; Reset: trigger (delayed) | While set, holds the prescaler toggle clock (`kanu`) — see load-settle |
| JESO | Free-running ÷2 bit | dffr | `ch4_1mhz` | Reset `apu_reset5_n` — never trigger-reset; its buffered output **is `hama_512khz`** (the 512 kHz timebase) |
| KYKU→KANU | Prescaler toggle clock | or + not | `kanu = ch4_fdis OR jeso_512k` | `kanu` is the `tclk_n` of JYCO; `ch4_fdis = 1` freezes the prescaler |
| GONE | `ch4_restart` synchroniser | dffr | **`hama_512khz`** | Captures the NR44-bit-7 write, aligned to the next 512 kHz rise |
| GYSU | `ch4_start` | dffr | `apu_phi` | First synchroniser stage; drives the HAZO latch (→ `hazo_n`) feeding GONE |

Each shift-divider stage is a plain ÷2 toggle (`.d = q_n`, clocked by the
previous stage's `q_n`); the divisor prescaler stages are toggle-with-load
cells (`l = 1` → `q ← NR43 bit`; `l = 0` → toggle on `tclk_n`). `hama_512khz`
is the buffered free-running `jeso_512k` bit (a ÷2 of `ch4_1mhz`, ≈ 512 kHz);
the same bit, OR'd with `ch4_fdis`, is also the prescaler's toggle clock
`kanu`.

```admonish tip "Rule: the divisor prescaler reloads, the shift divider does not"
A trigger reloads JYCO/JYRE/JYFU (via `huce`) and clears `gary`; it leaves the
14-bit shift divider CEXO…ESEP **untouched** (its only reset is `apu_reset`).
This is the silicon form of "the trigger does not reset the noise frequency
divider": the divider keeps its phase across a trigger, so a re-trigger's first
LFSR clock is referenced to wherever the divider already was, not to zero. A
model that reloads the whole frequency timer on every trigger gets the first
*cold* trigger right (divider sitting at 0) but a re-trigger wrong.
```

```admonish info "Measured: how the period emerges (dmg-sim, divisor code 0)"
For divisor code 0 the prescaler loads to `111` = terminal immediately, so
`gary` stays high and the shift divider runs at the full `ch4_1mhz` rate
(4 T). The shift code taps divider bit `shift`, so the LFSR clocks every
`8 << shift` T — bit 0 (CEXO, 8 T) at shift 0, bit 2 (EZEF, 32 T) at shift 2.
Period = `divisor << shift` T, with divisor 8 for code 0.
```

## The LFSR

A 15-bit shift register (DFFs HENO/HEPA/HEZU/HORY/HYRO, HAPE/JAJU/JAVO/JEPE/
JUXE, KETU/KOMU/KUTA/KUZY/KYWY), all reset by `goge = NOT(ch4_restart OR
apu_reset)` — **a trigger returns the LFSR to its reset state** (post-boot
gives the value as `0x7FFF`). The chain shifts on the `ch4_lfsr_clk1` edge,
fanned out as `ch4_lfsr_clk2/3` across the 15 cells. The feedback DFF
(JOTO) latches `XNOR(HYRO, HEZU)` — the two XNOR taps — on `NOT ch4_lfsr_clk1`,
and HEZU is the output bit: `lfsr_out = AND(ch4_active, HEZU)`, gated into the
envelope at `dato`. In 7-bit (short) mode (`ff22_d3`) the AO22 cell KAVU also
injects the feedback into the mid-chain stage JEPE, shortening the active loop
to 7 bits. (The shift/feedback algebra is behavioural Pan Docs territory; it
is not the subject of this chapter.)

## Trigger synchronisation and the first-clock latency

A NR44 write with bit 7 set propagates `hoga` → `ch4_start` (captured on
`apu_phi`) → the HAZO latch → `ch4_restart` (captured on **`hama_512khz`**,
8 T). `ch4_restart` is held high for one full `hama_512khz` period (8 T);
during it `gary` is forced low (`guny`), so the shift divider is frozen, then
resumes when `ch4_restart` falls.

```admonish info "Measured: cold first-clock latency (dmg-sim, divisor code 0)"
From `ch4_restart`↑ to the first LFSR shift, with the divider starting from
count 0 (a cold trigger after power-on):

| period (= 8 << shift) | first LFSR clock after `ch4_restart`↑ |
|---|---|
| 8  (shift 0) | 12 T |
| 32 (shift 2) | 24 T |

i.e. **8 T** (the `ch4_restart`-high / divider-freeze window) **+ period/2**
(the zeroed divider counting up to the tap bit's first rise, at count
`2^shift`). Relative to the **NR44 write**, add the two-stage synchroniser
delay (`hoga`→`ch4_start` on `apu_phi`, then →`ch4_restart` on `hama_512khz`),
which depends on the write's phase: measured 5–9 T, giving write→first-clock
17–21 T at period 8 — consistent with the hardware-test-ROM `channel_4_delay`
figure of `period + 3 M-cycles` (±1 M), the synchroniser phase being that `±1 M`.
```

### Re-trigger: the free-running divider phase

Because the shift divider is never reset, a re-trigger — or any cold re-enable
that is not the first trigger after power-on — resumes from the divider's
retained count, and the first LFSR clock lands wherever the tapped bit *next*
rises.

```admonish info "Measured: re-trigger phase dependence (dmg-sim)"
First LFSR clock from `ch4_restart`↑, by the divider's frozen count. At
period 32 (shift 2, the tap spans eight counts):

| frozen count | first LFSR clock |
|---|---|
| 0 (= power-on cold) | 24 T |
| 2 | 16 T |
| 4 (tap bit just rose) | 40 T |

— a spread of nearly a full sample period (16–40 T). The same dependence shows
at shift 0 in half-sample steps: only CEXO is tapped, frozen at 0 or 1 →
**12 T or 16 T** (in `channel_4_lfsr_restart`, the power-on cold trigger starts
at count 0 → 12 T; later re-enables resume at CEXO = 1 → 16 T). A trigger that
freezes the divider at count 0 is indistinguishable from a power-on cold
trigger — nothing about a trigger resets the divider.
```

A second, divider-independent source of variation is the synchroniser itself:

```admonish info "Measured: the synchroniser adds write-phase jitter (dmg-sim)"
`ch4_restart` is captured on `hama_512khz`, so the **write→first-clock** latency
also carries the write's phase against that 8 T clock. Within one
`channel_4_lfsr_restart` [cold, re-trigger] pair the cold and re-trigger freeze
the divider at the same count — the same restart→first-clock (12 T in the first
pair, 16 T after) — while their two *writes* sit a half-`hama_512khz` offset
apart; the pair's first clocks then differ by exactly the **4 T (half a sample)**
synchroniser phase (sync 5 T vs 9 T, consistent across pairs).
```

A re-trigger therefore differs from a power-on cold trigger through *both* the
retained divider phase and the synchroniser phase. Sampled at one-period
spacing, that is the one-sample restart delay the hardware tables record
(`restart[i] = cold[i−1]`); a model that resets the divider on every trigger and
has no synchroniser reproduces neither, and clocks the restart one sample early.

### The fdis load-settle (divisor code ≥ 1)

```admonish info "Measured: cold-only +1 on non-zero divisor codes (dmg-sim, code 1)"
For divisor code ≥ 1 the prescaler loads below terminal and must count up; its
toggle clock `kanu = ch4_fdis OR jeso_512k` is **held while `ch4_fdis` is set**,
so a channel-enabling trigger cannot advance the prescaler until `ch4_fdis`
clears (one `hama_512khz` period after the trigger). Measured at divisor code 1
(period 16), same divider phase: a **cold trigger's first LFSR clock is 8 T
(one `hama_512khz` period) later than a re-trigger's** (24 T vs 16 T from
`ch4_restart`↑). For divisor code 0 the prescaler pre-loads to terminal, so
`ch4_fdis` never gates a count: this load-settle adds nothing at code 0 (the
period-8 cold-vs-re-trigger difference is the divider-phase and synchroniser
effects above, not this path).
```

This is the CH4 analogue of the pulse channels' `chN_fdis` load-settle, but it
gates the **divisor prescaler** (not the main divider) and the extra hold is
one `hama_512khz` period.

```admonish note "For implementors"
Three independent gate-level effects shape the noise first-clock timing; a
faithful model needs all three:
1. **Do not reset the shift divider on a trigger** — only reload the divisor
   prescaler and clear `gary`. The divider's retained phase sets the
   re-trigger first-clock at **every** shift; resetting it makes every
   re-trigger (and every non-power-on re-enable) look like a count-0 cold.
2. **Synchronise the trigger to `hama_512khz`** — the NR44 write takes effect
   on the next 512 kHz edge, so the write→first-clock latency carries that
   clock's phase: a second, divider-independent contribution to the
   cold-vs-re-trigger difference.
3. **Gate the prescaler count on `ch4_fdis`** for divisor codes ≥ 1 — a
   channel-enabling trigger's first clock is one `hama_512khz` period late;
   a re-trigger of a running channel is not. (No effect at divisor code 0.)
```

## Mid-run divisor-code changes (non-trigger)

`huce = ch4_restart OR gary` carries **no NR43-write term**, so writing NR43
while the channel plays does not itself reload the prescaler. The new divisor
code waits at the load inputs (`ff22_d0..2_n`) and is loaded only when `huce` is
next asserted — the reload that already fires on every terminal count. The
14-bit shift divider is untouched (its only reset is `apu_reset`), so it keeps
its phase across the change. A mid-run divisor-code change therefore takes effect
at a prescaler reload, not at the write instant.

```admonish info "Measured: a mid-run code change is captured at the next terminal (dmg-sim)"
Divisor code 1 → 2 (`$11`→`$12`, shift fixed), with the write phase swept in
4 T steps. At code 1 the prescaler period is 8 T — `gary`/`huce` high for ~4 T
after each terminal (the load window), then low while counting:

| write vs the load window | prescaler at the write | new code in effect |
|---|---|---|
| **inside** the window | reloads to the new code at once | ≈ at the write |
| **outside** (counting) | finishes its period at the **old** code | at the next terminal (≈ +3 T) |

The capture point tracks the prescaler terminal, **not** the write instant:
across the sweep the new cadence stays locked to the prescaler grid — its phase
set by the terminal that catches the change, never re-phased to the write.
```

Two corollaries follow. First, **a change made while the channel is on divisor
code 0 takes effect at once:** code 0 pre-loads the prescaler to terminal,
holding `gary`/`huce` high continuously, so the write is always inside the load
window (dmg-sim: `$18`→`$1a`, code 0 → 2, switches the shift divider's clock from
4 T to 16 T at the write). Second, the `ch4_fdis` load-settle is **trigger-only**:
a non-trigger write, even one *into* a divisor code ≥ 1, never re-asserts
`ch4_fdis` (its sole edge across each change is the trigger), so the cold
"+1 `hama_512khz`" does not apply mid-run. The latch point is set entirely by the
load-window phase and the code-0 case; the `huce`-driven load has no
divisor-*direction* term.
