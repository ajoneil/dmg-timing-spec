# The Source Bus

The OAM bus is one side of DMA; the other is the **source bus** DMA reads
from — the external bus (cart ROM/RAM, WRAM) or the VRAM bus. The question
here is what the *CPU* observes when its own read or write targets the same
physical bus DMA is driving.

```admonish abstract "At a glance"
- The **address pads are MUXed to DMA; the data pads and CPU strobes are
  not gated at all** — every conflict behaviour falls out of that one
  asymmetry.
- CPU read on the DMA bus: the CPU latches **source[`dma_a`]** — the byte
  DMA is fetching that M-cycle.
- CPU write on the DMA bus: **OAM gets the CPU's byte**, and the cell at
  DMA's source address takes a spurious write of it.
- HRAM is on-chip, on neither bus: CPU access during DMA is uncontended.
- Sources $FE/$FF fold into WRAM: `work_ram[(src − $E000) & $1FFF]` — the
  echo region extended one page.
```

## Source-side bus arbitration

One asymmetry drives every conflict on this page: during DMA the
**address pads are MUXed to DMA**, but the **data pads and the CPU's
read/write strobes are not gated at all**.

| Gate | Role | Type | Logic |
|------|------|------|-------|
| LOGO | DMA bus-read window | not_x1 | NOT(RORU) — per-T-cycle DMA address window |
| MORY | DMA-vs-CPU address MUX select source | nand2 | NAND2(`dma_run`, LOGO) |
| LUMA | `dma_addr_ext` driver | not_x1 | `dma_addr_ext` = AND2(`dma_run`, LOGO) |
| per-bit address MUXes | address pad source | mux | sel = `dma_addr_ext`: DMA address vs CPU address |
| CEDE | inverted `dma_addr_ext` | not_x2 | gates CPU-side address-pad drivers off during DMA |

So whatever the CPU's instruction intended, the address reaching the bus
during DMA is DMA's:

| CPU target | `dma_addr_ext` during the CPU strobe | Address pads carry |
|------------|:---:|---------------------|
| external bus, DMA source on external bus | 1 | DMA's address — the CPU's intended address never reaches the pads |
| VRAM bus, DMA source on VRAM | 1 | DMA's address |
| HRAM ($FF80–$FFFE) | irrelevant | HRAM is on-chip, on neither bus; CPU access proceeds normally |

```admonish warning "Pitfall: there is no address collision to detect"
An emulator cannot model the conflict as "did the CPU's address collide
with DMA's?" — the CPU's address has already been *replaced* at the pads.
```

## CPU read conflict

The CPU's read strobe asserts on its normal cadence; the address pads carry
`dma_a`; the memory drives source[`dma_a`] onto the data pads; the CPU
latches it. The CPU's register receives the same byte DMA is fetching this
M-cycle — equivalently OAM[`dma_dest`] after this M-cycle's commit.

Anchored end to end (dmg-sim measurement, gambatte
`oamdma_src0000_busyread0000_1` ROM): at the conflict M-cycle the pads show
DMA's address $009E, the CPU's sample edge latches 0x04 = source[$009E],
and the DMA OAM commit of the same byte fires co-temporally on a distinct
strobe chain — A = 0x04, matching the hardware-verified `out4`.

## CPU write conflict

The CPU's write strobe and data drivers assert on their normal cadence,
overriding DMA's read-direction pad state — so when DMA's write arm
(POWU → WYJA) latches the data pad into OAM, the pad carries **the CPU's
data**. Three things follow:

- **OAM[`dma_dest`]** receives the CPU's byte.
- the **off-chip cell at DMA's source address** takes a spurious write of
  the same byte (the strobe asserts with DMA's address on the pads);
- the **CPU's intended target** never sees the write.

Anchored: the pad sequence shows the CPU's 0x00 winning the data bus
mid-settle, the OAM commit strobe latching it, and OAM[$9E] = 0x00 after
the transfer — matching the hardware-verified `out0` (dmg-sim measurement,
gambatte `oamdma_src8000_busywrite8000` ROM).

Both conflicts resolve deterministically — CPU and DMA share the master
clock, so the CPU's data-phase and DMA's write-arm sub-window are co-aligned
the same way every M-cycle. There is no sub-dot race here.

**HRAM exemption.** The HRAM chip-select chain derives from the CPU bus
phase and the $FFxx decode; `dma_run` is not an input. CPU access to HRAM
during DMA is uncontended (120 successful HRAM reads inside one DMA window
in the measurement above) — the basis of every HRAM-stub DMA routine.

**Stack accesses** are ordinary bus cycles at SP; the rules above apply
unchanged. Stack in HRAM is unaffected; stack in WRAM during a same-bus DMA
redirects identically.

## DMA sources $FE and $FF

For source pages $FE/$FF the address MUX behaves as for any other page:
`dma_a` goes onto the cart address pads. The difference is the cart
chip-select MUX (TYHO, sel = `dma_addr_ext`): during DMA it takes DMA's
a15 directly, **bypassing** the CPU-side decode (TOZA) that would
normally de-assert /CS for $FE–$FF. So the cart bus is driven with
$FE00–$FE9F / $FF00–$FF9F addresses — even though on the CPU side those
map to on-die OAM/IO/HRAM.

The package bus is now driven with an OAM/IO-range address; one device on it answers:

| Candidate | Responds? | Reason |
|-----------|:---:|--------|
| On-die OAM SRAM | No | OAM uses its own internal address bus, not the external pads; its output enable stays 0 throughout (measured) |
| On-die HRAM | No | The HRAM enable chain rides the CPU-internal address bus, not the external pads |
| **On-package WRAM die** | **Yes** | Selects on `cs_n_pad` + a14=1 alone (see below) |
| Cart devices | Moot | WRAM has already driven the bus; MBC RAM /CS requires a14=0 anyway |

The WRAM die in the DMG-CPU-B package selects on `cs_n_pad` and a14=1
**alone** — no a13 qualifier and no $FE/$FF exclusion, because those
exclusions live in the CPU die's internal decode, which TYHO has
bypassed. It drives the data pads with `work_ram[a[12:0]]`, so the
address wraps: $FExx → $1Exx, $FFxx → $1Fxx.

The behaviour is the documented echo region extended one page: **DMA from
$FE/$FF fetches `work_ram[(src − $E000) & $1FFF]`** — hardware-verified as
`work_ram[$1E00+i]` / `work_ram[$1F00+i]` per byte index.

A pre-filled-WRAM ROM family confirms 8-of-8 byte matches at the fold
offsets (dmg-sim measurement, gambatte `oamdma_src{FE,FF}00_busypop*`
ROMs), and both bank commit strobes fire exactly 80 times per transfer —
every byte index commits, with no per-index suppression. $FE and $FF
sources differ only in offset — `work_ram[$1Exx]` vs `work_ram[$1Fxx]` —
through the identical mechanism; there is no per-page behavioural
difference between them.

```admonish note "For implementors"
Treat $FE/$FF sources as the echo region extended one page — the single
mechanism covers the read-conflict and pop-conflict ROM families alike.
```
