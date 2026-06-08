# CH3: Wave Channel

CH3 reuses the square channels' divider topology with two changes — a
single /2 prescaler (so the divider runs at 2 MHz, halving the overflow
interval: Pan Docs' `(2048 − period) × 2` T-cycles) and a 16-byte wave RAM
stepped by a 5-bit position counter in place of the duty machinery. The
distinctive CH3 behaviours — the trigger delay, the locked CPU access, and
the retrigger corruption — all live in the small synchroniser and SRAM
circuits this chapter maps.

```admonish abstract "At a glance"
- Same 4+4+3 divider topology as CH1/CH2, but clocked at **2 MHz** by a
  single-prescaler chain — and the trigger synchroniser samples on
  **`ch3_2mhz`↓**, the opposite edge family from CH1/CH2.
- **Triggers reset the wave position to 0** (unlike the duty counter), and
  a wrap interlock dwells the counter at 0 for one extra overflow.
- CPU wave-RAM access while CH3 runs goes **wherever CH3 points**; writes
  land unconditionally at CH3's current byte.
- Retrigger corruption is a **4-byte row copy on DMG too** — the
  single-byte-on-DMG rule is inaccurate against the gate-level model.
```

## Period divider

Identical 4+4+3 `tffnl` topology to CH1/CH2
([CH1/CH2](apu-ch1-ch2.md)), at 2 MHz and with no sweep:

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| CERY | CH3 prescaler /2 stage | dffr | `apu_4mhz` (via CYBO buffer) | Toggle; free-running; q = `ch3_2mhz` directly; only reset is `apu_reset` — **never reloaded by triggers** |
| `ch3_fdis` | Channel-disable latch | nand_latch | Set: trigger-derived; Reset: DAC-off / `apu_reset` | While set, gates the divider toggle clock low |
| HEFO → JUTY | Divider toggle clock | nor2 + not | `ch3_2mhz` gated by `ch3_fdis` | One toggle per T-cycle |
| KUTU KUPE KUNU KEMU | Divider bits 0–3 | tffnl | bit-to-bit ripple | Load enable `kyko` |
| KYGU KEPA KAFO KENO | Divider bits 4–7 | tffnl | ripple via KYRU inverter | Load enable `kaso` |
| KEJU KEZA JAPU | Divider bits 8–10 | tffnl | ripple via KESE inverter | MSB q = `ch3_ftick` — the overflow edge |
| HYFO + HUNO | Overflow detector | not + dffr (/2 toggle) | clk: NOT(`ch3_ftick`) | One-`ch3_2mhz`-cycle self-clearing pulse `ch3_frst`; also async-reset by triggers |
| `ch3_restart` | Trigger synchroniser q | dffr | FABO (= NOT(`ch3_2mhz`)) | Captures the NR34-bit-7 write on `ch3_2mhz`↓ — **opposite-phase sample edge to CH1/CH2** |
| HERA | CH3 divider load enable (active-low) | nor2 | `ch3_frst`, `ch3_restart` | Fanned through `jera`/`kaso`/`kyko` (fan-out split only) |

The period source is the latched NR33/NR34 bits (`drlatch_ee` cells
written by the `apu_wr ∧ ff1d/ff1e` strobes) — there is no sweep adder in
the path, and the reload is level-sensitive exactly as on CH1/CH2.

## Wave-position counter

A 5-bit ripple counter clocked by `ch3_frst` falling — one step per
overflow. Bit 0 selects the nibble; bits 1–4 drive the wave-RAM byte
address.

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| DERO | CH3 wave-position counter clock | not_x1 | NOT(`ch3_frst`) | Rising edge = `ch3_frst` falling |
| EFAR | CH3 wave-position bit 0 (`wave_nibble_sel`) | dffr | DERO | Toggle; reset by ETAN |
| ERUS / EFUZ / EXEL / EFAL | Bits 1–4 (wave-RAM byte address) | dffr | bit-to-bit ripple | Reset by ETAN; EFAL's `q_n` clocks FETY |
| ETAN | Wave-position counter reset (active-low) | nor2 | `ch3_restart`, FETY | Clears the counter on trigger or wrap |
| FETY | CH3 counter-wrap interlock (/2 toggle) | dffr | EFAL↓ (counter wrap 31→0) | Holds the counter at 0 for one extra overflow; cleared by the next `ch3_frst`↑ |

- **Triggers reset it to 0** — `ch3_restart` is an input to ETAN. Unlike
  the duty counter, wave position does *not* survive a retrigger. (This
  asymmetry is the netlist's answer to Pan Docs' CH3-vs-pulse retrigger
  contrast.)
- **The wrap interlock (FETY)** holds the counter at 0 for one extra
  overflow after wrapping 31→0 before the next count resumes.

## The trigger synchroniser and its delay

An NR34 trigger traverses four stages — with a fifth cell shaping the
pulse:

| Stage | Cell | Type | Clock / Enable | Role |
|:---:|------|------|----------------|------|
| 1 | GAVU | drlatch_ee | enable: `apu_wr ∧ ff1e` | Write-strobe latch — captures the trigger bit at T3/T4 of the write M-cycle |
| 2 | FOBA | dffr | `apu_phi` | M-cycle-boundary DFF |
| 3 | GOFY | nor_latch | — | Tag latch |
| 4 | GARA | dffr | `ch3_2mhz`↓ | q **is** `ch3_restart` |
| 5 | GYTA | dffr | one edge behind GARA | Async-resets GARA — shapes the pulse to exactly one `ch3_2mhz` cycle |

Measured end to end (dmg-sim measurement): write commits at T4-mid;
boundary DFF at T1 of the next M-cycle; `ch3_restart`↑ on the next
`ch3_2mhz`↓ (T2 under quickboot phase); cleared two T-cycles later. The
aggregate from write strobe to first overflow at period 0x7FF is 9–10
T-cycles — the synchroniser accounts for 6–8 of them (phase-dependent),
which is the silicon content of the community's "+6 T-cycle trigger
delay". The T2-vs-T4 placement is `apu_reset`-locked, like every other APU
phase ([APU clocks](apu-clocks.md)).

## Wave RAM: address mux, read strobe, CPU access

The wave RAM's 4-bit address is muxed between the CPU bus and the position
counter's bits 1–4, steered by `ch3_active` (a DFF re-timed copy of the
trigger-armed FOZU latch — cleared by DAC-off, length-stop, or power-off).

CH3's own byte read is strobed by `wave_data_latch` — `ch3_frst` pushed
through a three-DFF `apu_4mhz` synchroniser (BUSA → BANO → AZUS). Measured:
the strobe rises ~1.5 T-cycles after `ch3_frst`↑ and stays high ~1 T-cycle;
the captured byte holds on latch cells until the next strobe.

```admonish tip "Rule: locked access goes wherever CH3 points"
With `ch3_active`=1, a CPU read or write of FF30–FF3F ignores the CPU's
address entirely — the mux is on the counter side. Reads return CH3's
current byte when the CPU's strobe overlaps the `wave_data_latch` window
(and an open bus value otherwise); writes land **unconditionally** at
CH3's current byte (the SRAM accepts them whenever the CPU write strobe is
high — no gating by `wave_data_latch`).
```

```admonish info "Measured: the nine-subtest locked-write sweep"
The write path is pinned by a hardware-verified nine-subtest sweep
(samesuite `channel_3_wave_ram_locked_write`, dmg-sim measurement): each
NOP of delay slides the landing byte by exactly one position, all nine
landing addresses matching the test's expected pattern. The same sweep
pins a gating subtlety: between subtests the DAC is off, `ch3_fdis`
freezes the divider, and the read-strobe chain is **idle** — so the
retrigger corruption (below) does not fire. Emulators that let the divider
free-run through DAC-off windows fire it spuriously.
```

## Retrigger corruption: a 4-byte row copy

Triggering CH3 while it is reading corrupts wave RAM: the netlist's SRAM
produces a **4-byte row copy** on DMG — not the single byte the
widely-quoted rule describes.

> When `ch3_restart`↑ lands in the window where the SRAM bit-line
> precharge is off — `(AZUS | AZET) = 1`, spanning from `wave_data_latch`↑
> to one T-cycle past its fall (≈ 2 T-cycles per overflow) — then with
> `row = wave_position >> 3`:
> **`ram[0..3] ← ram[row*4 .. row*4+3]`**, other bytes unchanged.
> Row 0 is naturally a no-op (source = destination — no special case).

The mechanism, in the SRAM cells: word lines are sticky until the
word-line driver precharges, so the trigger's address snap to 0 leaves the
*old* row's word line high while row 0's rises; the bit lines still hold
the old row's data (precharge off); and the write-back loop commits that
data to every enabled row — the whole source row lands in row 0. Verified
across 12 retrigger events spanning rows 1–3 and both nibble phases
(dmg-sim measurements with the full SRAM model).

```admonish note "For implementors"
Evaluate `(AZUS | AZET)` at the `ch3_restart` capture edge; if set, do the
row copy. And gate the divider on `ch3_fdis`, or the window will be open
when hardware has it closed.
```

## Post-boot state

The boot ROM never touches CH3: divider, position counter, overflow chain,
and `ch3_restart` all at their reset defaults; `ch3_fdis`=1; DAC off.
**Wave RAM itself has no reset path** — it survives NR52 power cycles, and
its cold-boot content is undefined on silicon. See
[post-boot state](post-boot.md).
