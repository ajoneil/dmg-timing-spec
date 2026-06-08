# Sprite Storage and X Matching

Fetched sprite pixels live in two dedicated 8-bit shift registers whose load
path implements sprite-to-sprite priority, and a bank of SACU-clocked
comparators decides — pixel by pixel — when a sprite fetch must fire.

```admonish abstract "At a glance"
- Two 8-bit planes with **per-stage transparency-gated parallel load**:
  priority is implemented in the register — the first-fetched
  (lowest-OAM-index) sprite's opaque pixels survive.
- Stage-0 cells have d = const0 and the shifters have **no reset domain**:
  content enters only via load and persists across LCD-off, unobservably.
- Ten **combinational per-slot comparators** (stored X vs PX) reduce
  through FEFY/FOVE to **FEPO** — the CLKPIPE freeze that is the sprite
  penalty mechanism.
- The store holds **NOT(X)**: the reset state decodes as X = 0xFF —
  unreachable — which is why a reset slot never fires and a written X=0
  does.
- The VEZO→VAVA mask pipe carries **BG-over-OBJ priority** (OAM attribute
  bit 7), not validity.
```

## Sprite pixel shift registers

| Gate family | Role | Type | Clock / Trigger | Count | Notes |
|-------------|------|------|-----------------|-------|-------|
| Sprite shifter plane A | Sprite tile data plane 0 | dffsr | SACU | 8 | NYLU–VUPY range |
| Sprite shifter plane B | Sprite tile data plane 1 | dffsr | SACU | 8 | NURO–WUFY range |
| Sprite temp A | VRAM bus capture, plane 0 | dlatch_ee_q | Enable: `latch_sp_bp_a` | 8 | PEFO, ROKA, MYTU, RAMU, SELE, SUTO, RAMA, RYDU |
| Sprite temp B | VRAM bus capture, plane 1 | dlatch_ee_q | Enable: `latch_sp_bp_b` | 8 | REWO, PEBA, MOFO, PUDU, SAJA, SUNY, SEMO, SEGA |
| Per-stage load gate | OR3 per stage | or3 | combinational | 8 | MEFU, MEVE, MYZO, RUDA, VOTO, VYSA, TORY, WOPE = OR3(XEFY, `sprite_px_aN`, `sprite_px_bN`) |
| Per-stage load enable | `sprite_onN` driver | not | per-stage OR3 | 8 | LESY, LOTA, LYKU, ROBY, TYTA, TYCO, SOKA, XOVU |

The 16 shifter cells: LEFE, LESU, MASO, NATY, NURO, NYLU, PEFU, PYJO, VAFO,
VANU, VARE, VUPY, WEBA, WORA, WUFY, WYHO. Chain order from dffsr `d`-pin
connectivity:

- **Plane A**: NYLU (stage 0, d = const0) → PEFU → NATY → PYJO → VARE →
  WEBA → VANU → VUPY (stage 7) — outputs `sprite_px_a[0..7]`.
- **Plane B**: NURO (stage 0, d = const0) → MASO → LEFE → LESU → WYHO →
  WORA → VAFO → WUFY (stage 7) — outputs `sprite_px_b[0..7]`.

Stage-7 outputs feed the pixel mux ([LCD output](../lcd-output.md)).
Stage-0 cells have d = const0 — content arrives only via parallel load.

## The transparency-gated parallel load

Unlike the BG shifter's single broadcast LOZE, each sprite stage gates its
own load: `sprite_onN = NOR3(XEFY, sprite_px_aN, sprite_px_bN)`, with XEFY =
NOT(WUTY) the fetch-done pulse.

During a fetch (WUTY=0, XEFY=1) every stage's load is disabled — and SACU is
frozen anyway. At fetch completion, WUTY pulses, XEFY drops, and
`sprite_onN` = NOR2 of the stage's current bits: **the load fires only where
the current content is transparent (both planes 0)**. Opaque stages keep
their content.

```admonish tip "Rule: priority lives in the load path"
The per-stage transparency gate *is* sprite-to-sprite priority: the
first-fetched sprite's opaque pixels survive, and only still-transparent
positions accept the second sprite's data. Combined with OAM-index fetch
order, this yields the documented DMG rule — lower OAM index wins at the
same X.
```

| Aspect | BG shifter | Sprite shifter |
|---|---|---|
| Load enable | Broadcast LOZE = NOT(NYXU) | Per-stage `sprite_onN` |
| Timing | AVAP / TEVO pulses | WUTY pulse per fetch |
| Scope | All 8 stages, unconditional | Transparent stages only |
| Role | Replace tile | Overlay sprite |

A broadcast load would lose overlap semantics; a transparency-gated BG load
would under-load tiles. The asymmetry is load-bearing.

The shifters have **no reset domain** — set/reset comes only from the load
path, so content persists across LCD-off. Residual content is unobservable:
VAVA gates sprite output at the mux so only active-sprite pixels reach the
LCD, and no fetch has loaded data until the first X-match of a scanline.

## Sprite X match

Matching is fully combinational from the store latches and the PX counter:

> stored bits ⊕ NOT(PX) → two NOR4s (per nibble) →
> NAND3 with AROR (the LCDC.1 gate) → per-slot active-low match →
> FEFY/FOVE → FEPO

The ten per-slot matches reduce through FEFY/FOVE to FEPO; FEPO=1 →
VYBO=0 → CLKPIPE freezes — the sprite penalty mechanism.

| Gate family | Role | Type | Clock | Count | Notes |
|-------------|------|------|-------|-------|-------|
| Per-slot X store | Holds **NOT(X)** — captures the inverted OAM X byte on the slot's save strobe | drlatch_ee | Enable: per-slot save strobe | 8 × 10 | Slot 0: XEPE, YLAH, ZOLA, ZULU, WELO, XUNY, WOTE, XAKO; save strobe FUXU (store-counter = 0 decode); `r_n` = per-slot reset |
| Per-slot comparator | XOR per bit vs NOT(PX), NOR4 per nibble | xor + nor4 | combinational | 10 × 10 | Slot 0: ZOGY, ZEBA, ZAHA, ZOKY / WOJU, YFUN, WYZA, YPUK → ZAKO / XEBA |
| Per-slot match | NAND3(AROR, low nibble, high nibble), active-low | nand3 | combinational | 10 | FEFY group: YDUG (slot 0), DYDU, DEGO, YLOZ, XAGE; FOVE group: EGOM, YBEZ, DYKA, EFYL, YGEM |
| FEFY / FOVE | Match aggregates (slots 0–4 / 5–9) | nand5 | — | 2 | |
| FEPO | Sprite X priority aggregate | or2 | FEFY, FOVE | 1 | Feeds VYBO — the CLKPIPE freeze ([the pixel clock](../bg-pipeline/pixel-clock.md)) |
| Per-slot reset | OR2(line reset, slot fetch-done) | or2 + dffr | — | 10 | Slot 0: DYWE = OR2(DYBA, EBOJ); EBOJ = DFF(d = GUVA, clk = WUTY) — sets when slot 0's fetch completes |
| Sprite attribute pipes | Per-pixel palette select + BG-over-OBJ priority | dffsr | CLKPIPE | 16 | LYME (output: `sprite_px_palette`), MODA, NUKE, PALU, ROSA, RUGO, SATA, SOMY, VOSA, VUMO, WODA, WURU, WYFU, XETE + VEZO → VAVA |
| VAVA / VEZO | Priority mask pipe MSB / LSB | dffsr | CLKPIPE | 2 | Load from DEPO (OAM attribute bit 7) |

The sixteen CLKPIPE-clocked dffsr cells beside the pixel planes are
**attribute pipes** — per-pixel palette-select and priority shift
registers loaded at fetch time — not match logic. Match state is never
clocked; it follows the store latches and PX combinationally.

**The mask pipe carries priority, not validity.** VEZO→VAVA holds the
per-pixel BG-over-OBJ flag loaded from DEPO (OAM attribute bit 7). VAVA
gates BG visibility at RYFU = AND2(RAJY, VAVA) in the output path: VAVA=0 —
sprite priority, BG suppressed; VAVA=1 — BG shows through. (Fine scroll is
clock suppression, not mask-pipe work.)

## Why a reset slot cannot fire

The store holds the X coordinate *inverted*, and the comparator compares
against the *inverted* PX bits — so a written slot matches at PX = X, and
the all-zeros reset state decodes as X = 0xFF, a coordinate the PX
counter (terminal count 167) never reaches. There is no hidden armed
flag: the "armed" state is the latch contents themselves.

- A **written X=0** stores 0xFF internally and fires at PX = 0.
- A **reset-held** slot stores 0x00 ≡ X = 0xFF and can never fire.
- The same mechanism stops refiring after a fetch: EBOJ (the slot's
  fetch-done flag) sets at WUTY and holds the slot reset, clearing its X
  back to the unreachable state for the rest of the line.

Measured (dmg-sim, a locally patched 1-sprite variant of
gambatte's `10spritesPrLine_10xposA7`, sprite at X = 0xA7): slot 0's
store reads 0x00 while reset-held and **0x58 = NOT(0xA7)** between save
and reset (22/22 lines); YDUG fires at exactly PX = 0xA7 (7/7); DYDU —
slot 1, reset-held at 0x00 — never fires across the capture even though
PX passes 0 every line.
