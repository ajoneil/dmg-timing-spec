# LCD-on → First WODU

The end-to-end path from the CPU's LCD-enabling write to the first H-Blank, and how it maps onto CPU-observable timing ([scanline and frame timing](../scanline-frame-timing.md)).

## The end-to-end path

The full path from `ff40_d7`↑ (the LCD-enabling write's effect) to the
first H-Blank condition on the first scanline — SCX=0, no sprites, no window
(dmg-sim measurement):

| Stage | Δ from `ff40_d7`↑ (dots) |
|-------|-------------------------:|
| → XODO↓ (VID_RST deasserts) | +0.001 |
| → first XUPY↑ (scan counter starts) | +0.480 |
| → first AVAP↑ (Mode 2 → 3) | **+78.491** |
| → first SACU↑ | +85.5 (= AVAP + 7.026) |
| → first WODU↑ | **+251.536** (= AVAP + 173.045) |

With two dot-of-M-cycle invariants: the write's `ff40_d7`↑ lands at
**+2.020 dots** into its M-cycle (the CUPA position), and the first WODU↑
lands at **+1.556 dots** into its M-cycle. Both are CPU-prelude-invariant —
ps-identical across five LCD-on events in three radically different
instruction streams (quickboot's mid-boot enable, a test ROM's own
enable, and a post-HALT interrupt-handler enable, twice) — the
instruction stream between the write and the first WODU is irrelevant to
the PPU cascade.

The first-scanline Mode 2 measured from the write spans ~78.5 dots rather
than 80 — not a Mode 2 shortening, but the dot-0 reference shifted by the
write phase (+2.020) and divider startup (+0.480); each subsequent Mode 2
is exactly 80 dots.

**SCX extension** (measured at SCX ∈ {0,1,2,3,7}): the entire +1-dot-per-
SCX&7 shift lands in the startup pipeline; every other stage is
SCX-invariant to picosecond precision. Total: **251.536 + (SCX & 7)** dots,
giving a first-WODU M-cycle phase of `(1.556 + (SCX & 7)) mod 4`.

### Mapping to CPU observables

For the canonical `LDH (LCDC),a; xor a; inc a; inc a; …` prelude, walking
the wall-clock M-cells from the write pins everything: 63 CLK9 edges
separate the write's M-cycle from WODU's, placing the first WODU at dot
1.556 of cell M65 with `inc#61` in flight — handler-entry A = 61, matching
the hardware-verified ROM expectation.

```admonish warning "Pitfall: the fetch-overlap cell"
A walk that charges 4 dots per instruction *after* the write's M3 comes
out 4 dots short, because the cell after a memory write hosts the writer's
internal M4 *and* the next instruction's fetch — the SM83 fetch overlap.
An emulator without that overlap cell computes `inc#62` in flight and
A = 62 at handler entry.
```

Per-SCX, the in-flight instruction and one further mechanism — a one-M-cycle
**dispatch slip** when the WODU-driven `int_pending` settles past the
dispatch-trigger setup boundary (first-WODU phases 2.556/3.556 — see
[interrupt dispatch](../interrupt-dispatch.md)) — combine into the
hardware-verified expectations:

| SCX | WODU M-cycle | phase | in-flight | slip | handler A |
|---:|:---:|---:|---:|:---:|---:|
| 0 | M65 | 1.556 | inc#61 | no | 61 |
| 1 | M65 | 2.556 | inc#61 | +1 | 62 |
| 2 | M65 | 3.556 | inc#61 | +1 | 62 |
| 3 | M66 | 0.556 | inc#62 | no | 62 |
| 4 | M66 | 1.556 | inc#62 | no | 62 |
| 5 | M66 | 2.556 | inc#62 | +1 | 63 |
| 6 | M66 | 3.556 | inc#62 | +1 | 63 |
| 7 | M67 | 0.556 | inc#63 | no | 63 |

### Skip-boot note

```admonish note "For implementors"
The first-scanline shortening happens once, in the first frame after the boot ROM's
LCD-on; by PC=0x0100 (hundreds of frames later) it is baked into the
machine's phase. Skip-boot emulators must adopt the
[post-boot state](../post-boot.md) values jointly — LX=98 with the divider
states as given — and must not synthesise a "pre-shortening" LX and
re-apply the shortening at an LCD-on edge that never happens on the
skip-boot path.
```
