# Sweep and Envelope

The sweep unit (CH1 only) and the volume envelopes (CH1/CH2) share one
silicon idiom: a small ripple up-counter loaded with `~pace` through a
level-sensitive load net, a fire latch that catches the counter's
saturation on a frame-sequencer strobe, and a pace-0 detector that holds
the fire latch in reset. Every documented quirk — "+1 on trigger near a
tick", pace-0 pausing, zombie volume, the saturation stop — is a
consequence of those gates.

```admonish abstract "At a glance"
- Both timers are **`~pace`-loaded up-counters** that fire on saturation —
  sweep at 128 Hz, envelope at 64 Hz.
- **Pace = 0 pauses, not "period 8"**: the pace-0 decode holds the fire
  latch in reset (the envelope's is doubly paused).
- The **"+1 quirk"** is the load window: a frame-sequencer tick landing
  inside the ~1 µs reload window is lost — first fire one period late.
- The **volume step is a two-edge event** (fire rise arms, fire fall
  commits) — every mid-channel NRx2 write subtlety follows from it.
- Saturation is a **stop latch, not a clamp** — an unstopped 4-bit counter
  would simply wrap.
```

## The CH1 sweep timer

A 3-bit counter clocked by the frame sequencer's 128 Hz tap and loaded
with NOT(NR10[6:4]) by either reload path through one shared net:

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| `cate_128hz` | Sweep toggle clock | not chain | = `byfe_128hz`, the frame-sequencer 128 Hz tap ([APU clocks](apu-clocks.md)) | Held low while `apu_reset`=1 |
| CUPO / CYPU / CAXY | Sweep counter bits 0–2 | tffnl | `cate_128hz`, then bit-to-bit ripple | Load: `q ← NOT(NR10[4..6])` — **no reset path at all** |
| COZE | Sweep counter-at-max detector | and3 | CUPO, CYPU, CAXY | High at counter = 7; drives BEXA's data |
| BAVE | Sweep pace = 0 detector | and3 | NR10[6:4] complements | Holds BEXA in async reset via BURY |
| BEXA | Sweep-fire latch | dffr | AJER (the 2 MHz CH1 prescaler) | Captures COZE; while high, reloads the counter; falls on the next AJER↑. **Held at 0 when pace = 0** |
| DAFA → CYMU | Load enable | nor2 + not | BEXA, `ch1_restart` | CYMU = OR(BEXA, `ch1_restart`) drives all three `l` pins |

The two reload paths:

- **Trigger reload** — the same `ch1_restart` synchroniser as the period
  divider; **no second synchroniser, no pending-reload latch**.
- **Self-reload on fire** — COZE (counter = 7) captured into BEXA on the
  next AJER edge; BEXA reloads the counter and drops one cycle later.

Fire interval = `pace × 1/128 s` (Pan Docs' rule, with the table exact for
every pace). **Pace = 0 pauses** via BAVE holding BEXA in async reset —
the counter still ripples and saturates invisibly.

```admonish tip "Rule: the +1 quirk is the load window"
`cate_128hz` free-runs; a trigger landing so that the 128 Hz tick falls
*inside* the ~1 µs load window loses that tick (the `l` pin overrides the
toggle path) — effective first-fire time `(pace + 1) × 1/128 s`.
Just-after, just-before, and inside-the-window are the three landing cases;
only the third loses a tick.
```

**On each fire, three independent things happen:**

1. The counter reloads (above).
2. The **sweep-adder commit** strobes (`ch1_freq_upd1/2`) fire — gated by
   AND3(BEXA, no-overflow, **shift ≠ 0**). Shift = 0 means the computed
   period is *not* written back (Pan Docs' rule, as a literal AND term).
3. The **overflow check** runs regardless of shift: with direction = add
   and adder carry-out, `ch1_sum_ovfl_n` falls, BONE rises, and the
   channel-stop OR4 clears the running latch — channel disabled, NR52
   bit 0 reads 0. With shift = 0 the "new period" is 2 × shadow, so any
   shadow ≥ 0x400 disables on the *first* fire.

```admonish info "Measured: the sweep timeline end to end"
The trigger-reload cascade (restart → CYMU → all three cells loaded in
~1.7 ns; window ≈ 975 ns = one `ch1_1mhz` cycle), the free-running ripple
(+1 per 128 Hz tick, ripple-binary), a pre-trigger fire on the NR10 write
itself (releasing BEXA's reset with COZE already saturated fires
immediately — harmless when the channel is stopped), the first natural
fire at `trigger→first-tick + (pace−1)` periods, the overflow disable
~1 µs after the fire, and steady-state fires at exactly `pace × 7.995 ms`
(sim time) thereafter (dmg-sim measurement). A hardware-verified NR52
probe between trigger and first fire reads the channel still enabled.
```

**Post-boot**: the sweep counter cells have **no reset at all** — their
power-on state is undefined on silicon, and the boot ROM never writes NR10
(pace 0 throughout), so BEXA stays reset and the counter state at handoff
is power-on residue. Tests needing a known state must write NR10 and
trigger.

## The volume envelope (CH2 cells; CH1 mirrors cell-for-cell)

Three pieces: the envelope counter, the fire latch, and the volume
up/down counter.

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| `kene` | 64 Hz envelope toggle clock | not chain from JYNA | JYNA = /2 of `byfe_128hz` | The frame-sequencer ripple's last stage ([APU clocks](apu-clocks.md)) |
| JORE / JONA / JEVY | Envelope counter bits 0–2 | tffnl | `kene`, then bit-to-bit ripple | Load: `q ← NOT(NRx2[0..2])` via JAKE |
| KYVO | Envelope counter-at-max detector | and3 | JORE, JONA, JEVY | High at counter = 7; drives JOPA's data |
| JUPU | Envelope pace = 0 detector | nor3 | NRx2[2:0] | Feeds both JOPA's reset (via HAFE) and HOFO |
| JOPA | Envelope-fire latch | dffr | `horu_512hz` | Captures KYVO; while high, reloads the counter; falls on the following 512 Hz tick. **Held at 0 when pace = 0** |
| HYLY → JAKE | Load enable | nor2 + not | JOPA, `ch2_restart` | JAKE = OR(JOPA, `ch2_restart`) drives all three `l` pins |
| HOFO | Volume bit-0 toggle clock | or3 | JOPA, JUPU, `ch2_eg_stop` | High while firing, paced-off, or stopped — the toggle commits on its **fall** |
| FENO / FETE / FOMY / FENA | Volume counter bits 0–3 | tffnl | HOFO, then direction-muxed ripple | Load: `q ← NRx2[7:4]` on trigger |
| (per-bit AO22 muxes) | Direction ripple selects | ao22 | NRx2[3] | Up: ripple on carry; down: ripple on borrow |
| EMYR / FYRE → GUFY | Saturation detectors | nor5 / nand5+not / or2 | — | vol 0 + down / vol 15 + up |
| HEPO | Saturation capture | dffr | JOPA↑ | Reset on trigger or `apu_reset` |
| JEME / `ch2_eg_stop` | Envelope-stop latch | nor_latch | Set: HEPO; Reset: trigger / `apu_reset` | Pins HOFO high — the saturation lockout |

Fire interval = `pace × 1/64 s`. **Pace = 0 is doubly paused**: JUPU both
holds JOPA in reset *and* pins HOFO high so the volume bit-0 clock can
never complete a pulse. (Pan Docs' "period 0 treated as 8" does **not**
apply to the DMG envelope — the gates pause it outright.)

```admonish tip "Rule: the volume step is a two-edge event"
HOFO rises at the fire (JOPA↑) and the `tffnl` slave commits the
toggle only on HOFO's **fall** (the natural JOPA↓ one 512 Hz cycle
later). Everything subtle about mid-channel NRx2 writes follows from it.
```

The mid-channel write cases:

- A pace→0 write *between* fires just freezes the volume (HOFO rises once
  and never falls — no toggle).
- A pace→0 write racing the fire itself: if the write commits **before**
  the `horu_512hz`↑ sample, JOPA is held in reset — tick suppressed; if
  **after**, JOPA has fired and the queued toggle completes — tick
  applied.
- A pace→0 write landing **inside the (JOPA↑, HOFO↓) window** freezes the
  in-flight toggle — the fire happened but the volume never steps.
- A direction flip mid-channel switches each ripple mux combinationally
  and can produce a spurious carry edge into the next bit — measured:
  `$50`→`$18` on a volume-5 counter lands at 10. One of two silicon paths
  behind "zombie mode" volume changes; the second — the per-write +1
  step — is measured below.

```admonish info "Measured: zombie volume stepping — every pace-0 write ticks the counter +1"
The NRx2 cells are transparent latches, and during each write strobe the
**whole data byte transiently reads `$FF`** (~16–19 ns) before settling to
the written value — measured directly: bits *written* 0 read 1 across the
transient. The **pace** bits dip JUPU, so HOFO completes one fall→rise pulse
and the volume counter gets exactly one clock; the **direction** bit
(NRx2[3]) reads 1 through the same transient, so the toggle commits — and the
whole carry ripple completes — while the muxes are still **up/carry**. The
step is therefore **+1 regardless of the direction the value selects**: a
decrease-mode write increments. Measured on a running CH2, pace = 0, zero
variance (dmg-sim, the purpose-built `zombie_x8_ch2` and decrease-mode
`zombie_x0_dec_ch2` ROMs): every `$18`-over-`$18` *and* every `$F0`-over-`$F0`
write steps +1, wrapping 15→0 freely — HEPO and JEME never fire under
pace = 0, so no stop exists (the "repeat to decrement" idiom). A pace 0→1
write still steps +1 (the transient dip still fires); a pace 1→0 return write
completes no HOFO pulse and does not step on its own — though in decrease mode
the transient's direction edges can switch a ripple mux combinationally (the
same spurious carry a deliberate flip produces), so `$F0`-over-`$F1` lands +2
where `$18`-over-`$19` holds.
```

**Saturation stop**: the saturated-low/high decodes (volume 0 + down /
15 + up) are captured by HEPO on each fire and latch the envelope-stop
(JEME), which pins HOFO high — there is no arithmetic clamp; an unstopped
4-bit counter would simply wrap. The next trigger clears both latches.

### The trigger window and the "+1 quirk"

Identical shape to the sweep: the reload window is one `ch2_1mhz` cycle
(measured ≈ 975 ns, opening ~950 ps after `ch2_restart`↑), `kene`
free-runs, and a 64 Hz tick inside the window is skipped — first fire one
full period late. Pan Docs documents this as the envelope's
trigger-near-a-tick reload quirk; the gate content is just the `l` pin
override.

### The three measured regimes

Three measured anchors pin the complete model (dmg-sim measurements,
gambatte `ch2_init[_reset]_env_counter_timing` ROMs):

1. **The suppression race** (`_3`): counter saturates 14.1 ms after the
   trigger; the ROM's pace→0 write commits 0.18 ms *before* the next
   512 Hz sample; JOPA never fires; volume stays 0 — silent, matching
   hardware.
2. **The frame-sequencer-phase race** (`_1`, env1 — a ROM that never
   touches NR52 or DIV, so its `kene` phase is the **boot ROM's leftover
   ripple seed, Δ = 3** ([APU clocks](apu-clocks.md))). At the real boot
   phase, the fire lands ~1 ms before the pace→0 write — but the write
   falls **inside the (JOPA↑, HOFO↓) window**, freezing the in-flight
   toggle: volume never steps, channel silent (dmg-sim measurement with
   the real boot ROM). Under the quickboot harness's wrong seed (Δ = 7)
   the same ROM comes out audible — a harness artifact, not hardware. The
   anchor is what pins the post-boot Δ = 3 requirement for skip-boot
   emulators.
3. **The +1-quirk case** (`_11`, a `_reset_` ROM that re-locks the phase
   deterministically): the 64 Hz tick lands ~1.5 ns *inside* the
   ~975 ns load window — measured with the counter provably not advancing
   across it — first fire one full period late, channel silent.

```admonish note "For implementors"
The unified rule: **one load-window test fixes the first-fire time** (hold
the counter at `~pace` while the load net is open; ignore any tick edge
inside it), and **the HOFO toggle-vs-pace-0-write race fixes whether a
fire steps the volume**. The two compose; no further mechanism is needed
across the test family.
```

**Post-boot**: CH1's envelope is saturated-stopped (volume decayed to 0,
stop latch set, counter holding `~3` from the chime's pace); CH2's is
all-zero with the stop latch clear. See [post-boot state](post-boot.md).
