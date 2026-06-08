# LCD-on Power-up

What happens on the first frame after LCDC.7 goes high: the shortened first scanline, the startup transient at the LCD pins, and which PPU flip-flops are reset by what ([scanline and frame timing](../scanline-frame-timing.md)).

## The first scanline: the 454-dot line

Immediately post-LCD-on, the first scanline measures 453.5 ≈ 454 dots instead of
456, with the deficit entirely in Mode 0 — Mode 2 and Mode 3 match
steady-state. The gate-level decomposition (dmg-sim measurement):

| Component | First scanline | Steady-state | Diff |
|---|---:|---:|---:|
| Line-start reference → first LX-incrementing TALU↑ | 2.021 (XODO↓ phase) + 1.483 (divider startup) = 3.504 | 6.000 (first TALU↑ lands inside the RUTU pulse and doesn't count) | −2.496 |
| 112 further LX increments | 448.000 | 448.000 | 0 |
| LX=113 → SANU → RUTU | ≈2.0 | ≈2.0 | ≈0 |
| **Total** | **453.500** | **456.000** | **−2.5 ≈ −2** |

The structural sum lands at −2.5 while the directly-measured Mode-0 deficit
(below) is exactly −2.0 dots (488,000 ps): the ~0.5-dot difference is sub-dot
TALU phase, conserved through the LX chain and never realised as a whole-dot
change in line length, so the line runs an integer 454 dots and the Mode-0
deficit is exactly 2.

The deficit's location is measured directly: the first scanline's Mode 3 + Mode 0
span (XYMU↓ → RUTU↑) is 48,680,932 ps (199.512 dots) against 49,168,932 ps
(201.512) on scanlines 1–5 — shorter by exactly 488,000 ps = 2 dots — and
the pure-Mode-0 interval (XYMU↑ → RUTU↑) shows the same 488,000 ps gap,
placing the whole deficit in Mode 0 (dmg-sim measurement).

Three structural facts compose the deficit — and none of them is "drift":

1. **No leading RUTU pulse.** A steady-state line spends one TALU period
   inside the RUTU pulse where LX is held at 0; the first scanline starts counting
   immediately. Saves 6 dots against the reference.
2. **The LCD-on write lands in T3 of its M-cycle** (the CUPA position,
   +2.020 dots); VID_RST deasserts a gate delay later at XODO↓ (+2.021), so the
   chain's first effect is 2.021 dots late against the M-cycle boundary.
3. **The divider stack starts mid-phase**: the first TALU↑ lands 1.483 dots
   after VID_RST deasserts — 3.504 dots past the M-cycle boundary instead
   of the 6.000 a steady line spends reaching its first counting rise.

That phase is then **conserved exactly**: the write sets the divider phase
for the whole epoch, and every TALU↑ thereafter — all 113 on the first
scanline and every steady-state rise after — lands 854,890 ps past its M-cycle
boundary with zero variance (155,000+ edges, dmg-sim measurement). No
per-line adjustment exists anywhere in the chain. The LX chain is a
delay-conserving propagator, not an amplifier.

```admonish note "For implementors"
Reproduce the three structural inputs (write phase, divider phase, no
leading RUTU pulse) and let the trajectory fall out — patching the LX==113
detector misframes a mechanism that contains no scanline-specific gating.
```

## The LCD-on startup transient at the pins

| Pin | Pre-LCD-on | First edge after LCD-on | Cadence |
|-----|------------|--------------------------|---------|
| `s` (VSYNC) | static 0 | **+17.1 ms — the LY 153→0 boundary ending the first frame** | once per frame |
| `st` (HSYNC) | static 0 | +19.2 µs (first-scanline Mode 3) | per scanline |
| `cp` | static 0 | +20.7 µs | half-dot period during Mode 3 |
| `cpg` | static 0 | +6.7 µs | per scanline |
| `cpl` / `fr` | alternating at ~8/~4 kHz (audio mux) | seamless | per scanline |
| `ld0` / `ld1` | static 0 | first Mode 3 pixel | pixel rate |

Seven of eight pins are normal from the first scanline. **VSYNC is silent for all
of the first frame**: MEDA can only capture the LY=0 decode at an LY 153→0
wrap, and the first such wrap is the one *ending* the first frame. From the
second frame on it fires once per frame; the first frame is distinguished by
the absence of a pulse, not an altered one.

The community-documented consequence — the first frame never reaches the glass
("first frame is blank") — rests on pin-role references for the LCD's
Y-driver, not on a measured glass-side behaviour; the PPU-side mechanism
(MEDA's reset domain) is netlist-derived and measured. The 2-dot
first-scanline shortening changes no pixels — it only moves where the mode
and LY transitions fall — so it has **no display consequence**, but it is
fully observable to the CPU: STAT mode reads, LY reads, and per-scanline
interrupt timing all see it.

**VBlank IF is not gated by the first frame.** The IF[0] latch (LOPE) has no
LCD-off reset path; POPU resumes normally after LCD-on, and the first
frame's VBlank entry raises IF[0] with the same edge structure as any
steady-state frame (dmg-sim measurement) — there is no first-frame
interrupt suppression of any kind.

## Pipeline state at LCD-on: the reset-domain taxonomy

Every PPU DFF belongs to one of five reset classes, and the whole
cross-frame story follows from the classification:

| Class | Reset condition | Members | Behaviour |
|---|---|---|---|
| **Direct LCD-off** | `ppu_reset_n` / `ppu_reset2_n` | dividers WUVU/VENA/WOSU; RUTU, NYPE, POPU, MYTA, MEDA; NOPA | Held at 0 while LCDC.7=0; released at the VID_RST edge |
| **Indirect LCD-off** | combinational nets containing VID_RST terms | LX, LY, PX, VOGA, scan counter, fine-scroll counter, BG fetch counter, AVAP generator, sprite fetch counter, sprite store | Held at 0 during LCD-off; reset at their per-scanline/per-tile boundaries during rendering |
| **Mode-3-only** | the `mode3` net | LYZU, PUXA, NYZE, PYGO, RENE, RYFA, PAHO, SEBA, TOBU, VONU | Zero at every Mode 3 entry, on every scanline, every frame — identical by construction |
| **No-reset persistence** | tied-high `r_n` or data-driven set/reset only | BG + sprite shift registers (32 cells), tile temp latches, SOBU/SUDA/TYFO, XYMU, ROXY | Hold state indefinitely across LCD-off |
| **CPU-register** | CPU-write/system-reset only | LCDC, SCX/SCY, WX/WY, palettes, LYC, STAT enables, ROPO | Survive LCDC.7 toggles; palettes have *no* reset at all (power-on state undefined on silicon) |

What matters is whether any persisted stale state reaches LD0/LD1.
For the BG plane, no — the two-parallel-loads mechanism
([BG pipeline](../bg-pipeline/shifters.md)) propagates stale content at load #1 (AVAP)
and overwrites it at load #2 (first TEVO, dot 5.996) before the first
CLKPIPE shift at 7.026, and CP is PX≥9-gated besides. For the sprite plane,
stale shifter content is mux-gated until a real fetch fires. That leaves
exactly **two** pieces of cross-frame state to examine at the first
scanline; only the first reaches LD0/LD1:

1. **The sprite store is empty on the first frame** — the LCD-on bypass never sets
   BESU, CARE never arms, and `oam_data_latch` never pulses
   ([mode transitions](../mode-transitions.md)). Consequences by OAM content:

   | Case | OAM at LY=0 | Divergence on LD0/LD1? |
   |------|-------------|------------------------|
   | 1 | no Y-visible sprite | none — both frames render spriteless. Anchored across 777 Y-non-matching scanlines (zero TEKY/FEPO, zero variance) and 842 scanlines of DMA overlap across all three regimes: DMA cannot convert an empty-store line into an extended one |
   | 2 | Y-visible sprite, on-screen X | **yes** — the first frame misses the sprite; steady-state frames render it |
   | 3 | Y-visible sprite at OAM X=0 | timing only — a steady-state frame's save-written X=0 fires the X match and its fetch penalty; the first frame's reset-held store cannot fire (the store holds NOT(X); reset decodes as X = 0xFF — [sprite pipeline](../sprite-pipeline/storage-matching.md)) |

2. **The window's REJO latch.** REPU clears it on every VBlank entry
   ([window control](../window.md)), so it carries nothing across frames —
   no divergence from this path.

Everything else either lands outside the pixel path (MEDA's silent
first-frame VSYNC; BESU's missing Mode 2 status/IRQ on the first scanline of
the first frame) or is
overwritten before it can be sampled.

One cold-boot footnote: ROPO (the LYC-match capture) resets only at system
reset, so STAT bit 2 between LCD-on and the first TALU edge briefly exposes
the *pre-LCD-off* match value — and on the very first boot, the power-on
zero.
