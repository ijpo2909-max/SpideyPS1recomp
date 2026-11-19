# Contributing

Thanks for helping investigate the original PlayStation version of Spider-Man.

## What is useful

- MIPS disassembly review and call-graph/xref analysis.
- PS1 GTE and PsyQ matrix semantics.
- Ordering-table and double-buffering analysis.
- Reproducible native Linux build/runtime testing.
- Independent review of the camera/input data flow.

## Evidence standards

Please:
- label statements as confirmed, partial, hypothesis, open, or negative;
- include the exact executable revision/region where known;
- include addresses, instruction PCs, and relevant register values;
- identify the build, scene, frame range, and controls used for runtime captures;
- keep captures from different runs separate;
- explain what your test does **not** prove.

Avoid presenting PC-port code as authoritative PS1 behavior unless equivalence is demonstrated.

## Code and assets

Do not submit game ISOs, ROMs, BIOS files, proprietary assets, or copyrighted executables. Submit original analysis scripts, documentation, and code you have the right to distribute.

Before proposing a behavioral change, explain how it preserves the original L1 + L3 camera behavior and Web Target Mode.
