# The IF Register

IF (FF0F) is five chip-level DFFs — one per source — outside the SM83 core,
each with **D tied high and the source as its clock**. That one structural
fact generates all the behaviour: edge-triggered capture, held-high sources
that cannot re-set after a clear, and two distinct clear-vs-rise races with
opposite outcomes.

```admonish abstract "At a glance"
- Each IF bit is a DFF with **D tied high, clocked by its source** — a
  held-high source cannot re-set a cleared bit.
- **FF0F-write clear vs same-cycle rise: rise wins** (the clear pulse
  closes before the mid-M-cycle sources fire).
- **Dispatch-acknowledge clear vs same-cycle rise: clear wins** (the ack
  pulse straddles the boundary and absorbs the edge).
- IF survives LCD-off — no first-frame VBlank suppression of any kind.
```

| Bit | Cell | Clock (source) | Source driver |
|-----|------|----------------|---------------|
| 0 (VBlank) | LOPE | `int_vbl` | POPU.q (mode1) through TOLU + VYPU |
| 1 (STAT) | LALU | `int_stat` | SUKO through TUVA + VOTY ([STAT interrupts](stat-interrupts.md)) |
| 2 (Timer) | NYBO | `int_timer` | MOBA.q — a CLK9-aligned DFF ([timer](timer.md)) |
| 3 (Serial) | UBUL | `int_serial` | CALY toggle at the transfer clock |
| 4 (Joypad) | ULAK | `int_jp` | AND2(APUG, BATU) — combinational, no source latch |

Each cell's `s_n` (FF0F write of 1) and `r_n` (FF0F write of 0, or dispatch
acknowledge) are active-low overrides. Since D is constant, **only the
clock matters**: a held-high source cannot re-set a cleared bit — a fresh
rising edge is required. That is the LALU edge-trigger rule generalised to
all five bits.

## Source cascades, measured

**VBlank** (POPU↑ → IF[0]):

> POPU↑ → two-NOT buffer (768 ps) → `cpu_irq0` (+1,064 ps) →
> `irq_pending` (+4,144 ps)

The whole chain is sub-dot. VBlank IF and the Mode 1 STAT condition co-rise
on the same dot with nanosecond ordering.

**The Mode 1 STAT leg at VBlank entry** depends on the active enables:

- **LYC leg active** — the entry traverses the SUKO glitch (the combining
  gate dips and recovers); IF[1] sets ~4.4 ns after POPU, ~3.6 ns behind
  IF[0].
- **Mode 0 + Mode 1 enables** — the Mode 1 arm covers SUKO before the
  Mode 0 arm drops, so **IF[1] does not change at all** at the boundary;
  the measured handler reads IF[0]=1, IF[1]=0.

(dmg-sim measurements; the case analysis is the leg-swap table in
[STAT interrupts](stat-interrupts.md).)

**The boundary into the CPU**: the per-bit IF&IE latch inside the SM83 is
an *enabled D-latch* transparent outside the data phase
([interrupt dispatch](interrupt-dispatch.md)) — a mid-M-cycle IF rise
reaches `irq_pending` with no clock-edge wait; the first CLK9-aligned
capture is YOII. Timer is the one boundary-aligned source (its driver is
itself a CLK9 DFF), which is the entire origin of the source-class split
in [HALT wake latencies](halt-ei.md).

## Race 1: CPU FF0F-write-clear vs same-M-cycle source rise

The FF0F write-clear pulse rides the CPU write strobe — measured
co-located with CUPA (T3 to mid-T4, width 1.493 dots,
matching to sub-ps). The mid-M-cycle PPU sources rise mid-T4 (+3.5 dots) —
*after* the clear pulse closes (measured: the reset releases 1,155 ps
before the VBlank clock arrives) — and the boundary-aligned timer source
rose in T1, *before* the write window opened. Either way:

> **Rise wins.** A source firing in the same M-cycle as a CPU clear of
> its bit leaves the bit set.

(The held-high case is the separate rule above: a clock that rose in an
*earlier* M-cycle and stayed high cannot re-set after the clear.)

## Race 2: dispatch-acknowledge clear vs same-M-cycle source rise

The acknowledge pulse is different: it asserts in T4 of the dispatch's
vector-resolution M-cycle and **releases only at the next M-cycle
boundary** — width ≈1.000 dot, straddling into the successor M-cycle's
opening. A source edge landing inside it — measured with a second STAT
dispatch deliberately aligned so the LY 143→144 SUKO-glitch edge fires
+0.529 dots into the ack pulse — is *captured but suppressed*: the DFF
sees the clock edge while `r_n`=0 holds Q at zero, and when the reset
releases the clock is already high, so no later edge exists.

> **Clear wins.** A source edge inside the acknowledge window is absorbed;
> IF stays 0 (dmg-sim measurement).

| Clear path | Pulse position | Width | vs +3.5-dot source rise |
|------------|----------------|-------|------------------------|
| FF0F write | +2.0–3.5 dots (T3–T4) | 1.493 dots | lands after — **rise wins** |
| Dispatch ack | +3.0–4.0 dots (T4 into next M-cycle) | 1.000 dot | lands inside — **clear wins** |

```admonish note "For implementors"
Emulators that clear the IF bit as a single step "at the end of dispatch"
miss the absorbing behaviour; the faithful model holds the reset active
for the full acknowledge window.
```

## Across LCD-off

LOPE (and its siblings) have **no LCD-off reset path** — their resets are
the FF0F-write and acknowledge chains plus system reset only. IF survives
LCDC.7 toggles; POPU (which *is* LCD-off-reset) restarts cleanly and the
first post-LCD-on VBlank entry sets IF[0] with the steady-state edge
structure — no first-frame gating of any kind (dmg-sim measurement;
[LCD-on power-up](scanline-frame-timing/lcd-on.md)).
