# Appendix E: Sources

## Primary sources (ground truth)

- **[msinger/dmg-schematics](https://github.com/msinger/dmg-schematics)**
  (msinger and [rgalland](https://github.com/rgalland)) — transistor-level
  reverse engineering of the DMG-CPU B die. The origin of every gate name in
  this book.
- **[msinger/dmg-sim](https://github.com/msinger/dmg-sim)** — SystemVerilog
  simulation generated from the schematics, run under Icarus Verilog. The
  source of every "dmg-sim measurement" citation.
- **Static netlist analysis**
  ([gb-propagation-delay-analysis](https://github.com/ajoneil/gb-propagation-delay-analysis))
  — graph analysis over the netlist: combinational depths (ge), race
  pairs, clock domains, critical paths.

## Hardware test ROMs (hardware-verified expectations)

Test ROMs whose expected values are validated on real hardware corroborate
behavioural conclusions throughout:

- [gbmicrotest](https://github.com/aappleby/gbmicrotest) (and GateBoy, its
  die-photo-derived sibling simulator)
- [Mooneye](https://github.com/Gekkio/mooneye-test-suite) and the
  wilbertpol fork
- the gambatte hardware-test suite
- [SameSuite](https://github.com/LIJI32/SameSuite)
- [AGE test ROMs](https://github.com/c-sp/age-test-roms)
- [blargg's test ROMs](https://github.com/retrio/gb-test-roms)
- Hacktix's test ROMs (`strikethrough.gb`)
- Mealybug Tearoom's mid-Mode-3 write tests, whose DMG reference
  photographs anchor the below-netlist LCD overlay rules

## Behavioural documentation

- **[gb-ctr](https://gekkio.fi/files/gb-docs/gbctr.pdf)** (Gekkio's *Game
  Boy: Complete Technical Reference*) — the behavioural reference this
  book deliberately does not duplicate; cited wherever a chapter condenses
  well-covered ground.
- **[Pan Docs](https://gbdev.io/pandocs/)** — community reference; several
  of its narrative rules are given gate-level form (and in two cases
  corrected) in this book.
- **TCAGBD** and the **Mealybug Tearoom PPU notes** — cross-references for
  high-level framing.
- **`dmgcpu` wiki pad documentation** — pin-role descriptions for the LCD
  interface (explicitly marked as not netlist-derivable where used).

## Behavioural emulators

No emulator was used as a source of hardware truth, and none is cited in the
text. The gambatte references throughout are to its hardware-test ROMs
(above), not to the emulator's behaviour.
