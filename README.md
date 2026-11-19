# SpideyPS1recomp

Research and development toward a native Linux recompilation of **Spider-Man (2000) for the original PlayStation**, with a focus on understanding the game's camera and controller pipeline without replacing original game behavior unnecessarily.

This repository is a public engineering notebook: verified findings, reproducible diagnostics, open questions, and collaboration requests. It is not a distribution of the game.

## Project goals

- Build and run the recompilation natively on Linux.
- Preserve the original **L1 + L3** camera behavior, including Web Target Mode.
- Investigate an independent right-stick (R3) camera orbit:
  - horizontal R3 controls azimuth;
  - vertical R3 controls elevation;
  - the camera continues to track Spider-Man;
  - the orbit persists when R3 returns to neutral.
- Keep experimental camera code separate from original MIPS-recompiled routines.
- Measure stability and performance against controlled baselines.

The current focus is reverse-engineering the PS1 camera/rendering data flow. An orbital controller is **not yet validated**.

## Current technical status

The project has reached native recompilation and runtime testing on Linux. Recent investigation has identified several camera-related routines, matrix writes, controller input paths, and a global pointer that initially appeared relevant to a camera call. Further analysis showed that the pointer is updated by a routine that alternates between two structures and then calls the PS1 ordering-table clear routine. Its exact relationship to the matrix consumer is still under investigation; it must not be mislabeled as a camera pointer.

See [Technical status and investigation notes](docs/TECHNICAL_STATUS.md) for evidence, addresses, caveats, and next steps.

## Target environment used for testing

- Ubuntu 26.04.1 LTS, x86_64
- Intel Celeron laptop
- Intel UHD Graphics (Tiger Lake-LP GT2)
- 4 GB RAM
- Mesa 26.2.3

These are the current maintainer's test conditions, not minimum system requirements.

## Collaboration

Reviewers with experience in PS1 MIPS, PsyQ, GTE, GPU ordering tables, recompilation frameworks, and fixed-point camera math are very welcome.

Useful contributions include:
- independent interpretation of the documented MIPS routines;
- cross-references for the listed addresses;
- validation of GTE matrix semantics and data flow;
- reproducible Linux build/runtime reports;
- corrections to the evidence classification.

Please distinguish **observed facts** from **hypotheses** and include the exact game revision, build, scene, and test conditions where possible. See [CONTRIBUTING.md](CONTRIBUTING.md).

## Legal and asset policy

This repository will contain research notes, original diagnostic code, and documentation only. Do not upload game ISOs, ROM images, BIOS files, proprietary game assets, or copyrighted executables. Anyone testing the project must provide their own legally obtained game files.

## Status

**Research in progress — camera orbit not yet implemented/validated.**
