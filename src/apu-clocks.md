# APU Clock Tree and Frame Sequencer

```admonish abstract "In this part"
Five chapters cover the APU at gate level: the clock tree and frame
sequencer (this chapter), the square channels'
[period divider and duty](apu-ch1-ch2.md), the
[wave channel](apu-ch3.md), the [sweep and envelope
timers](apu-sweep-envelope.md), and the [length counters and power-cycle
behaviour](apu-length-power.md).
```

The APU shares the die with the PPU and CPU but is structurally independent:
per-channel prescalers off a dedicated 4 MHz tap, and a frame sequencer
hanging off the timer's `reg_div16`. This chapter anchors every APU clock
edge against the CPU's T-cycle grid, and pins the frame sequencer's silicon
form — a 3-bit ripple counter with its own reset domain, whose re-lock rule
across an NR52 power-cycle has a measured closed form.

```admonish abstract "At a glance"
- APU register writes commit on the **same dot-2–3.5 window** as PPU
  register writes — `apu_wr` and CUPA are sibling buffers.
- The frame sequencer **is** a 3-bit ripple counter (CARU/BYLU/JYNA),
  clocked by the 512 Hz divider tap but **reset by `apu_reset`** — two
  different reset domains.
- `(reg_div16 >> 11) & 7` is the frame-sequencer step **only when
  Δ = 0** — which no release guarantees; the re-lock offset Δ has a
  measured closed form.
- The real boot ROM hands off with **Δ = 3** — not the divider phase, and
  not the quickboot harness's Δ = 7.
```

## The clock tree

```
atal_4mhz → AZOF → ATAG → apu_4mhz ──┬─→ APUV  (CH1 prescaler clock)
                                     ├─→ AZEG  (CH2 prescaler clock)
                                     └─→ CYBO  (CH3 prescaler clock)
cpu_wr → BAFU → apu_wr               (APU register-write strobe)
```

Phase relations:

- `apuv` / `azeg` / `cybo` — in phase with `atal_4mhz` (≈ +1 ns buffer delay);
- `apu_4mhz` — the opposite phase;
- `apu_wr` — in phase with `cpu_wr`.

`apu_wr` and the PPU's CUPA are sibling buffer outputs of the same source,
so **APU register writes commit on the same T3–T4 window as PPU register
writes** ([register writes](registers/writes.md)).

Measured T-cycle anchors (dmg-sim measurement; quickboot phase):

| Signal | Edge | Phase | Δ from `atal_4mhz`↑ |
|---|---|---|---:|
| AJER / ATEP / CERY (prescaler stage 1, all channels) | toggle | start of every T-cycle | +1,900 to +3,200 ps |
| `apu_wr` | rise | start of T3 | +7,500 ps |
| `apu_wr` | fall | mid T4 | +4,700 ps from T4 `atal`↓ |
| `ch1_1mhz` | ↑ / ↓ | T2 / T4 of every M-cycle | +5,100 / +4,700 ps |
| `ch3_2mhz` | ↑ / ↓ | T1+T3 / T2+T4 | +2,400 ps |

```admonish note "For implementors"
**The second prescaler stages are phase-locked at power-on, not
absolutely.** CALO/CEMO (and CERY's phase) lock to whatever M-cycle the
`apu_reset` release (NR52 bit 7 ← 1) landed in, then free-run; the
alternative phase rotation is silicon-equivalent. Skip-boot emulators
should pick one phase deterministically and never re-phase it — the
prescalers ignore register writes and DAC toggles entirely.
```

**Trigger-write timing in T-cycle terms** (NR14; NR24 mirrors it on `ch2_1mhz`): the
write commits at T4-mid of M-cycle N; the NR14 sub-strobe propagates at T1
of M-cycle N+1; and the `chN_restart` synchroniser samples it at the first
prescaler edge after the strobe — under quickboot phase, T2 of M-cycle N+1.
For CH1/CH2 the capturing edge is the first `chN_1mhz`↑ *after* the write
strobe falls; CH3's NR34 instead samples on `ch3_2mhz`↓, the opposite edge
family ([CH3](apu-ch3.md)).

## The frame-sequencer strobes

The 512/256/128/64 Hz strobes hang off `reg_div16`
([timer](timer.md)) — but in two structurally different ways:

**Family A — `reg_div16`-direct.** `horu_512hz` is combinational from
`reg_div16` bit 10 (through the BURE/FYNE/GALE/GEXY/HORU buffer chain) —
no DFF, no reset. It free-runs unbroken across everything except a DIV
write.

**Family B — the `apu_reset`-reset ripple.** The 256/128/64 Hz strobes tap
a 3-bit ripple counter clocked by the 512 Hz BURE and **async-reset to 0 by
`apu_reset`**:

```
BURE (512 Hz) ─clk→ CARU (/2 → 256 Hz) ─→ bufy_256hz  (LENGTH clock)
                      └─→ BYLU (/2 → 128 Hz) ─→ byfe_128hz / cate_128hz (SWEEP clock)
                            └─→ JYNA (/2 → 64 Hz) ─→ kene  (ENVELOPE clock)
```

`(CARU, BYLU, JYNA)` **is** the silicon frame-sequencer step counter.
Measured steady-state phases (dmg-sim measurement): all strobe edges land
at T2 with per-signal offsets of +6,400 to +9,700 ps, exact period ratios
1 : 2 : 4, and zero off-phase edges across hundreds of cycles. `kene`↓ —
the envelope advance — is the ripple's step-7→0 wrap.

## The bare DIV write

A DIV write clears `reg_div16` through `reset_div_n` (UCOB —
[timer](timer.md)) but does **not** assert `apu_reset`. Unlike a
power-cycle it therefore does not reset the Family-B ripple; it can only
**clock** it.

CARU's clock BURE rises when `reg_div16` bit 10 falls. A DIV write
landing while bit 10 is **high** forces it 1→0, and the resulting BURE↑
toggles CARU: the ripple steps once. While bit 10 is **low** the reset
makes no edge and the ripple holds. This is the same gate behaviour as a
DIV write that drops a TAC-selected bit clocking TIMA
([timer](timer.md)).

Measured (dmg-sim, two `late_div_write` FSTs one sub-step apart): a bare
write at `reg_div16 = 0x3FF` (bit 10 low) holds the ripple; at `0x400`
(bit 10 high) the reset drops bit 10 and CARU steps once. The BURE↑ that
clocks it trails the reset edge by ≈ one T-cycle, so the advance is a
**level** test on bit 10 at the write M-cycle, not a sub-ns
BURE-vs-`reset_div_n` race.

## The NR52 power-cycle re-lock rule

Because Family B resets at power-off and `reg_div16` does not, an in-game
`NR52 = 0 → 0x80` cycle desynchronises the two. The re-lock has a measured
closed form (dmg-sim measurement, eight power-cycle FSTs plus the quickboot
release, with the counter bits read directly):

```
Δ = (S_pon + a − 1) mod 8
    S_pon = (reg_div16_pon >> 11) & 7        # coarse 512 Hz step at power-on
    a     = 1 if (reg_div16_pon & 0x7FF) ≥ 1023 else 0
counter C = ((reg_div16 >> 11) & 7) − Δ  (mod 8)
```

| Strobe (consumer) | fires at | re-lock step |
|---|---|---|
| `horu_512hz`↑ (Family A) | — | no shift — pure `reg_div16` |
| `bufy_256hz`↑ (length) | CARU rising (C 0→1) | (Δ + 1) mod 8 |
| `byfe_128hz`↓ (sweep) | BYLU falling (C 3→4 / 7→0) | (Δ + 4) mod 8 |
| `kene`↓ (envelope) | JYNA falling (C 7→0) | Δ |

The sub-step threshold is pinned to a single count — a power-on at sub-step
≤ 1022 catches the in-flight BURE edge (a = 0); at ≥ 1023 it misses by one
256 Hz half-period (a = 1) — bracketed on both sides by measurement. It is
**one counter with one offset**: the three tick strobes are taps of the
same ripple, so they share its phase exactly — any per-strobe
non-uniformity is an artifact of comparing different counter transitions,
not a real difference between the strobes.

```admonish warning "Pitfall: deriving the frame sequencer from the divider"
1. **`(reg_div16 >> 11) & 7` is the frame-sequencer step only when Δ = 0 —
   which no release guarantees.** A model deriving the frame sequencer
   purely from the divider is wrong across any in-game power-cycle. The
   step counter must be modelled as this separately-reset ripple.
2. **The boot path does not synchronise it either.** The real DMG boot
   ROM's handoff leaves the ripple at a directly-measured `(0, 1, 0)` →
   **Δ = 3** (dmg-sim measurement with the real boot ROM) — not the
   divider phase, and not the quickboot harness's Δ = 7. This seed decides
   real-hardware outcomes for tests that never touch NR52 or DIV
   ([sweep and envelope](apu-sweep-envelope.md)).
```

The single off-phase transient at a power-cycle (one strobe edge at the
wrong T-phase from the `reg_div16` restart, then steady-state lock) is
measured and single-cycle.

## Prescaler vs frame sequencer: the cross-grid phase

Both grids share the single `apu_reset`↓ anchor, and their sub-M-cycle
relationship is fixed: measured at a power-cycle re-lock (dmg-sim
measurement), the prescaler edge `ch2_1mhz`↑ and the envelope edge `kene`↓
land in the **same T-cycle (T2)**, prescaler first by **6,291 ps** — and
`kene`↓'s absolute phase (T2 + 9,597 ps) reproduces the steady-state strobe
anchor exactly. An independent raw-VCD check confirms `ch2_1mhz` is a
clean ~976 ns square wave (no sub-ns pulses).

Why this matters: the trigger synchroniser's load window opens at the
prescaler edge (+~4 ns), so an envelope tick landing ~6 ns later falls
*inside* the load window — the mechanism behind the envelope "+1 quirk"
case in [sweep and envelope](apu-sweep-envelope.md). The cross-grid phase
is not a free parameter; it composes directly with the strobe table above.
