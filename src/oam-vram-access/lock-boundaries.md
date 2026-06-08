# Lock Boundaries

Away from mode boundaries both enable chains are stable, so a CPU write
sees a fully-open or fully-closed window — never a partial one. Outcomes
flip exactly one M-cycle apart across a lock transition (dmg-sim
measurement, gbmicrotest `oam_write_l0_*` / `vram_write_l[01]_*`
families).

The exception is a write whose strobe straddles a boundary transition.
On the OAM side all three boundaries are characterised in
[CPU-visible mode boundaries](../cpu-visible-boundaries.md): a Mode 2→3
straddle lands through the ~6.5 ns permit gap, a Mode 3→0 straddle lands
the moment the permit opens, and Mode 0→2 closes cleanly — the write is
blocked. The VRAM-side straddle reaches the off-chip SRAM as a partial
cycle, outside the netlist's scope (above; [Appendix C](../open-questions.md)).
