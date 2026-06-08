# The Fetch State Machine

The sprite fetcher is a one-shot trigger chain, a 3-bit counter on the
opposite clock edge from the BG fetcher, and a running latch (TAKA) whose
lifetime extends well beyond the fetch itself.

```admonish abstract "At a glance"
- Trigger: **TEKY → SOBU (DFF) → RYCE → TAKA**, a one-shot closed by SUDA;
  the counter runs 0–5 on ALET rising — the opposite edge from the BG
  fetcher.
- TEKY's AND4 requires FEPO (X match), TUKU (no window activation), LYRY
  (BG fetch complete), and SOWO (no fetch already running).
- WUTY ends the fetch at **+6.010 dots from TEKY↑**, clears TAKA, and
  gates the shifter load.
- The BG drain cascade is **pinned throughout** — no NYXU fires during a
  sprite fetch.
- TAKA is high for *most* of every scanline: ATEJ re-asserts it at each
  line end, and the startup TAVE clears the carry-over.
```

## The gates

| Gate | Role | Type | Clock / Trigger | Notes |
|------|------|------|-----------------|-------|
| TOXE / TULY / TESE | Fetch counter bits 0–2 | dffr | SABE, then ~Q ripple | Reset by TAKA↑ |
| SABE | Gated fetch clock | nand2 | LAPE, TAME | Advances on ALET rising — opposite edge from the BG fetcher |
| TAME | Self-stop | nand2 | TESE, TOXE | Freezes SABE at count 5 |
| TEKY | X-match trigger | and4 | FEPO, TUKU, LYRY, SOWO | TUKU = NOT(RYDY) blocks during window activation; LYRY requires BG fetch complete; SOWO = NOT(TAKA) blocks re-trigger |
| SOBU | Fetch request | dffr | TAVA (clk5 = ALET rising) | Captures TEKY — the one DFF in the trigger chain. No reset domain (r_n tied high) |
| SUDA | Trigger one-shot partner | dffr | LAPE (clk4 = ALET falling) | Captures SOBU ~1 dot later; `suda_n` closes RYCE. No reset domain |
| RYCE | One-shot fire | and2 | SOBU, `suda_n` | High from SOBU's capture until SUDA mirrors it |
| SECA | TAKA set net | nor3 | RYCE, ROSY, ATEJ | Three set arms: Mode-3 trigger, LCD-off, and the **line-end pulse** |
| TAKA | Fetch-running latch | nand_latch | Set: SECA; Reset: VEKU = NOR2(WUTY, TAVE) | Freezes SACU via FEPO's persistence; WUTY = normal end-of-fetch, TAVE = startup carry-over clear |
| WUTY | Fetch done | not | VUSA | Clears TAKA; clocks the per-slot fetched-flag DFFs |
| VUSA | Fetch-done aggregate | or2 | TYNO, `tyfo_n` | Two parallel decode branches |
| TYNO | Done decode A | nand3 | SEBA, TOXE, VONU | |
| TYFO | Done decode B | dffr | LAPE | Captures TOXE; no reset domain |
| TOBU / VONU / SEBA | TULY capture chain | dffr | clk5 / clk5 / clk4 | Mode-3-only (r_n = mode3); feed TYNO |

## The trigger chain

> **TEKY → SOBU (DFF) → RYCE → TAKA**, with SUDA closing the
> one-shot

Measured with zero variance (dmg-sim measurement, purpose-built
`sprite_steady` ROM —
one sprite at SPRITEX 80, 64 firing scanlines):

| Stage | Δ from TEKY↑ (dots) |
|-------|---------------------|
| TEKY rises (ALET falling edge) | 0.000 |
| SOBU captures on the next ALET rising | +0.493 |
| RYCE fires (a gate delay from SOBU) | +0.493 |
| TAKA fires | +0.496 |
| Counter resets to 0 | ~+0.50 |
| SUDA captures, closing RYCE | +0.997 |

The counter then counts 0→5 over five dots (one tick per ALET rising via
SABE); at count 5 WUTY fires and clears TAKA at **+6.010 dots from TEKY↑**.

The net penalty is 6 dots (six suppressed SACU edges; the TEKY↑ → TAKA↓
freeze is 6.010 dots gate-resolved), zero SACU transitions inside the
window, SACU resuming 1.024 dots after TAKA falls (the next ALET falling
edge) — developed with its
alignment overhead in [the penalty model](penalty-model.md).

A brief TEKY glitch pulse (~0.016 dots ≈ 3.9 ns) appears as TAKA clears
and FEPO deasserts (combinational race); it is too narrow for SOBU's
capture and has no pipeline effect.

## The BG cascade freezes too

Throughout each TAKA-high window, the BG drain-detect cascade is pinned:
SEGU stuck at 1 gives RYFA no clock edges (frozen at 1 — the pre-freeze
capture of PANY=1 is guaranteed, because LYRY=1 was required for TEKY);
RENE keeps re-capturing RYFA=1; SEKO stays 0; TEVO and NYXU never fire. The
BG counter holds at 5 by the ordinary MOCE self-stop, and **no BG-shifter
parallel load occurs during a sprite fetch**. Verified across 30 TAKA-high
windows with zero NYXU/RENE/RYFA/SEGU/SEKO transitions inside any of them,
while the same scanlines show the normal 22-pulse NYXU cadence outside the
cluster (dmg-sim measurement, the gambatte `10spritesPrLine` stacked-sprite
ROM).

## OAM byte capture during the fetch

The fetcher has no dedicated tile/attribute DFFs — it **reuses the
two-stage capture chain** that Mode 2 uses for (Y, X), with the byte roles
following the address:

| Stage | Cells | Mode 2 (scan) | Mode 3 (fetch) |
|-------|-------|----------------|----------------|
| 1, bank B | YDYV…ZECA | Y byte (OAM byte 0) | **tile index (byte 2)** |
| 1, bank A | XYKY…ECED | X byte (byte 1) | **attribute (byte 3)** |
| 2, Y-side | XUSO, XEGU, YJEX, XYJU, YBOG, WYSO, XOTE, YZAB | Y-compare operand B | sprite tile VRAM row bits |
| 2, X-side | YLOR, ZYTY, ZYVE, ZEZY, GOMO, BAXO, YZOS, DEPO | X-store fan-out | attribute bits: GOMO = palette, BAXO = X-flip, YZOS = Y-flip, DEPO = priority (bits 0–3 unused on DMG) |

In Mode 3 the capture enable reduces to `oam_data_latch` = WEFY =
AND2(TUVO, `tyfo_n`), with TUVO high while the fetch counter ∈ {0, 1} — one
pulse per fetch, near TAKA-rise, capturing the byte-pair at the
fetcher-driven address (sprite index × 4 + 2, + 3). Stage 2 is transparent
through most of Mode 3, so the bytes flow straight through.

```admonish warning "Pitfall: the capture is not gated off under DMA"
Under DMA, the fetcher's bus driver is tri-stated but neither capture
enable is gated — each fetch's pulse captures the byte-pair at **DMA's
current address** instead: garbage tile index to the VRAM path,
DMA-source-byte attribute bits. Measured: 10 capture pulses across one
DMA + Mode 3 overlap, each at TAKA=1/counter=0, each overwriting Stage 1
with the byte-pair at the then-current `dma_a` (dmg-sim measurement,
Hacktix `strikethrough` ROM).
```

## TAKA outside Mode 3

TAKA's set net SECA = NOR3(RYCE, ROSY, ATEJ) includes the **line-end
pulse**: ATEJ fires at every scanline boundary (including VBlank lines) and
re-asserts TAKA during H-Blank. TAKA is therefore high for *most* of every
scanline — low only between the startup carry-over clear (TAVE↑ at
AVAP+5.996; TAKA↓ at +5.996) and the line's first real fetch, and between
fetch completions. The carry-over is cleared each scanline by TAVE through
VEKU's second reset arm.

Measured per-scanline waveform (zero variance,
six scanlines): ATEJ↑ at AVAP+376.000 dots sets TAKA (= the scanline
boundary: 456 − 80 dots of Mode 2); ATEJ pulse width 1.001 dots (one
half-XUPY cycle); TAKA holds through H-Blank until the next line's TAVE.

A side effect of the carry-over clear: SOWO = NOT(TAKA) rising at +5.999
briefly satisfies TEKY's AND4 — a glitch pulse too narrow for SOBU to
capture.
