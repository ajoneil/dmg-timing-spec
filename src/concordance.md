# Appendix A: Signal Concordance

Every netlist gate named in this book — 951 signals — with its role, cell
type, and the chapter that owns its primary description. Gate names come
from the DMG-CPU B netlist via
[msinger/dmg-schematics](https://github.com/msinger/dmg-schematics); each
is independently checkable against the schematics and the dmg-sim Verilog.
Roles are taken from the owning chapter's reference blocks; gates without
a role row appear in prose or chain descriptions, where the linked chapter
carries the context.

This index is generated from the book's own text and validated against the
netlist sources, so every entry corresponds to a real cell and a real
mention.

| Gate | Role | Type | Primary chapter | Also appears in |
|------|------|------|-----------------|-----------------|
| ABAF |  |  | [Mode Transitions](mode-transitions.md) |  |
| ABAK | Per-slot-reset shared gating term | or2 | [Mode 2: OAM Scan](oam-scan.md) | [Mode Transitions](mode-transitions.md) |
| ABEZ |  |  | [Mode Transitions](mode-transitions.md) |  |
| ABON | FAMU enable | not_x2 chain | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| ACYL |  |  | [OAM DMA](dma.md) |  |
| ADAR |  |  | [Register Writes](registers/writes.md) |  |
| ADEH | ATAL complement | not_x1 | [The Clock Tree](clock-tree.md) | [Register Writes](registers/writes.md) |
| ADYK | Phase generator ring stage 4 | dff | [The Clock Tree](clock-tree.md) | [Register Writes](registers/writes.md) |
| AFAS | Ring decode driving CUPA (inputs ADAR, ATYP) | nor2 | [Register Writes](registers/writes.md) |  |
| AFUR | Phase generator ring stage 1 | dff | [The Clock Tree](clock-tree.md) | [Register Writes](registers/writes.md) |
| AGET | Length counter load enable (upper bits) | not_x1 | [Length Counters and Power Cycling](apu-length-power.md) |  |
| AJEP |  |  | [OAM DMA](dma.md) |  |
| AJER | CH1 prescaler /2 stage 1 | dffr | [APU Clock Tree and Frame Sequencer](apu-clocks.md) | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md), [Sweep and Envelope](apu-sweep-envelope.md) |
| AJON | Mode 3 + non-DMA | and2 | [Rendering Mode Control](mode-control.md) | [OAM and VRAM Access](oam-vram-access.md), [OAM DMA](dma.md), [CPU-Visible Mode Boundaries](cpu-visible-boundaries.md) |
| AJUJ | OAM-access permit | nor3 | [OAM and VRAM Access](oam-vram-access.md) | [OAM DMA](dma.md), [CPU-Visible Mode Boundaries](cpu-visible-boundaries.md) |
| AKYD | CH2 length counter bit 5 (MSB) | tffnl | [Length Counters and Power Cycling](apu-length-power.md) |  |
| ALEF | Phase generator ring stage 2 | dff | [The Clock Tree](clock-tree.md) | [Register Writes](registers/writes.md) |
| ALES |  |  | [Mode Transitions](mode-transitions.md) |  |
| ALET | 4 MHz PPU main clock | not_x2 | [The Clock Tree](clock-tree.md) | [Rendering Mode Control](mode-control.md), [Register Writes](registers/writes.md), [Register Reads](registers/reads.md) |
| AMAB | OAM CPU-write address gate | and2 | [OAM and VRAM Access](oam-vram-access.md) |  |
| AMUV | LCDC.3 → ~ma10 tri-state driver | not_if0 | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| AMYG |  |  | [Mode 2: OAM Scan](oam-scan.md) |  |
| ANEL | Scan counter reset chain stage 2 | dffr | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md), [Mode Transitions](mode-transitions.md) |
| ANOM | Scan counter reset | nor2 | [Mode 2: OAM Scan](oam-scan.md) | [Mode Transitions](mode-transitions.md) |
| ANOS | Crystal feedback NAND | nand2 | [The Clock Tree](clock-tree.md) |  |
| ANYP | SCY read enable (AND2 with `ppu_rd`) | and2 | [Register Reads](registers/reads.md) |  |
| APAR |  |  | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md) |
| APOV | CUPA driver stage 4 | not | [Register Writes](registers/writes.md) |  |
| APUG |  |  | [The IF Register](if-register.md) |  |
| APUK | Phase generator ring stage 3 | dff | [The Clock Tree](clock-tree.md) | [Register Writes](registers/writes.md) |
| APUV | CH1 prescaler clock buffer | not_x1 | [APU Clock Tree and Frame Sequencer](apu-clocks.md) |  |
| AREV | CUPA driver stage 5 | nand2 | [Register Writes](registers/writes.md) |  |
| AROR | Sprite-trigger gate (LCDC.1-gated rendering enable) | and2 | [LCDC Structure](registers/lcdc.md) | [Mode 2: OAM Scan](oam-scan.md), [Mode 3: The BG Pipeline](bg-pipeline.md), [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| AROV |  |  | [Register Writes](registers/writes.md) |  |
| ARUR | SCX write enable | and2 | [Register Writes](registers/writes.md) |  |
| ASAM | OAM CPU-address disable | or3 | [Rendering Mode Control](mode-control.md) | [OAM DMA](dma.md) |
| ASEN | BESU reset driver | or2 | [Rendering Mode Control](mode-control.md) | [Mode 2: OAM Scan](oam-scan.md), [Mode Transitions](mode-transitions.md) |
| ATAG | APU 4 MHz polarity restore | not_x1 | [APU Clock Tree and Frame Sequencer](apu-clocks.md) |  |
| ATAL | 4 MHz buffered enable | not_x2 | [The Clock Tree](clock-tree.md) | [Register Writes](registers/writes.md) |
| ATAR | VID_RST active-high | not | [Rendering Mode Control](mode-control.md) | [Mode 2: OAM Scan](oam-scan.md), [Mode Transitions](mode-transitions.md) |
| ATEJ |  |  | [Rendering Mode Control](mode-control.md) | [Mode 2: OAM Scan](oam-scan.md), [Mode 3: The BG Pipeline](bg-pipeline.md), [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| ATEP | CH2 prescaler /2 stage 1 | dffr | [APU Clock Tree and Frame Sequencer](apu-clocks.md) | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |
| ATYP |  |  | [Register Writes](registers/writes.md) |  |
| AVAP | Mode 2→3 trigger | not_x2 | [Mode 2: OAM Scan](oam-scan.md) | [The Clock Tree](clock-tree.md), [Rendering Mode Control](mode-control.md), [Register Writes](registers/writes.md), [Register Reads](registers/reads.md) |
| AVER | = NAND2(mode2, XYSO) — scan-phase term in BYCU | nand2 | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md), [OAM and VRAM Access](oam-vram-access.md) |
| AVET | Crystal feedback NAND | nand2 | [The Clock Tree](clock-tree.md) |  |
| AVOG | SCX read enable (AND2 with `ppu_rd`) | and2 | [Register Reads](registers/reads.md) |  |
| AWOH |  |  | [Mode Transitions](mode-transitions.md) |  |
| AXAD |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| AZEG | CH2 prescaler clock buffer | not_x1 | [APU Clock Tree and Frame Sequencer](apu-clocks.md) |  |
| AZEM | AROR second input (rendering-active AND mid-scanline-control) | and2 | [Rendering Mode Control](mode-control.md) | [Mode 2: OAM Scan](oam-scan.md) |
| AZET | Wave-RAM precharge-window term (with AZUS) | dffr | [CH3: Wave Channel](apu-ch3.md) |  |
| AZOF | PPU clock buffer | not_x6 | [The Clock Tree](clock-tree.md) | [APU Clock Tree and Frame Sequencer](apu-clocks.md) |
| AZUS | wave_data_latch synchroniser stage 3 | dffr | [CH3: Wave Channel](apu-ch3.md) |  |
| AZYB | Slot counter reset driver | not_x1 | [Mode 2: OAM Scan](oam-scan.md) | [Mode Transitions](mode-transitions.md) |
| BAFU | apu_wr buffer stage 1 | not_x1 | [APU Clock Tree and Frame Sequencer](apu-clocks.md) |  |
| BAFY | AMUV enable (BG tilemap stage) | not_x2 | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| BAGY | BYBA/DOBA reset driver | not_x1 | [Mode 2: OAM Scan](oam-scan.md) |  |
| BALU | OAM-parse reset (active-high) | not_x1 | [Mode 2: OAM Scan](oam-scan.md) | [Mode Transitions](mode-transitions.md) |
| BANO | wave_data_latch synchroniser stage 2 | dffr | [CH3: Wave Channel](apu-ch3.md) |  |
| BAPY |  |  | [Register Writes](registers/writes.md) |  |
| BATU |  |  | [The IF Register](if-register.md) |  |
| BAVE | Sweep pace = 0 detector | and3 | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| BAXO |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| BEBA | = NOT(AVOG) — SCX read drivers' `ena_n` | not_x1 | [Register Reads](registers/reads.md) |  |
| BEBU | Scan-complete aggregate | or3 | [The Clock Tree](clock-tree.md) | [Rendering Mode Control](mode-control.md), [Mode 2: OAM Scan](oam-scan.md), [Mode Transitions](mode-transitions.md) |
| BEDY | SCY write enable | and2 | [Register Writes](registers/writes.md) |  |
| BEGO | Slot counter bit 2 | dffr | [Mode 2: OAM Scan](oam-scan.md) |  |
| BELU | `data_phase` gate (= NOR2(ATYP, `clk_ena_n`)) | nor2 | [Register Reads](registers/reads.md) |  |
| BERU |  |  | [Register Writes](registers/writes.md) |  |
| BESE | Slot counter bits 0–3 | dffr (toggle) | [Mode 2: OAM Scan](oam-scan.md) |  |
| BESU | Scan-active latch (Q=1 during Mode 2) | nor_latch | [Rendering Mode Control](mode-control.md) | [Register Writes](registers/writes.md), [Register Reads](registers/reads.md), [OAM and VRAM Access](oam-vram-access.md) |
| BETE |  |  | [OAM DMA](dma.md) |  |
| BEVA | `data_phase` rebuffer | not_x6 | [Register Reads](registers/reads.md) |  |
| BEXA | Sweep-fire latch | dffr | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| BODE |  |  | [OAM DMA](dma.md) |  |
| BOGA | PPU timing divider (held 0 under VID_RST) |  | [Register Writes](registers/writes.md) | [Post-Boot State](post-boot.md) |
| BOGE |  |  | [OAM and VRAM Access](oam-vram-access.md) | [OAM DMA](dma.md), [Mode 2: OAM Scan](oam-scan.md) |
| BONE | Sweep overflow channel-disable term (= NOT(`ch1_sum_ovfl_n`)) | not_x1 | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| BUKO | Length counter bit-3→4 ripple inverter | not_x1 | [Length Counters and Power Cycling](apu-length-power.md) |  |
| BURE | Frame-sequencer 512 Hz clock | not_x2 | [APU Clock Tree and Frame Sequencer](apu-clocks.md) |  |
| BURY | BEXA async-reset driver (pace 0) | nor2 | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| BUSA | wave_data_latch synchroniser stage 1 | dffr | [CH3: Wave Channel](apu-ch3.md) |  |
| BUVA | CH2 length counter bit 4 | tffnl | [Length Counters and Power Cycling](apu-length-power.md) |  |
| BUWY | = NOT(ANYP) — SCY read drivers' `ena_n` | not_x1 | [Register Reads](registers/reads.md) |  |
| BUZA |  |  | [Rendering Mode Control](mode-control.md) |  |
| BYBA | Scan-done flag | dffr | [The Clock Tree](clock-tree.md) | [Mode 2: OAM Scan](oam-scan.md), [Mode 3: The BG Pipeline](bg-pipeline.md), [Mode Transitions](mode-transitions.md) |
| BYCU | = NAND3(CUFE, XUJY, AVER) — COTA driver | nand3 | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md), [OAM and VRAM Access](oam-vram-access.md) |
| BYHA |  |  | [Mode Transitions](mode-transitions.md) |  |
| BYJO | AZEM first input (mid-scanline control) | not_x1 | [Mode 2: OAM Scan](oam-scan.md) |  |
| BYLU | Frame sequencer /2 → 128 Hz (sweep tap) | dffr | [APU Clock Tree and Frame Sequencer](apu-clocks.md) | [Post-Boot State](post-boot.md) |
| BYMO | Length counter load enable (lower bits) | not_x1 | [Length Counters and Power Cycling](apu-length-power.md) |  |
| BYRY | `data_phase` inverter (of BELU) | not_x2 | [Register Reads](registers/reads.md) |  |
| CAGY | CH2 duty counter bit 1 | dffr_cc | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| CAKE |  |  | [Mode 2: OAM Scan](oam-scan.md) |  |
| CALA | CH1 overflow detector (NOT stage) | not_x1 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| CALO | CH1 prescaler /2 stage 2 | dffr | [APU Clock Tree and Frame Sequencer](apu-clocks.md) | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |
| CALY | int_serial driver (toggle DFF on serial transfer clock) | dffr | [The IF Register](if-register.md) |  |
| CAMA | CH2 divider toggle clock (NOR stage) | nor2 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| CAME | CH2 length counter bit 3 | tffnl | [Length Counters and Power Cycling](apu-length-power.md) |  |
| CANO | CH2 duty counter bit 0 (LSB) | dffr | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| CAPE |  |  | [Mode 2: OAM Scan](oam-scan.md) |  |
| CAPY | CH1 length-clock gate | nor3 | [Length Counters and Power Cycling](apu-length-power.md) |  |
| CARE | Global sprite-save enable | and3 | [Mode 2: OAM Scan](oam-scan.md) | [Mode Transitions](mode-transitions.md), [Scanline and Frame Timing](scanline-frame-timing.md) |
| CARU | Frame sequencer /2 → 256 Hz (length tap) | dffr | [APU Clock Tree and Frame Sequencer](apu-clocks.md) | [Length Counters and Power Cycling](apu-length-power.md), [Post-Boot State](post-boot.md) |
| CASY | SCX bit-7 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| CATA | SCX bit-5 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| CATU | Scan-boundary trigger | dffr | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md), [Mode Transitions](mode-transitions.md), [CPU-Visible Mode Boundaries](cpu-visible-boundaries.md) |
| CAVA | CH1 duty-pattern select | nor2 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| CAXO | CH1 duty-pattern select | nor2 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| CAXU |  |  | [Mode 2: OAM Scan](oam-scan.md) |  |
| CAXY | Sweep counter bit 2 | tffnl | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| CEDE | inverted `dma_addr_ext` | not_x2 | [OAM DMA](dma.md) |  |
| CEDU | SCX bit-4 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| CEDY |  |  | [Mode 2: OAM Scan](oam-scan.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| CEHA |  |  | [Mode 2: OAM Scan](oam-scan.md) |  |
| CEMO | CH2 prescaler /2 stage 2 | dffr | [APU Clock Tree and Frame Sequencer](apu-clocks.md) | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |
| CENO | XUPY-phased scan-active copy | dffr | [The Clock Tree](clock-tree.md) | [Rendering Mode Control](mode-control.md), [Mode 2: OAM Scan](oam-scan.md), [Races](races.md) |
| CERA | CH2 length counter bit 1 | tffnl | [Length Counters and Power Cycling](apu-length-power.md) |  |
| CERY | CH3 prescaler /2 stage | dffr | [APU Clock Tree and Frame Sequencer](apu-clocks.md) | [CH3: Wave Channel](apu-ch3.md) |
| CEVU | CH1 duty-pattern select | nor2 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| CODO | CH1 duty counter-state decode | not_x1 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| COMY | CH1 overflow capture (/2 toggle) | dffr | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| CONU | CH2 length counter bit 2 | tffnl | [Length Counters and Power Cycling](apu-length-power.md) |  |
| COPU | CH1 divider bit 10 (MSB) | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| COSO | CH1 duty-pattern select | nor2 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| COTA | Stage-2 capture enable; complement WOVU drives the OAM bitline precharge | not_x1 | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md), [OAM and VRAM Access](oam-vram-access.md) |
| COWE | CH1 PWM gated by channel-running | and2 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| COZE | Sweep counter-at-max detector | and3 | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| CUFE | = OAI21(FEXX, `dma_run`, `dma_phi_n`) — $FExx/DMA bus-cycle term in BYCU | oai21 | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md), [OAM and VRAM Access](oam-vram-access.md) |
| CUGA | SCX bit-2 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| CULE | CH2 duty counter clock | not_x1 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| CUPA | Shared PPU register write strobe | combinational (derived from phase gen) | [Register Writes](registers/writes.md) | [Register Reads](registers/reads.md), [OAM and VRAM Access](oam-vram-access.md), [Mode 2: OAM Scan](oam-scan.md) |
| CUPO | Sweep counter bit 0 | tffnl | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| CUSA | SCY bit-4 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| CUXY | Slot counter bit 1 | dffr | [Mode 2: OAM Scan](oam-scan.md) |  |
| CYBO | CH3 prescaler clock buffer | not_x1 | [APU Clock Tree and Frame Sequencer](apu-clocks.md) |  |
| CYKE |  |  | [Mode 2: OAM Scan](oam-scan.md) |  |
| CYMU | Sweep counter load enable (NOT stage) | not_x1 | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| CYPU | Sweep counter bit 1 | tffnl | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| CYRA |  |  | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md) |
| CYRE | CH2 length-clock gate (DEME) input | dffr | [Length Counters and Power Cycling](apu-length-power.md) |  |
| CYSE | CH2 PWM gated by channel-running | and2 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| CYVO | CH2 divider bit 3 | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| DAFA | Sweep counter load enable (NOR stage) | nor2 | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| DAJO | CH1 duty counter clock | not_x1 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| DAPE | CH1 duty counter bit 2 (MSB) | dffr_cc | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| DARE | CH2 duty counter-state decode | not_x1 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| DASA |  |  | [Mode 2: OAM Scan](oam-scan.md) |  |
| DEGO | Per-slot X match, FEFY group (active-low) | nand3 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| DEME | CH2 length-clock gate | nor3 | [Length Counters and Power Cycling](apu-length-power.md) |  |
| DEPO | OAM attribute bit 7 latch (BG-over-OBJ priority) | dlatch_ee | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| DERO | CH3 wave-position counter clock | not_x1 | [CH3: Wave Channel](apu-ch3.md) |  |
| DERU | CH1 divider ripple inverter (bits 8–10) | not_x1 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| DOBA | ALET-clocked scan-done-prev | dffr | [The Clock Tree](clock-tree.md) | [Mode 2: OAM Scan](oam-scan.md), [Mode Transitions](mode-transitions.md), [Scanline and Frame Timing](scanline-frame-timing.md) |
| DOCA | CH2 divider toggle clock (NOT stage) | not_x1 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| DODA | CH3 length-clock gate | nor3 | [Length Counters and Power Cycling](apu-length-power.md) |  |
| DOJU | CH2 duty-pattern select | nor2 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| DOKA | Overflow-capture self-clear term (= AND2(COMY, `ch1_1mhz`)) | and2 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| DOME | CH2 PWM latch | dffr | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| DOMO | CH2 duty-pattern select | nor2 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| DONE | CH2 divider bit 0 | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| DOVE | CH2 duty-pattern select | nor2 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| DOXE | SCX bit-6 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| DUGA |  |  | [OAM DMA](dma.md) |  |
| DUJU | CH2 divider load enable | nor2 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| DUWO | CH1 PWM latch | dffr | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| DYBA | = NOT(BYVA) — per-slot line-reset term | not_x1 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| DYBE | Slot counter bit 3 (MSB) | dffr | [Mode 2: OAM Scan](oam-scan.md) |  |
| DYDU | Per-slot X match, FEFY group (active-low) | nand3 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| DYKA | Per-slot X match, FOVE group (active-low) | nand3 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| DYKY | CUPA driver stage 1 | not | [Register Writes](registers/writes.md) |  |
| DYMU | CH2 duty counter-state decode | and2 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| DYNU | CH2 divider bit 1 | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| DYRO | Gated 256 Hz length tick (= NOT(DEME)) | not_x1 | [Length Counters and Power Cycling](apu-length-power.md) |  |
| DYRU | COMY async-reset driver (= NOR(`apu_reset`, `ch1_restart`, DOKA)) | nor3 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| DYTA | CH2 duty-pattern select | nor2 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| DYVE | CH2 duty counter bit 2 (MSB) | dffr_cc | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| DYWE | Slot-0 reset driver = OR2(DYBA, EBOJ) | or2 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| EBEB | Slot-3 write-enable inverter | not_x1 | [Mode 2: OAM Scan](oam-scan.md) | [Races](races.md) |
| EBOJ | Slot-0 fetch-done flag (d = GUVA, clk = WUTY) | dffr | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) | [Mode 2: OAM Scan](oam-scan.md) |
| EBOS |  |  | [Mode 2: OAM Scan](oam-scan.md) |  |
| ECED |  |  | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md), [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| EDOS | SCX bit-0 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| EFAL | CH3 wave-position bit 4 (MSB) | dffr | [CH3: Wave Channel](apu-ch3.md) |  |
| EFAR | CH3 wave-position bit 0 (`wave_nibble_sel`) | dffr | [CH3: Wave Channel](apu-ch3.md) |  |
| EFUZ | CH3 wave-position bit 2 | dffr | [CH3: Wave Channel](apu-ch3.md) |  |
| EFYL | Per-slot X match, FOVE group (active-low) | nand3 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| EGOG | CH2 duty counter-state decode | and2 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| EGOM | Per-slot X match, FOVE group (active-low) | nand3 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| EKOB | SCX bit-1 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| EKOV | CH1 divider bit 7 | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| ELYN | Scan counter bit 3 | dffr | [Mode 2: OAM Scan](oam-scan.md) |  |
| EMUS | CH1 divider bit 8 | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| EMYR | Volume saturation detector (vol 0 + down) | nor5 | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| ENEF | Y-compare carry-chain bit 1 | full_add | [Mode 2: OAM Scan](oam-scan.md) |  |
| ENEK | CH1 duty counter-state decode | and2 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| EPYK | CH1 divider load enable | nor2 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| EROS | CH1 duty counter bit 1 | dffr_cc | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| ERUC | Y-compare carry-chain bit 0 | full_add | [Mode 2: OAM Scan](oam-scan.md) |  |
| ERUS | CH3 wave-position bit 1 | dffr | [CH3: Wave Channel](apu-ch3.md) |  |
| ERYC | CH2 length counter bit 0 | tffnl | [Length Counters and Power Cycling](apu-length-power.md) |  |
| ESUT | CH1 duty counter bit 0 (LSB) | dffr | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| ETAN | Wave-position counter reset (active-low) | nor2 | [CH3: Wave Channel](apu-ch3.md) |  |
| EVAK | CH1 divider bit 9 | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| EXEL | CH3 wave-position bit 3 | dffr | [CH3: Wave Channel](apu-ch3.md) |  |
| EZOF | CH2 divider bit 2 | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| EZOZ | CH1 duty counter-state decode | and2 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| FABO | `ch3_restart` sample clock (= NOT(`ch3_2mhz`)) | not_x1 | [CH3: Wave Channel](apu-ch3.md) |  |
| FAHA | Scan counter bit 4 | dffr | [Mode 2: OAM Scan](oam-scan.md) |  |
| FAMU | `~ma4` tri-state driver | not_if0 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| FECO | Y-compare carry-chain bit 2 | full_add | [Mode 2: OAM Scan](oam-scan.md) |  |
| FEFY | Sprites 0–4 X-match aggregate | nand5 | [Mode 2: OAM Scan](oam-scan.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| FEMO |  |  | [Mode 2: OAM Scan](oam-scan.md) |  |
| FENA | Volume counter bit 3 | tffnl | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| FENO | Volume counter bit 0 | tffnl | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| FEPO | Sprite X priority aggregate | or2 | [Rendering Mode Control](mode-control.md) | [Mode 2: OAM Scan](oam-scan.md), [Mode 3: The BG Pipeline](bg-pipeline.md), [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| FEPU |  |  | [Mode 2: OAM Scan](oam-scan.md) |  |
| FETE | Volume counter bit 1 | tffnl | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| FETO | Scan-done decode | and4 | [Mode 2: OAM Scan](oam-scan.md) | [Mode Transitions](mode-transitions.md) |
| FETY | CH3 counter-wrap interlock (/2 toggle) | dffr | [CH3: Wave Channel](apu-ch3.md) |  |
| FEVA | CH1 divider bit 6 | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| FEXX | $FExx address-range decode (= NOT(ROPE)) | not_x2 | [OAM and VRAM Access](oam-vram-access.md) |  |
| FFFF |  |  | [Interrupt Dispatch](interrupt-dispatch.md) |  |
| FOBA | NR34 trigger M-cycle synchroniser | dffr | [CH3: Wave Channel](apu-ch3.md) |  |
| FOCO | Slot-3 selector | nand4 | [Mode 2: OAM Scan](oam-scan.md) |  |
| FOFA |  |  | [Mode 2: OAM Scan](oam-scan.md) |  |
| FOMY | Volume counter bit 2 | tffnl | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| FONE |  |  | [Mode 2: OAM Scan](oam-scan.md) |  |
| FONO |  |  | [Mode 2: OAM Scan](oam-scan.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| FONY | Scan counter bit 5 (MSB) | dffr | [Mode 2: OAM Scan](oam-scan.md) |  |
| FOVE | Sprites 5–9 X-match aggregate | nand5 | [Mode 2: OAM Scan](oam-scan.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| FOZU | CH3 active-bit nor_latch | nor_latch | [CH3: Wave Channel](apu-ch3.md) |  |
| FUFO | XYMO complement | not_x1 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| FUKY |  |  | [Mode 2: OAM Scan](oam-scan.md) |  |
| FULO | CH1 divider toggle clock (NOR stage) | nor2 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| FUVE |  |  | [Mode 2: OAM Scan](oam-scan.md) |  |
| FUWA | Y-compare carry-chain bit 5 | full_add | [Mode 2: OAM Scan](oam-scan.md) |  |
| FUXO | CH2 divider bit 4 | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| FUXU | Slot-0 X save strobe (store-counter = 0 decode) | not_x1 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| FYCU |  |  | [Mode 2: OAM Scan](oam-scan.md) |  |
| FYNE | `horu_512hz` buffer chain stage | not_x3 | [APU Clock Tree and Frame Sequencer](apu-clocks.md) |  |
| FYRE | Volume saturation detector (vol 15 + up) | not_x1 | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| GACE | NOT(GOPU) — Y-compare sum bit 4, complemented | not_x1 | [Mode 2: OAM Scan](oam-scan.md) |  |
| GALE | `horu_512hz` buffer chain stage | mux | [APU Clock Tree and Frame Sequencer](apu-clocks.md) |  |
| GANE | CH2 divider bit 7 | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| GANO | CH2 divider bit 5 | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| GARA | `ch3_restart` synchroniser DFF | dffr | [CH3: Wave Channel](apu-ch3.md) |  |
| GAVA | Scan counter clock | or2 | [Mode 2: OAM Scan](oam-scan.md) |  |
| GAVU | NR34 trigger-bit capture | drlatch_ee | [CH3: Wave Channel](apu-ch3.md) |  |
| GAXE | CH1 divider bit 0 | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| GEJY | OBJ_SIZE mux | ao22 | [LCDC Structure](registers/lcdc.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| GEKU | CH1 divider toggle clock (NOT stage) | not_x1 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| GEPY | CH4 length-clock gate | nor3 | [Length Counters and Power Cycling](apu-length-power.md) |  |
| GESE | `sprite_y_match` driver | not_x1 | [Mode 2: OAM Scan](oam-scan.md) |  |
| GEWY | NOT(WUHU) — Y-compare sum bit 7, complemented | not_x1 | [Mode 2: OAM Scan](oam-scan.md) |  |
| GEXY | `horu_512hz` buffer chain stage | not_x1 | [APU Clock Tree and Frame Sequencer](apu-clocks.md) |  |
| GOBA | SCY bit-1 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| GOCA | CH2 divider bit 6 | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| GODO | SCY bit-3 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| GOFY | NR34 trigger nor_latch (q_n is held) | nor_latch | [CH3: Wave Channel](apu-ch3.md) |  |
| GOJU | Y-compare carry-chain bit 6 | full_add | [Mode 2: OAM Scan](oam-scan.md) |  |
| GOMO |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| GONU | SCY bit-2 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| GOPU | Y-compare carry-chain bit 4 | full_add | [Mode 2: OAM Scan](oam-scan.md) |  |
| GOSO | Scan counter bit 2 | dffr | [Mode 2: OAM Scan](oam-scan.md) |  |
| GOVU | LCDC.2 mask | or2 | [LCDC Structure](registers/lcdc.md) | [Mode 2: OAM Scan](oam-scan.md) |
| GUFY | Volume saturation detector aggregate | or2 | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| GUNE | SCY bit-6 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| GUSU |  |  | [Mode 2: OAM Scan](oam-scan.md) |  |
| GUVA | = NOT(YDUG) — slot-0 match select | nor2 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| GUVU | NOT(FUWA) — Y-compare sum bit 5, complemented | not_x1 | [Mode 2: OAM Scan](oam-scan.md) |  |
| GYDA | NOT(GOJU) — Y-compare sum bit 6, complemented | not_x1 | [Mode 2: OAM Scan](oam-scan.md) |  |
| GYKY | Carry-chain bit 3 | full_add | [Mode 2: OAM Scan](oam-scan.md) |  |
| GYTA | CH3 trigger self-clear delay | dffr | [CH3: Wave Channel](apu-ch3.md) |  |
| GYZA | SCY bit-7 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| GYZO | SCY bit-5 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| HAFE | JOPA reset driver (pace 0) | nor4 | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| HEFO | CH3 divider toggle clock (NOR stage) | nor2 | [CH3: Wave Channel](apu-ch3.md) |  |
| HEPO | Saturation capture | dffr | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| HEPU | CH2 divider bit 9 | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| HERA | CH3 divider load enable (active-low) | nor2 | [CH3: Wave Channel](apu-ch3.md) |  |
| HERO | CH2 divider bit 10 (MSB) | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| HEVY | CH2 divider bit 8 | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| HOFO | Volume bit-0 toggle clock | or3 | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| HORU | `horu_512hz` buffer chain stage | not_x3 | [APU Clock Tree and Frame Sequencer](apu-clocks.md) |  |
| HUNO | CH3 overflow capture (/2 toggle) | dffr | [CH3: Wave Channel](apu-ch3.md) |  |
| HYFE | CH1 divider bit 1 | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| HYFO | CH3 overflow detector (NOT stage) | not_x1 | [CH3: Wave Channel](apu-ch3.md) |  |
| HYKE | CH1 divider bit 5 | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| HYLY | Envelope counter load enable (NOR stage) | nor2 | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| JAKE | Envelope counter load enable (NOT stage) | not_x1 | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| JAPU | CH3 divider bit 10 (MSB) | tffnl | [CH3: Wave Channel](apu-ch3.md) |  |
| JEMA | CH1 divider bit 4 | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| JEME | Envelope-stop latch | nor_latch | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| JERA | CH3 divider load-enable fan-out | not_x1 | [CH3: Wave Channel](apu-ch3.md) |  |
| JEVY | Envelope counter bit 2 | tffnl | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| JONA | Envelope counter bit 1 | tffnl | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| JOPA | Envelope-fire latch | dffr | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| JORE | Envelope counter bit 0 | tffnl | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| JUPU | Envelope pace = 0 detector | nor3 | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| JUTY | CH3 divider toggle clock (NOT stage) | not_x1 | [CH3: Wave Channel](apu-ch3.md) |  |
| JYNA | Frame sequencer /2 → 64 Hz (envelope tap) | dffr | [APU Clock Tree and Frame Sequencer](apu-clocks.md) | [Post-Boot State](post-boot.md) |
| JYTY | CH1 divider bit 2 | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| KAFO | CH3 divider bit 6 | tffnl | [CH3: Wave Channel](apu-ch3.md) |  |
| KAHE | CPL source (AO22 LCDC.7 mux) | ao22 | [LCDC Structure](registers/lcdc.md) | [LCD Output](lcd-output.md) |
| KASA | CPL PPU-clock source (LCD-on arm) | not | [LCD Output](lcd-output.md) |  |
| KASO | CH3 divider load enable (bits 4–7) | not_x1 | [CH3: Wave Channel](apu-ch3.md) |  |
| KEBO | FR PPU-clock source (LCD-on arm) | not | [LCD Output](lcd-output.md) |  |
| KEDY | Mux selector complement (LCDC.7=0 arm) | not | [LCD Output](lcd-output.md) |  |
| KEJU | CH3 divider bit 8 | tffnl | [CH3: Wave Channel](apu-ch3.md) |  |
| KEMU | CH3 divider bit 3 | tffnl | [CH3: Wave Channel](apu-ch3.md) |  |
| KENO | CH3 divider bit 7 | tffnl | [CH3: Wave Channel](apu-ch3.md) |  |
| KEPA | CH3 divider bit 5 | tffnl | [CH3: Wave Channel](apu-ch3.md) |  |
| KESE | CH3 divider ripple inverter (bits 8–10) | not_x1 | [CH3: Wave Channel](apu-ch3.md) |  |
| KEZA | CH3 divider bit 9 | tffnl | [CH3: Wave Channel](apu-ch3.md) |  |
| KOFO | FR pad driver | not | [LCD Output](lcd-output.md) |  |
| KUNU | CH3 divider bit 2 | tffnl | [CH3: Wave Channel](apu-ch3.md) |  |
| KUPA | FR source (AO22 LCDC.7 mux) | ao22 | [LCDC Structure](registers/lcdc.md) | [LCD Output](lcd-output.md) |
| KUPE | CH3 divider bit 1 | tffnl | [CH3: Wave Channel](apu-ch3.md) |  |
| KUTU | CH3 divider bit 0 | tffnl | [CH3: Wave Channel](apu-ch3.md) |  |
| KYGU | CH3 divider bit 4 | tffnl | [CH3: Wave Channel](apu-ch3.md) |  |
| KYKO | CH3 divider load enable (bits 0–3) | not_x1 | [CH3: Wave Channel](apu-ch3.md) |  |
| KYMO | CPL pad driver | not | [LCD Output](lcd-output.md) |  |
| KYNA | CH1 divider bit 3 | tffnl | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| KYPE | CH1 divider ripple inverter (bits 4–7) | not_x1 | [CH1/CH2: Pulse Channels](apu-ch1-ch2.md) |  |
| KYRU | CH3 divider ripple inverter (bits 4–7) | not_x1 | [CH3: Wave Channel](apu-ch3.md) |  |
| KYVO | Envelope counter-at-max detector | and3 | [Sweep and Envelope](apu-sweep-envelope.md) |  |
| LACE | BGP bit-4 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| LAFO | LY bit 7 | dffr | [Line Counters](line-counters.md) | [Register Reads](registers/reads.md) |
| LAJU | OBP1 bit-0 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| LALU | STAT interrupt latch (IF bit 1) | dffsr | [STAT Interrupts](stat-interrupts.md) | [Interrupt Dispatch](interrupt-dispatch.md), [The IF Register](if-register.md) |
| LAMA | LY reset | nor2 | [Line Counters](line-counters.md) | [Register Reads](registers/reads.md), [Mode Transitions](mode-transitions.md), [CPU-Visible Mode Boundaries](cpu-visible-boundaries.md) |
| LAPE |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| LARA |  |  | [OAM DMA](dma.md) |  |
| LARY | BGP bit-7 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| LAVY |  |  | [OAM DMA](dma.md) |  |
| LAXU | Fetch counter bit 0 | dffr | [The Clock Tree](clock-tree.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [Scanline and Frame Timing](scanline-frame-timing.md), [Races](races.md) |
| LEBA | OBP1 bit-6 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| LEBO | Fetch-counter gated clock | nand2 | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Scanline and Frame Timing](scanline-frame-timing.md) |
| LEFE |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| LEGU |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| LELU | OBP1 bit-7 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| LEMA | LY bit 5 | dffr | [Line Counters](line-counters.md) |  |
| LEPA | OBP1 bit-1 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| LESU |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| LESY |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| LEXA | LY bit 2 | dffr | [Line Counters](line-counters.md) |  |
| LOBE | BGP bit-3 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| LOBY | POKY reset driver | not_x1 | [Rendering Mode Control](mode-control.md) | [Mode 3: The BG Pipeline](bg-pipeline.md) |
| LODE | OBP1 bit-2 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| LODY | BGP bit-6 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| LOGO | DMA bus-read window | not_x1 | [OAM DMA](dma.md) |  |
| LOKA | WY bit-3 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| LOKY |  |  | [OAM DMA](dma.md) |  |
| LOLE | WX bit-3 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| LOMA |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Races](races.md) |
| LONY | Tile-fetch latch | nand_latch | [Rendering Mode Control](mode-control.md) | [Scanline and Frame Timing](scanline-frame-timing.md) |
| LOPE | IF[0] capture (VBlank) | dffsr | [Scanline and Frame Timing](scanline-frame-timing.md) | [The IF Register](if-register.md) |
| LOTA |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| LOTE | = NOT(MUMY) — OBP1 read drivers' `ena_n` | not_x1 | [Register Reads](registers/reads.md) |  |
| LOVA | WX bit-0 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| LOVU | LY bit 4 | dffr | [Line Counters](line-counters.md) |  |
| LOZE |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| LUGA | OBP1 bit-5 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| LUKY | OBP1 bit-4 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| LUMA | `dma_addr_ext` driver | not_x1 | [OAM DMA](dma.md) |  |
| LUNA | Temp-latch enable chain | not / nand3 | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Races](races.md) |
| LURY |  |  | [Rendering Mode Control](mode-control.md) |  |
| LUZO |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| LYCO | LY bit-2 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| LYDO | LY bit 3 | dffr | [Line Counters](line-counters.md) |  |
| LYFA |  |  | [Register Writes](registers/writes.md) |  |
| LYHA |  |  | [Line Counters](line-counters.md) | [Mode Transitions](mode-transitions.md) |
| LYKA | BGP bit-5 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| LYKU |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| LYME | Sprite palette pipe output stage (`sprite_px_palette`) | dffsr | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| LYRY | "Fetch complete" combinational | not_x1 | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| LYZA | OBP1 bit-3 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| LYZU | Fetch counter bit 0 sample | dffr | [The Clock Tree](clock-tree.md) | [Rendering Mode Control](mode-control.md), [Mode 3: The BG Pipeline](bg-pipeline.md), [Scanline and Frame Timing](scanline-frame-timing.md) |
| MACU |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| MAKA |  |  | [OAM DMA](dma.md) |  |
| MARA | WX bit-7 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| MASO |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| MATO | LY bit 6 | dffr | [Line Counters](line-counters.md) |  |
| MATU |  |  | [OAM DMA](dma.md) |  |
| MECO | FR PPU-clock feeder | (DFF chain) | [LCD Output](lcd-output.md) |  |
| MEDA | LCD vertical-sync source | dffr | [The Clock Tree](clock-tree.md) | [Line Counters](line-counters.md), [LCD Output](lcd-output.md), [Mode Transitions](mode-transitions.md) |
| MEFU |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| MEGA | WY bit-4 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| MEGU |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| MEHE | NUNU clock buffer | not | [Window Control](window.md) |  |
| MEKE | Reload enable chain | not / nand3 | [The Timer](timer.md) |  |
| MELE | WX bit-4 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| MERA | WY bit-7 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| MERY | Wrap detect | nor2 | [The Timer](timer.md) |  |
| MESU | Fetch counter bit 1 | dffr | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Scanline and Frame Timing](scanline-frame-timing.md) |
| METE |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Races](races.md) |
| MEVE |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| MEXU |  |  | [The Timer](timer.md) |  |
| MOBA | Wrap capture | dffr | [The Timer](timer.md) | [HALT and EI](halt-ei.md), [The IF Register](if-register.md) |
| MOCE | Self-stop condition | nand3 | [The Clock Tree](clock-tree.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [Mode 3: The Sprite Pipeline](sprite-pipeline.md), [Post-Boot State](post-boot.md) |
| MODA | Sprite attribute pipe stage | dffsr | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| MODU |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| MOFO |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| MOJU |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| MOKA | OBP1 palette AO2222 combiner | ao2222 | [LCD Output](lcd-output.md) |  |
| MOKO | WX bit-2 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| MOLU | DMA read enable (AND2 with `ppu_rd`) | and2 | [Register Reads](registers/reads.md) |  |
| MORY | DMA-vs-CPU address MUX select source | nand2 | [OAM DMA](dma.md) |  |
| MOSU | Window trigger | not_x2 | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Window Control](window.md), [Mode Transitions](mode-transitions.md) |
| MOXE | ALET complement (fine-scroll path) | not_x1 | [The Clock Tree](clock-tree.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [Races](races.md) |
| MUDE | LX reset | nor2 | [Line Counters](line-counters.md) |  |
| MUFE | WX bit-5 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| MUKA | WX bit-1 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| MUKU |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| MULY | WX bit-6 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| MUMY | OBP1 read enable (AND2 with `ppu_rd`) | and2 | [Register Reads](registers/reads.md) |  |
| MURE | `s_pad` (VSYNC) driver | not_x1 | [LCD Output](lcd-output.md) |  |
| MUWY | LY bit 0 | dffr | [Line Counters](line-counters.md) | [Register Reads](registers/reads.md), [Mode 2: OAM Scan](oam-scan.md), [Mode Transitions](mode-transitions.md) |
| MYDE |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| MYJY |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| MYRO | LY bit 1 | dffr | [Line Counters](line-counters.md) |  |
| MYTA | FRAME_END capture | dffr | [The Clock Tree](clock-tree.md) | [Line Counters](line-counters.md), [Rendering Mode Control](mode-control.md), [Mode Transitions](mode-transitions.md) |
| MYTE |  |  | [OAM DMA](dma.md) |  |
| MYTU |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| MYVO | ALET complement | not_x1 | [The Clock Tree](clock-tree.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [Scanline and Frame Timing](scanline-frame-timing.md), [Races](races.md) |
| MYZO |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| NAFY |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Races](races.md), [Post-Boot State](post-boot.md) |
| NAPO |  |  | [The Clock Tree](clock-tree.md) |  |
| NASA |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| NATY |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| NAXY |  |  | [OAM DMA](dma.md) |  |
| NAZE |  |  | [Window Control](window.md) |  |
| NEDA |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| NEFO |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| NELE | WY decode completion | not / nand5 / not | [Window Control](window.md) |  |
| NEPO |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| NERU | LY=0 decode | nor8 | [Line Counters](line-counters.md) | [Mode Transitions](mode-transitions.md) |
| NETA | VURY enable producer (tile-data stages) | and2 | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| NEVU |  |  | [Window Control](window.md) |  |
| NOCU | `in_window` driver | not | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| NOFU |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Races](races.md) |
| NOGY | WX register match decoder | nand5 | [Window Control](window.md) |  |
| NOJO |  |  | [Window Control](window.md) |  |
| NOKO | FRAME_END decode | and4 | [Line Counters](line-counters.md) | [Rendering Mode Control](mode-control.md), [Mode Transitions](mode-transitions.md) |
| NOLO |  |  | [OAM DMA](dma.md) |  |
| NOPA | Window mode | dffr | [The Clock Tree](clock-tree.md) | [Window Control](window.md), [Scanline and Frame Timing](scanline-frame-timing.md), [Races](races.md) |
| NOZO |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| NUDU |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| NUFA |  |  | [Window Control](window.md) |  |
| NUGA |  |  | [The Timer](timer.md) |  |
| NUKE | Sprite attribute pipe stage | dffsr | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| NUKO | WX match | not | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Window Control](window.md) |
| NULY | Sprite-visibility mask | nor2 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| NUNU | WX-match capture 2 | dffr | [Window Control](window.md) |  |
| NUNY | Window-trigger pulse | and2 | [Window Control](window.md) |  |
| NUPA |  |  | [Window Control](window.md) |  |
| NURA | BGP palette AO2222 combiner | ao2222 | [LCD Output](lcd-output.md) |  |
| NURO |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| NUVY | DMA bit-7 read driver | not_if1 | [Register Reads](registers/reads.md) |  |
| NYBO | IF[2] capture | dffsr | [The Timer](timer.md) | [The IF Register](if-register.md) |
| NYDU | NUGA pre-wrap capture | dffr | [The Timer](timer.md) |  |
| NYDY |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Races](races.md) |
| NYFO | Trigger buffer | not / not | [Window Control](window.md) |  |
| NYGO | = NOT(MOLU) — DMA read-enable inverter stage 1 | not_x1 | [Register Reads](registers/reads.md) |  |
| NYKA | Fetch-complete capture | dffr | [The Clock Tree](clock-tree.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [Window Control](window.md), [Scanline and Frame Timing](scanline-frame-timing.md) |
| NYLU |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| NYPE |  |  | [The Clock Tree](clock-tree.md) | [Line Counters](line-counters.md), [Rendering Mode Control](mode-control.md), [LCD Output](lcd-output.md) |
| NYVA | Fetch counter bit 2 | dffr | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Scanline and Frame Timing](scanline-frame-timing.md), [Races](races.md) |
| NYXU | BG fetch counter reset | nor3 | [Rendering Mode Control](mode-control.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [Mode 3: The Sprite Pipeline](sprite-pipeline.md), [Window Control](window.md) |
| NYZE | Match capture (even) | dffr | [The Clock Tree](clock-tree.md) | [Rendering Mode Control](mode-control.md), [Mode 3: The BG Pipeline](bg-pipeline.md), [Scanline and Frame Timing](scanline-frame-timing.md) |
| PABA | BGP bit-1 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| PAFU | WY register match decoder | nand5 | [Window Control](window.md) |  |
| PAGA |  |  | [Window Control](window.md) |  |
| PAGO |  |  | [STAT Interrupts](stat-interrupts.md) |  |
| PAHA | ROXY set driver | not_x1 | [Rendering Mode Control](mode-control.md) | [Mode 3: The BG Pipeline](bg-pipeline.md) |
| PAHO | PX-bit-3 capture | dffr | [Rendering Mode Control](mode-control.md) | [LCD Output](lcd-output.md), [Scanline and Frame Timing](scanline-frame-timing.md), [Post-Boot State](post-boot.md) |
| PALO | WY bits 4–7 + LCDC.5 decode | nand5 | [Window Control](window.md) |  |
| PALU | Sprite attribute pipe stage | dffsr | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| PALY | LY==LYC comparator | not_x1 | [STAT Interrupts](stat-interrupts.md) | [CPU-Visible Mode Boundaries](cpu-visible-boundaries.md), [HALT and EI](halt-ei.md) |
| PANE | DMA bit-3 read driver | not_if1 | [Register Reads](registers/reads.md) |  |
| PANY | RYFA data driver (window/fine-scroll NOR) | nor2 | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md), [Window Control](window.md) |
| PARE | DMA bit-4 read driver | not_if1 | [Register Reads](registers/reads.md) |  |
| PARU | Mode 1 condition | not_x1 | [STAT Interrupts](stat-interrupts.md) |  |
| PASO | Fine-counter reset | nor2 | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Post-Boot State](post-boot.md) |
| PATY | Pixel-mux output | or3 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) | [LCD Output](lcd-output.md) |
| PEBA |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| PEBO |  |  | [Window Control](window.md) |  |
| PECU | Fine-counter gated clock | nand2 | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| PEDA |  |  | [The Timer](timer.md) |  |
| PEFO |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| PEFU |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| PELA | WY bit-5 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| PERO | Palette-output multiplexer signal | combinational | [Palette Latches](registers/palette-latches.md) | [LCD Output](lcd-output.md) |
| PERU |  |  | [The Timer](timer.md) |  |
| PEZO |  |  | [Window Control](window.md) |  |
| PODA | WY bit-1 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| POFO | STAT bit-4 read driver (mode-1 enable, from `rufo_n`) | not_if0 | [Register Reads](registers/reads.md) |  |
| POFY | Loop inverter + feedback | not_x1 | [LCD Output](lcd-output.md) | [Mode Transitions](mode-transitions.md), [Post-Boot State](post-boot.md) |
| POGU | `cpg_pad` (clock pulse gate) driver | not_x1 | [LCD Output](lcd-output.md) |  |
| POHU | Match comparator | not_x1 | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| POKA | BG-pixel combine | nor3 | [LCD Output](lcd-output.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| POKY | Pixel-pipe data ready | nor_latch | [Rendering Mode Control](mode-control.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [Window Control](window.md), [Scanline and Frame Timing](scanline-frame-timing.md) |
| POLO | WY bit-6 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| POLY | DMA bit-0 read driver | not_if1 | [Register Reads](registers/reads.md) |  |
| POME | Loop AVAP gate | nor2 | [LCD Output](lcd-output.md) | [Mode Transitions](mode-transitions.md) |
| POMO |  |  | [Window Control](window.md) |  |
| POPU | VBlank capture | dffr | [The Clock Tree](clock-tree.md) | [Line Counters](line-counters.md), [Rendering Mode Control](mode-control.md), [Mode Transitions](mode-transitions.md) |
| PORE |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| PORY | NYKA pipeline stage | dffr | [The Clock Tree](clock-tree.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [Window Control](window.md), [Scanline and Frame Timing](scanline-frame-timing.md) |
| POTE | STAT bit-6 read driver (LYC enable, from `rugu_n`) | not_if0 | [Register Reads](registers/reads.md) |  |
| POVA | Match pulse | and2 | [Rendering Mode Control](mode-control.md) | [Register Reads](registers/reads.md), [Mode 3: The BG Pipeline](bg-pipeline.md), [LCD Output](lcd-output.md) |
| POVY |  |  | [The Timer](timer.md) |  |
| POWU |  |  | [OAM and VRAM Access](oam-vram-access.md) | [OAM DMA](dma.md) |
| PUDU |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| PUFY | LYC bit-7 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| PUKU | Window hit-and-latch | nor2 / nor3 | [Window Control](window.md) |  |
| PUKY | WX-match decode | nand5 / not / nand5 / not | [Window Control](window.md) |  |
| PUNU | WY bit-0 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| PUPU |  |  | [Register Writes](registers/writes.md) |  |
| PUSY | = NOT(NYGO) — DMA read drivers' active-high `ena` | not_x1 | [Register Reads](registers/reads.md) |  |
| PUXA | Match capture (odd) | dffr | [The Clock Tree](clock-tree.md) | [Rendering Mode Control](mode-control.md), [Mode 3: The BG Pipeline](bg-pipeline.md), [Scanline and Frame Timing](scanline-frame-timing.md) |
| PUZO | STAT bit-3 read driver (mode-0 enable, from `roxe_n`) | not_if0 | [Register Reads](registers/reads.md) |  |
| PYBO |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) | [LCD Output](lcd-output.md) |
| PYCO | WX-match capture 1 | dffr | [Window Control](window.md) |  |
| PYGO | PORY pipeline stage | dffr | [The Clock Tree](clock-tree.md) | [Rendering Mode Control](mode-control.md), [Mode 3: The BG Pipeline](bg-pipeline.md), [Scanline and Frame Timing](scanline-frame-timing.md) |
| PYGU | WY bit-2 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| PYJO |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| PYJU | Tile-index bit-7 capture DFF | dffr_cc_q | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| PYNU | Window-armed latch | nor_latch | [Register Reads](registers/reads.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [Window Control](window.md) |
| RACE | LYC bit-4 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| RACO |  |  | [OAM and VRAM Access](oam-vram-access.md) |  |
| RAGE |  |  | [The Timer](timer.md) |  |
| RAHU |  |  | [OAM and VRAM Access](oam-vram-access.md) |  |
| RAJY | BG plane A output (LCDC.0 gated) | and2 | [LCDC Structure](registers/lcdc.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md), [LCD Output](lcd-output.md) |
| RALU |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| RALY | DMA bit-5 read driver | not_if1 | [Register Reads](registers/reads.md) |  |
| RAMA |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| RAMU |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| RAPE |  |  | [STAT Interrupts](stat-interrupts.md) |  |
| RARO | BGP bit-0 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| RATE |  |  | [The Timer](timer.md) |  |
| RAVO | LD1 / LD0 drivers | not | [Register Reads](registers/reads.md) | [LCD Output](lcd-output.md) |
| RAZU | LYC bit-2 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| REDO | BGP bit-2 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| REDY | LYC bit-3 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| REFE |  |  | [STAT Interrupts](stat-interrupts.md) |  |
| REGA | TIMA bits 0–7 | tffnl (toggle FF with load) | [The Timer](timer.md) |  |
| REJO | WY-match frame latch | nor_latch | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Window Control](window.md), [Scanline and Frame Timing](scanline-frame-timing.md) |
| REMA | DMA bit-2 read driver | not_if1 | [Register Reads](registers/reads.md) |  |
| REMY | LD0 pad driver | not_x2 | [Register Reads](registers/reads.md) | [LCD Output](lcd-output.md) |
| RENE | Drain-detect stage 2 | dffr | [The Clock Tree](clock-tree.md) | [Rendering Mode Control](mode-control.md), [Mode 3: The BG Pipeline](bg-pipeline.md), [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| REPU | REJO reset | or2 | [Window Control](window.md) | [Scanline and Frame Timing](scanline-frame-timing.md) |
| RESU | DMA bit-6 read driver | not_if1 | [Register Reads](registers/reads.md) |  |
| RETU | LYC bit-0 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| REVO |  |  | [OAM and VRAM Access](oam-vram-access.md) |  |
| REWO |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| ROBY |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| ROCO |  |  | [Window Control](window.md) |  |
| ROCY | Per-bit data-pad tri-state enable chain | combinational | [OAM and VRAM Access](oam-vram-access.md) |  |
| ROFO | DMA bit-1 read driver | not_if1 | [Register Reads](registers/reads.md) |  |
| ROGA | Fine count bit 1 | dffr | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| ROGE | WY match | not | [Window Control](window.md) |  |
| ROKA |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| ROMO | SUVU NAND4 input (POKY inverter) | not_x1 | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| RONE | Drain-gated match aggregate | nand4 | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| ROPE | = NAND2(SOHA, RYCU) — $FExx decode stage | nand2 | [OAM and VRAM Access](oam-vram-access.md) |  |
| ROPO | LY==LYC synced match | dff17 | [The Clock Tree](clock-tree.md) | [Line Counters](line-counters.md), [Register Reads](registers/reads.md), [STAT Interrupts](stat-interrupts.md) |
| ROPY | Mode 3 inverter for VRAM enables | not_x1 | [Rendering Mode Control](mode-control.md) | [OAM and VRAM Access](oam-vram-access.md) |
| RORU |  |  | [OAM DMA](dma.md) |  |
| ROSA | Sprite attribute pipe stage | dffsr | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| ROSY |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| ROXE | STAT.3/.4/.5/.6 enables | drlatch_ee | [STAT Interrupts](stat-interrupts.md) |  |
| ROXO | Inverted CLKPIPE buffer (fine-scroll clock source) | not_x1 | [Mode 3: The BG Pipeline](bg-pipeline.md) | [LCD Output](lcd-output.md) |
| ROXY | Fine-scroll gate | nor_latch | [Rendering Mode Control](mode-control.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [Scanline and Frame Timing](scanline-frame-timing.md), [Post-Boot State](post-boot.md) |
| ROZE | Fine-counter self-stop | nand3 | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| RUBU | Fine count bit 2 | dffr | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| RUBY |  |  | [The Timer](timer.md) |  |
| RUDA |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| RUFO |  |  | [STAT Interrupts](stat-interrupts.md) |  |
| RUGO | Sprite attribute pipe stage | dffsr | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| RUGU |  |  | [STAT Interrupts](stat-interrupts.md) |  |
| RUJU | Loop OR combiner | or3 | [LCD Output](lcd-output.md) |  |
| RUPO | STAT bit 2 visible latch | nor_latch | [STAT Interrupts](stat-interrupts.md) |  |
| RUTA | BG plane B gated by sprite priority | and2 | [LCD Output](lcd-output.md) |  |
| RUTU | LINE_END capture | dffr | [The Clock Tree](clock-tree.md) | [Line Counters](line-counters.md), [Rendering Mode Control](mode-control.md), [Register Writes](registers/writes.md) |
| RUZE | ST pad buffer | not_x3 | [LCD Output](lcd-output.md) |  |
| RYCE | One-shot fire | and2 | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| RYCU | ROPE input (= NOT(`fexx_ffxx_n`)) | not_x1 | [OAM and VRAM Access](oam-vram-access.md) |  |
| RYDU |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| RYDY | Window hit | nor3 | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md), [Window Control](window.md) |
| RYFA | Drain-detect stage 1 | dffr | [Rendering Mode Control](mode-control.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [Mode 3: The Sprite Pipeline](sprite-pipeline.md), [Window Control](window.md) |
| RYFU | BG plane A gated by sprite priority | and2 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) | [LCD Output](lcd-output.md) |
| RYKU | Fine count bit 0 | dffr | [The Clock Tree](clock-tree.md) | [Mode 3: The BG Pipeline](bg-pipeline.md) |
| RYNO |  |  | [LCD Output](lcd-output.md) |  |
| RYPO | CP pad driver | not_x3 | [LCD Output](lcd-output.md) |  |
| RYSA |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| RYVE |  |  | [Register Writes](registers/writes.md) |  |
| SABE | Gated fetch clock | nand2 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) | [Scanline and Frame Timing](scanline-frame-timing.md) |
| SACU | Pixel pipe shift clock (CLKPIPE) | or2 | [Register Writes](registers/writes.md) | [Register Reads](registers/reads.md), [Mode 2: OAM Scan](oam-scan.md), [Mode 3: The BG Pipeline](bg-pipeline.md) |
| SADU |  |  | [Rendering Mode Control](mode-control.md) | [Register Reads](registers/reads.md), [CPU-Visible Mode Boundaries](cpu-visible-boundaries.md) |
| SADY |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| SAJA |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| SANU | LINE_END decode | and4 | [The Clock Tree](clock-tree.md) | [Line Counters](line-counters.md), [Rendering Mode Control](mode-control.md), [Mode Transitions](mode-transitions.md) |
| SARY | WY-match sampler | dffr | [Window Control](window.md) |  |
| SASY | STAT bit-5 read driver (mode-2 enable, from `refe_n`) | not_if0 | [Register Reads](registers/reads.md) |  |
| SATA | Sprite attribute pipe stage | dffsr | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| SAVY | PX bit 1 | dffr | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| SAXO | LX bit 0 | dffr | [The Clock Tree](clock-tree.md) | [Line Counters](line-counters.md), [Races](races.md), [Post-Boot State](post-boot.md) |
| SAZO |  |  | [OAM and VRAM Access](oam-vram-access.md) |  |
| SEBA | Fetch-counter TULY capture stage 3 | dffr | [Rendering Mode Control](mode-control.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md), [Scanline and Frame Timing](scanline-frame-timing.md), [Races](races.md) |
| SEBY | VRAM read-latch enable | not_x1 | [OAM and VRAM Access](oam-vram-access.md) |  |
| SECA | TAKA set net | nor3 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) | [Mode Transitions](mode-transitions.md) |
| SEGA |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| SEGO | STAT bit-2 read driver (LYC match, from `rupo_n`) | not_if1 | [Register Reads](registers/reads.md) |  |
| SEGU | Buffered CLKPIPE | not_x4 | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md), [Scanline and Frame Timing](scanline-frame-timing.md), [Post-Boot State](post-boot.md) |
| SEKO | Tile-boundary drain detector | nor2 | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md), [Window Control](window.md) |
| SELA |  |  | [Mode Transitions](mode-transitions.md) | [STAT Interrupts](stat-interrupts.md) |
| SELE |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| SEMO |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| SEMU | CP pixel-clock source (pre-buffer) | or2 | [LCD Output](lcd-output.md) |  |
| SEPA | STAT write enable | and2 | [Register Writes](registers/writes.md) |  |
| SERE | CPU data-bus enable stage 2 | and2 | [OAM and VRAM Access](oam-vram-access.md) |  |
| SETU |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| SOBO |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| SOBU | Fetch request | dffr | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md), [Scanline and Frame Timing](scanline-frame-timing.md) |
| SOCY | Window-halt gate | not_x1 | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Window Control](window.md), [Post-Boot State](post-boot.md) |
| SOGU | TIMA clock gate | nor2 | [The Timer](timer.md) |  |
| SOHA | ROPE input (= NOT(`ffxx`)) | not_x1 | [OAM and VRAM Access](oam-vram-access.md) |  |
| SOHU |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) | [LCD Output](lcd-output.md) |
| SOHY | Write-strobe driver | nand2 | [OAM and VRAM Access](oam-vram-access.md) |  |
| SOKA |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| SOMY | Sprite attribute pipe stage | dffsr | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| SONO | 1 MHz LINE_END capture clock (TALU complement) | not_x1 | [The Clock Tree](clock-tree.md) | [Line Counters](line-counters.md), [Rendering Mode Control](mode-control.md), [Register Writes](registers/writes.md) |
| SOVU |  |  | [STAT Interrupts](stat-interrupts.md) |  |
| SOWO | TEKY re-trigger block (TAKA inversion) | not_x1 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| SOZU | Fine-count bit 2 match | xnor | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| STOP |  |  | [OAM DMA](dma.md) | [The Timer](timer.md) |
| SUBO |  |  | [STAT Interrupts](stat-interrupts.md) |  |
| SUDA | Trigger one-shot partner | dffr | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md), [Scanline and Frame Timing](scanline-frame-timing.md) |
| SUDE | LX bit 4 | dffr | [Line Counters](line-counters.md) |  |
| SUHA | Per-bit SCX match | xnor | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| SUKO | Condition combining gate | ao2222 | [STAT Interrupts](stat-interrupts.md) | [The IF Register](if-register.md) |
| SUNY |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| SUTO |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| SUVU | Startup-/window-restart NAND4 (TAVE input) | nand4 | [Rendering Mode Control](mode-control.md) | [Mode 3: The BG Pipeline](bg-pipeline.md) |
| SUZU | Window-activation trigger | not | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Window Control](window.md) |
| SYBE |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| SYBY | Fine-count bit 1 match | xnor | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| SYGU |  |  | [The Clock Tree](clock-tree.md) | [LCD Output](lcd-output.md), [Races](races.md) |
| SYLO | RYDY first inverter (consumer-chain stage 1) | not | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Window Control](window.md) |
| SYSY |  |  | [OAM and VRAM Access](oam-vram-access.md) |  |
| TACA |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| TADE | BG plane B output (LCDC.0 gated) | and2 | [LCDC Structure](registers/lcdc.md) | [LCD Output](lcd-output.md) |
| TADY | VOGA + PX-counter reset | nor2 | [Rendering Mode Control](mode-control.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [Window Control](window.md), [Mode Transitions](mode-transitions.md) |
| TAHA | LX bit 5 | dffr | [Line Counters](line-counters.md) |  |
| TAKA | Fetch-running latch | nand_latch | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md), [Window Control](window.md), [Mode Transitions](mode-transitions.md) |
| TAKO |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| TALU | 1 MHz LX counter clock | not_x4 | [The Clock Tree](clock-tree.md) | [Line Counters](line-counters.md), [Register Writes](registers/writes.md), [Register Reads](registers/reads.md) |
| TAME | Self-stop | nand2 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| TAPA | Mode 2 condition | and2 | [STAT Interrupts](stat-interrupts.md) |  |
| TAPE |  |  | [The Timer](timer.md) |  |
| TAPU | CUPA driver stage 2 | not | [Register Writes](registers/writes.md) |  |
| TARU | Mode 0 condition | and2 | [STAT Interrupts](stat-interrupts.md) |  |
| TAVA | SOBU clock gate | not_x1 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| TAVE | Startup / window-restart trigger | not | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| TAXY | CPU-bus-phase strobe assembly | combinational | [OAM and VRAM Access](oam-vram-access.md) |  |
| TEBY | STAT bit-0 read driver (from SADU) | not_if1 | [Register Reads](registers/reads.md) |  |
| TECY |  |  | [The Timer](timer.md) |  |
| TEGO | = NOT(VAMA) — $FF49 active-high select | not_x2 | [Register Reads](registers/reads.md) |  |
| TEKO |  |  | [The Timer](timer.md) |  |
| TEKY | X-match trigger | and4 | [Mode 2: OAM Scan](oam-scan.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [Mode 3: The Sprite Pipeline](sprite-pipeline.md), [Scanline and Frame Timing](scanline-frame-timing.md) |
| TELU | LX bit 3 | dffr | [Line Counters](line-counters.md) |  |
| TEPA | NOT(mode3) | not_x1 | [Rendering Mode Control](mode-control.md) |  |
| TEPO | BGP write enable sub-signal | combinational | [Register Writes](registers/writes.md) | [Palette Latches](registers/palette-latches.md) |
| TEPY | = NOT(VUSO) — BGP read drivers' `ena_n` | not_x1 | [Register Reads](registers/reads.md) |  |
| TESE | Sprite fetch counter bit 2 (SFETCH_S2) | dffr | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| TEVO | Tile-boundary / restart trigger | or3 | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md), [Window Control](window.md), [Mode Transitions](mode-transitions.md) |
| TOBA |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) | [LCD Output](lcd-output.md) |
| TOBE | FF41 read enable | and2 | [Register Reads](registers/reads.md) |  |
| TOBU | TULY capture chain | dffr | [Rendering Mode Control](mode-control.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md), [Scanline and Frame Timing](scanline-frame-timing.md), [Races](races.md) |
| TOCA | Upper-nibble clock | not | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Races](races.md) |
| TOFU | Video reset | not | [Rendering Mode Control](mode-control.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [LCD Output](lcd-output.md), [Mode Transitions](mode-transitions.md) |
| TOLE | CPU data-bus enable stage 1 | combinational | [OAM and VRAM Access](oam-vram-access.md) |  |
| TOLU | mode1 inverter | not_x1 | [STAT Interrupts](stat-interrupts.md) | [The IF Register](if-register.md) |
| TOMU | RYDY second inverter (consumer-chain stage 2) | not | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Window Control](window.md) |
| TOMY |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| TORY |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| TOXE | Fetch counter bits 0–2 | dffr | [The Clock Tree](clock-tree.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| TOZA |  |  | [OAM DMA](dma.md) |  |
| TUHU | PX bits 4–7 | dffr | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| TUKU | Sprite-trigger window block (consumer-chain stage 3) | not | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) | [Window Control](window.md) |
| TUKY |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| TULY | Sprite fetch counter bit 1 (SFETCH_S1) | dffr | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| TUTO |  |  | [OAM and VRAM Access](oam-vram-access.md) |  |
| TUVA | Inverter pair | not | [STAT Interrupts](stat-interrupts.md) | [The IF Register](if-register.md) |
| TUVO |  |  | [OAM DMA](dma.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| TUXY | Window-trigger NAND (SUZU input) | nand2 | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Window Control](window.md) |
| TYCO |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| TYFA | CLKPIPE gate | and3 | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Scanline and Frame Timing](scanline-frame-timing.md), [Post-Boot State](post-boot.md) |
| TYFO | Done decode B | dffr | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) | [Scanline and Frame Timing](scanline-frame-timing.md) |
| TYHO |  |  | [OAM DMA](dma.md) |  |
| TYJY |  |  | [OAM and VRAM Access](oam-vram-access.md) |  |
| TYNO | Done decode A | nand3 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| TYPO | LX bit 1 | dffr | [Line Counters](line-counters.md) |  |
| TYRY | LX bit 6 | dffr | [Line Counters](line-counters.md) | [Post-Boot State](post-boot.md) |
| TYTA |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| UBAL | CUPA driver stage 3 | muxi | [Register Writes](registers/writes.md) |  |
| UBUL | IF[3] capture (Serial) | dffsr | [The IF Register](if-register.md) |  |
| UCOB |  |  | [The Timer](timer.md) |  |
| UKAP | TAC mux chain | muxi | [The Timer](timer.md) |  |
| UKUP | `reg_div16` bits 0–15 | dff | [The Timer](timer.md) |  |
| ULAK | IF[4] capture (Joypad) | dffsr | [The IF Register](if-register.md) |  |
| UMOB | CPL audio-clock source (LCD-off arm) | (external) | [LCD Output](lcd-output.md) |  |
| UPOF |  |  | [The Timer](timer.md) |  |
| USEC | FR audio-clock source (LCD-off arm) | (external) | [LCD Output](lcd-output.md) |  |
| UVYT |  |  | [OAM DMA](dma.md) |  |
| VAFE | LYC bit-6 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| VAFO |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| VAHA | LCDC bit-6 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| VAMA | $FF49 address decode | nand5 | [Register Reads](registers/reads.md) |  |
| VANU |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| VARE |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| VARO | = NOT(WAFU) — LY read drivers' `ena_n` | not_x1 | [Register Reads](registers/reads.md) |  |
| VARY | = NOT(WOFA) — $FF41 active-high select | not_x2 | [Register Reads](registers/reads.md) | [Register Writes](registers/writes.md) |
| VATO | LCDC bit-5 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| VAVA | Priority mask pipe MSB / LSB | dffsr | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) | [LCD Output](lcd-output.md) |
| VAVE | NOT(TOBE) — STAT enable-bit read gate | not_x1 | [Register Reads](registers/reads.md) |  |
| VAZU | LYC bit-5 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| VCLK |  |  | [LCD Output](lcd-output.md) |  |
| VEGA | LY bit-0 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| VEKU |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| VELY | BGP write enable | and2 | [Register Writes](registers/writes.md) | [Register Reads](registers/reads.md) |
| VENA | 1 MHz toggle divider | dffr | [The Clock Tree](clock-tree.md) | [Line Counters](line-counters.md), [Register Writes](registers/writes.md), [Scanline and Frame Timing](scanline-frame-timing.md) |
| VEVY | LCDC.6 → ~ma10 tri-state driver | not_if0 | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| VEZO | Mask pipe (LSB end) | dffsr | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| VOCA | = NOT(WORU) — $FF40 active-high select | not_x2 | [Register Reads](registers/reads.md) | [Register Writes](registers/writes.md) |
| VOGA | H-Blank capture DFF | dffr | [The Clock Tree](clock-tree.md) | [Rendering Mode Control](mode-control.md), [Mode 3: The BG Pipeline](bg-pipeline.md), [Mode Transitions](mode-transitions.md) |
| VOJO | LYC bit-1 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| VOKE | LCDC bit-4 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| VOMY | = NOT(WAXU) — WY read drivers' `ena_n` | not_x1 | [Register Reads](registers/reads.md) |  |
| VONU | Fetch-counter TULY capture stage 2 | dffr | [Rendering Mode Control](mode-control.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md), [Scanline and Frame Timing](scanline-frame-timing.md), [Races](races.md) |
| VOSA | Sprite attribute pipe stage | dffsr | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| VOTO |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| VOTY |  |  | [STAT Interrupts](stat-interrupts.md) | [The IF Register](if-register.md) |
| VUJO |  |  | [Window Control](window.md) |  |
| VUMO | Sprite attribute pipe stage | dffsr | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| VUMY | = NOT(WAGE) — $FF4B active-high select | not_x2 | [Register Reads](registers/reads.md) |  |
| VUPY |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| VURY | VUZA → ~ma12 tri-state driver | not_if1 | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| VUSA | Fetch-done aggregate | or2 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| VUSO | BGP read enable (AND2 with `ppu_rd`) | and2 | [Register Reads](registers/reads.md) |  |
| VUZA | TILE_SEL XOR tile-index MSB | nor2 | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| VYBO | Operational per-dot gate | nor3 | [Mode 3: The BG Pipeline](bg-pipeline.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md), [Scanline and Frame Timing](scanline-frame-timing.md), [Post-Boot State](post-boot.md) |
| VYCU | = NOT(WYZE) — WX read drivers' `ena_n` | not_x1 | [Register Reads](registers/reads.md) |  |
| VYGA | = NOT(WYVO) — $FF4A active-high select | not_x2 | [Register Reads](registers/reads.md) |  |
| VYNE | LY bit-4 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| VYNO | Window line counter LSB | dffr | [Window Control](window.md) |  |
| VYPU | int_vbl driver | not_x3 | [The IF Register](if-register.md) |  |
| VYRE | LCDC read enable (AND2 with `ppu_rd`) | and2 | [Register Reads](registers/reads.md) |  |
| VYSA |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| VYXE |  |  | [LCDC Structure](registers/lcdc.md) | [LCD Output](lcd-output.md) |
| VYZO | LX bit 2 | dffr | [Line Counters](line-counters.md) |  |
| WAFU | LY read enable (AND2 with `ppu_rd`) | and2 | [Register Reads](registers/reads.md) |  |
| WAGE | $FF4B address decode | nand5 | [Register Reads](registers/reads.md) |  |
| WAGO | 8×16 half-select | xor | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| WAHE | OAM wordline-driver precharge (`oam_wldrv_precharge_n` = NOT(WUJY)) | not_x3 | [OAM and VRAM Access](oam-vram-access.md) |  |
| WAMA | LY bit-5 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| WAME | OAM output-enable inverter | not_x2 | [OAM and VRAM Access](oam-vram-access.md) |  |
| WARE | SCY bit-0 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| WARU | LCDC write enable | and2 | [Register Writes](registers/writes.md) | [Mode 3: The BG Pipeline](bg-pipeline.md) |
| WATE | $FF46 address decode | nand5 | [Register Reads](registers/reads.md) |  |
| WAVO | LY bit-6 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| WAVU | $FF43 address decode | nand5 | [Register Reads](registers/reads.md) |  |
| WAXU | WY read enable (AND2 with `ppu_rd`) | and2 | [Register Reads](registers/reads.md) |  |
| WAZO | WUJY input — wordline-precharge timing | or3 | [OAM and VRAM Access](oam-vram-access.md) |  |
| WAZY | Window line counter clock | not_x1 | [Window Control](window.md) |  |
| WEBA |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| WEBU | $FF42 address decode | nand5 | [Register Reads](registers/reads.md) |  |
| WEFY |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| WEGO | XYMU set driver | or2 | [Rendering Mode Control](mode-control.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [Mode Transitions](mode-transitions.md) |
| WEKU | = NOT(XYLY) — LYC read drivers' `ena_n` | not_x1 | [Register Reads](registers/reads.md) |  |
| WELO | Slot-0 X store bit 4 (holds NOT(X)) | drlatch_ee | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| WERA | = NOT(WYBO) — $FF47 active-high select | not_x2 | [Register Reads](registers/reads.md) | [Register Writes](registers/writes.md) |
| WETA | $FF48 address decode | nand5 | [Register Reads](registers/reads.md) |  |
| WETY | $FF45 address decode | nand5 | [Register Reads](registers/reads.md) |  |
| WEWY | Scan counter bit 1 | dffr | [Mode 2: OAM Scan](oam-scan.md) |  |
| WEXU | LCDC.4 (TILE_SEL) latch | drlatch_ee | [LCDC Structure](registers/lcdc.md) | [Mode 3: The BG Pipeline](bg-pipeline.md) |
| WEZE | LY bit-7 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| WODA | Sprite attribute pipe stage | dffsr | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| WODU | Mode 0 condition | and2 | [Rendering Mode Control](mode-control.md) | [Register Writes](registers/writes.md), [Register Reads](registers/reads.md), [Mode 3: The BG Pipeline](bg-pipeline.md) |
| WOFA | $FF41 address decode | nand5 | [Register Reads](registers/reads.md) |  |
| WOJO | LCD control phase | nor2 | [The Clock Tree](clock-tree.md) |  |
| WOJU | Slot-0 X comparator bit 4 (XOR vs NOT(PX)) | xor | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| WOJY | LY bit-3 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| WOKY | LCDC.6 (WIN_MAP) latch | drlatch_ee | [LCDC Structure](registers/lcdc.md) | [Mode 3: The BG Pipeline](bg-pipeline.md) |
| WONE |  |  | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md) |
| WONY | SCX bit-3 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| WOPE |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| WORA |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| WORU | $FF40 address decode | nand5 | [Register Reads](registers/reads.md) |  |
| WOSU | WUVU sampler | dffr | [The Clock Tree](clock-tree.md) | [Scanline and Frame Timing](scanline-frame-timing.md) |
| WOTA | Y-compare decoder | nand6 | [Mode 2: OAM Scan](oam-scan.md) |  |
| WOTE | Slot-0 X store bit 6 (holds NOT(X)) | drlatch_ee | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| WOVU | OAM bitline precharge driver (`oam_bl_precharge_n` = NOT(COTA)) | not_x2 | [OAM and VRAM Access](oam-vram-access.md) |  |
| WOXA | Plane-A pixel-output gate | and2 | [LCDC Structure](registers/lcdc.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| WUDA |  |  | [Mode 2: OAM Scan](oam-scan.md) |  |
| WUFU | OBP0 palette AO2222 combiner | ao2222 | [LCD Output](lcd-output.md) |  |
| WUFY |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| WUGA | STAT bit-1 read driver (from XATY) | not_if1 | [Register Reads](registers/reads.md) |  |
| WUHU | Y-compare carry-chain bit 7 | full_add | [Mode 2: OAM Scan](oam-scan.md) |  |
| WUJY | = NAND2(WAZO, `oam_bl_precharge_n`) — wordline-precharge timing | nand2 | [OAM and VRAM Access](oam-vram-access.md) |  |
| WUKA | LCDC bit-3 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| WUKO | VEVY enable (window tilemap stage) | not_x2 | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| WUKY | Y-flip term | not_x1 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| WURU | Sprite attribute pipe stage | dffsr | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| WUTY | Fetch done | not | [Mode 2: OAM Scan](oam-scan.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| WUVA | LY bit-1 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| WUVU | 2 MHz toggle divider | dffr | [The Clock Tree](clock-tree.md) | [Register Writes](registers/writes.md), [Mode 2: OAM Scan](oam-scan.md), [Scanline and Frame Timing](scanline-frame-timing.md) |
| WYBO | $FF47 address decode | nand5 | [Register Reads](registers/reads.md) |  |
| WYCE | = NOT(VYRE) — LCDC read drivers' `ena_n` | not_x1 | [Register Reads](registers/reads.md) |  |
| WYFU | Sprite attribute pipe stage | dffsr | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| WYHO |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| WYJA | OAM write enable (combined) | ao21 | [OAM and VRAM Access](oam-vram-access.md) | [OAM DMA](dma.md), [CPU-Visible Mode Boundaries](cpu-visible-boundaries.md) |
| WYJU | LCDC bit-2 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| WYLE | $FF44 address decode | nand5 | [Register Reads](registers/reads.md) |  |
| WYMO |  |  | [LCDC Structure](registers/lcdc.md) |  |
| WYNO |  |  | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md) |
| WYPO | LCDC bit-0 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| WYSO |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| WYVO | $FF4A address decode | nand5 | [Register Reads](registers/reads.md) |  |
| WYZA | Slot-0 X comparator bit 6 (XOR vs NOT(PX)) | xor | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| WYZE | WX read enable (AND2 with `ppu_rd`) | and2 | [Register Reads](registers/reads.md) |  |
| XAFO | LCDC.3 (BG_MAP) latch | drlatch_ee | [LCDC Structure](registers/lcdc.md) | [Mode 3: The BG Pipeline](bg-pipeline.md) |
| XAFU |  |  | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md) |
| XAGE | Per-slot X match, FEFY group (active-low) | nand3 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| XAJU | OBP0 bit-4 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| XAKO | Slot-0 X store bit 7 (holds NOT(X)) | drlatch_ee | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| XANE | VRAM address tri-state-disable source | nor2 | [Rendering Mode Control](mode-control.md) | [OAM and VRAM Access](oam-vram-access.md) |
| XANO | "PX at terminal count" (inverted XUGU) | not_x1 | [Rendering Mode Control](mode-control.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [Mode Transitions](mode-transitions.md), [STAT Interrupts](stat-interrupts.md) |
| XAPO |  |  | [Rendering Mode Control](mode-control.md) | [Mode 2: OAM Scan](oam-scan.md) |
| XARO | = NOT(WEBU) — $FF42 active-high select | not_x2 | [Register Reads](registers/reads.md) | [Register Writes](registers/writes.md) |
| XARY | OBP0 bit-0 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| XATY |  |  | [Rendering Mode Control](mode-control.md) | [Register Reads](registers/reads.md), [CPU-Visible Mode Boundaries](cpu-visible-boundaries.md) |
| XAVY | = NOT(WAVU) — $FF43 active-high select | not_x2 | [Register Reads](registers/reads.md) | [Register Writes](registers/writes.md) |
| XAWO | OBP0 bit-7 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| XAXA | OBP0 bit-6 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| XAYO | = NOT(WETA) — $FF48 active-high select | not_x2 | [Register Reads](registers/reads.md) |  |
| XAYU | = NOT(WETY) — $FF45 active-high select | not_x2 | [Register Reads](registers/reads.md) |  |
| XEBA | Slot-0 match, high-nibble collapse | nor4 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| XEBU | LCDC bit-7 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| XEDA | = NOT(WATE) — $FF46 active-high select | not_x2 | [Register Reads](registers/reads.md) |  |
| XEDU | VRAM address tri-state enable | not_x2 | [OAM and VRAM Access](oam-vram-access.md) |  |
| XEFY | Global sprite shifter parallel-load disable | not_x1 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| XEGU |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| XEHO | PX bit 0 | dffr | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| XENA | "No sprite match" (inverted FEPO) | not_x1 | [Rendering Mode Control](mode-control.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [Mode Transitions](mode-transitions.md), [STAT Interrupts](stat-interrupts.md) |
| XEPE | Slot-0 X store bit 0 (holds NOT(X)) | drlatch_ee | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| XERO | LCDC bit-1 read driver | not_if0 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) | [Register Reads](registers/reads.md) |
| XETE | Sprite attribute pipe stage | dffsr | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| XEZE |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| XOBO | OBP0 bit-5 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| XOCE |  |  | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md) |
| XODO | VID_RST signal | combinational | [Register Writes](registers/writes.md) | [LCDC Structure](registers/lcdc.md), [Scanline and Frame Timing](scanline-frame-timing.md) |
| XODU | PX bit 2 | dffr | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| XOFO | PYNU reset (WIN_EN gate) | nand3 | [LCDC Structure](registers/lcdc.md) | [Window Control](window.md) |
| XOGS |  |  | [Interrupt Dispatch](interrupt-dispatch.md) |  |
| XOGY | = NOT(WYLE) — $FF44 active-high select | not_x2 | [Register Reads](registers/reads.md) |  |
| XOKE | OBP0 bit-1 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| XONA | LCDC.7 storage latch | drlatch_ee | [Register Writes](registers/writes.md) | [LCDC Structure](registers/lcdc.md) |
| XOTA | WUVU clock | not_x1 | [The Clock Tree](clock-tree.md) |  |
| XOTE |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| XOVU |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| XOZY | = NOT(XUFY) — OBP0 read drivers' `ena_n` | not_x1 | [Register Reads](registers/reads.md) |  |
| XUBO |  |  | [Register Writes](registers/writes.md) |  |
| XUBY | OBP0 bit-3 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| XUFY | OBP0 read enable (AND2 with `ppu_rd`) | and2 | [Register Reads](registers/reads.md) |  |
| XUGU | Terminal count decode | nand5 | [Rendering Mode Control](mode-control.md) | [Mode 3: The BG Pipeline](bg-pipeline.md), [Window Control](window.md), [Mode Transitions](mode-transitions.md) |
| XUJA |  |  | [OAM DMA](dma.md) |  |
| XUJY | Sprite-fetch counter term in BYCU | combinational | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md), [OAM and VRAM Access](oam-vram-access.md) |
| XUKE |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| XULA | Plane-B pixel-output gate | and2 | [LCDC Structure](registers/lcdc.md) | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| XUNO | OBP0 bit-2 read driver | not_if0 | [Register Reads](registers/reads.md) |  |
| XUNY | Slot-0 X store bit 5 (holds NOT(X)) | drlatch_ee | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| XUPY | 2 MHz scan-counter clock (NOT(wuvu_n) = WUVU.q) | not_x2 | [The Clock Tree](clock-tree.md) | [Register Writes](registers/writes.md), [Mode 2: OAM Scan](oam-scan.md), [Mode 3: The BG Pipeline](bg-pipeline.md) |
| XURE |  |  | [Register Writes](registers/writes.md) |  |
| XUSO | OAM Y-offset bit 0 latch (Mode 2) / tile-index bit 0 latch (Mode 3) | dlatch_ee | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md), [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| XYDO | PX bit 3 | dffr | [Mode 3: The BG Pipeline](bg-pipeline.md) | [LCD Output](lcd-output.md), [Races](races.md) |
| XYFY | XOTA complement | not_x1 | [The Clock Tree](clock-tree.md) |  |
| XYJU |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| XYKY |  |  | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md), [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| XYLE |  |  | [Mode 3: The BG Pipeline](bg-pipeline.md) |  |
| XYLO | LCDC.1 (OBJ_EN) register bit | drlatch_ee | [LCDC Structure](registers/lcdc.md) | [Mode 2: OAM Scan](oam-scan.md), [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| XYLY | LYC read enable (AND2 with `ppu_rd`) | and2 | [Register Reads](registers/reads.md) |  |
| XYMO | LCDC.2 (OBJ_SIZE) register bit | drlatch_ee | [LCDC Structure](registers/lcdc.md) | [Mode 2: OAM Scan](oam-scan.md), [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| XYMU | Rendering-mode latch (active-low Mode 3 indicator) | nor_latch | [Rendering Mode Control](mode-control.md) | [Register Writes](registers/writes.md), [OAM and VRAM Access](oam-vram-access.md), [OAM DMA](dma.md) |
| XYSO |  |  | [Mode 2: OAM Scan](oam-scan.md) |  |
| XYVA | Parallel divider branch | not_x1 | [The Clock Tree](clock-tree.md) |  |
| XYVO | VBlank decode | and2 | [Line Counters](line-counters.md) | [Rendering Mode Control](mode-control.md), [Mode Transitions](mode-transitions.md) |
| YBEZ | Per-slot X match, FOVE group (active-low) | nand3 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| YBOG |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| YCEB |  |  | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md) |
| YDUG | Slot-0 X match (active-low) | nand3 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| YDYV |  |  | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md), [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| YFEL | Scan counter bit 0 (LSB) | dffr | [Mode 2: OAM Scan](oam-scan.md) |  |
| YFUN | Slot-0 X comparator bit 5 (XOR vs NOT(PX)) | xor | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| YGEM | Per-slot X match, FOVE group (active-low) | nand3 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| YJEX |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| YKUA |  |  | [HALT and EI](halt-ei.md) |  |
| YLAH | Slot-0 X store bit 1 (holds NOT(X)) | drlatch_ee | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| YLOR |  |  | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md), [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| YLOZ | Per-slot X match, FEFY group (active-low) | nand3 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| YLYC | OAM A-side / B-side write strobe drivers | per-byte selector | [OAM and VRAM Access](oam-vram-access.md) | [OAM DMA](dma.md) |
| YNKW |  |  | [HALT and EI](halt-ei.md) |  |
| YNYC |  |  | [OAM and VRAM Access](oam-vram-access.md) | [OAM DMA](dma.md) |
| YOII |  |  | [Interrupt Dispatch](interrupt-dispatch.md) | [HALT and EI](halt-ei.md), [The IF Register](if-register.md) |
| YPUK | Slot-0 X comparator bit 7 (XOR vs NOT(PX)) | xor | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| YRUM |  |  | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md) |
| YSES |  |  | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md) |
| YSEX |  |  | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md) |
| YVEL |  |  | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md) |
| YZAB |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| YZOS |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| ZACW |  |  | [Interrupt Dispatch](interrupt-dispatch.md) |  |
| ZAGO |  |  | [Mode 2: OAM Scan](oam-scan.md) |  |
| ZAHA | Slot-0 X comparator bit 2 (XOR vs NOT(PX)) | xor | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| ZAIJ |  |  | [Interrupt Dispatch](interrupt-dispatch.md) | [HALT and EI](halt-ei.md) |
| ZAKO | Slot-0 match, low-nibble collapse | nor4 | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| ZAOC |  |  | [Interrupt Dispatch](interrupt-dispatch.md) |  |
| ZAXE |  |  | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md) |
| ZAXY | Buffer stage | not_x2 | [The Clock Tree](clock-tree.md) |  |
| ZEBA | Slot-0 X comparator bit 1 (XOR vs NOT(PX)) | xor | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| ZECA |  |  | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md), [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |
| ZEME | Buffer stage | not_x4 | [The Clock Tree](clock-tree.md) |  |
| ZEZY |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| ZFEX |  |  | [Interrupt Dispatch](interrupt-dispatch.md) |  |
| ZIVV |  |  | [HALT and EI](halt-ei.md) |  |
| ZJJE | `ime_pending` — "IME pending" SR latch (q drives dmg-sim wire `ime_state`) | sm83_srlatch_r_n | [HALT and EI](halt-ei.md) |  |
| ZKOG |  |  | [Interrupt Dispatch](interrupt-dispatch.md) |  |
| ZLOZ |  |  | [Interrupt Dispatch](interrupt-dispatch.md) |  |
| ZOGY | Slot-0 X comparator bit 0 (XOR vs NOT(PX)) | xor | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| ZOKY | Slot-0 X comparator bit 3 (XOR vs NOT(PX)) | xor | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| ZOLA | Slot-0 X store bit 2 (holds NOT(X)) | drlatch_ee | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| ZRSY | RETI / reset gating SR latch (q = `zrsy`) | sm83_srlatch_r_n | [HALT and EI](halt-ei.md) |  |
| ZUCA |  |  | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md) |
| ZULU | Slot-0 X store bit 3 (holds NOT(X)) | drlatch_ee | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| ZUVE |  |  | [OAM DMA](dma.md) | [Mode 2: OAM Scan](oam-scan.md) |
| ZYTY |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| ZYVE |  |  | [Mode 3: The Sprite Pipeline](sprite-pipeline.md) |  |
| ZZOM |  |  | [Interrupt Dispatch](interrupt-dispatch.md) |  |
