# Technical status — Spider-Man (PS1)

Last updated: 2026-11-19

This document is a concise public snapshot of the ongoing investigation. The longer Portuguese master diary remains the detailed continuity record. Evidence labels are intentional:

- **CONFIRMED** — directly observed in code, disassembly, logs, execution, or identified primary documentation.
- **PARTIAL** — concrete evidence exists, but the full interpretation or data flow is not established.
- **HYPOTHESIS** — plausible explanation that still requires proof.
- **OPEN** — unresolved question.
- **NEGATIVE** — a bounded search/test did not find the target; this does not prove it does not exist elsewhere.

Do not combine values from different runs or scenes.

## 1. Functional target

The intended feature is an independent R3-controlled orbit while preserving the game's original controls:

1. L1 + L3 must remain original, including Web Target Mode.
2. R3 must work without L1.
3. Horizontal R3 controls orbit azimuth; vertical R3 controls elevation.
4. The camera should continue tracking Spider-Man at approximately constant distance.
5. Releasing R3 should preserve the selected orbit.
6. No original MIPS routines or button mappings should be replaced without evidence and explicit review.

No experimental orbit controller is currently considered validated.

## 2. Confirmed input and camera-path findings

- **CONFIRMED:** the right analog values reach the runtime through `Controller.RightX/RightY`, `LibPad.Refresh`, and `PadPatches.Buf1` (offsets +4 and +5).
- **CONFIRMED:** the Spider-Man gameplay callback observed in the investigation is `0x80071378`; its central camera-related routine is `func_800701F0` at `0x800701F0`.
- **CONFIRMED:** `func_800701F0` wraps `func_800701F0_Impl` with camera trace hooks.
- **PARTIAL:** `func_800701F0_Impl` updates the global camera position at `0x800A575C` and works with a transform destination at `0x800A5734`.
- **CONFIRMED:** the observed player pointer at `0x800B5268` pointed to `0x8011AF8C`; player coordinates were read at player +4, +8, +0xC in Q20.12 in the recorded gameplay traces.
- **PARTIAL:** the region `0x800A57CC–0x800A57DD` receives nine halfword components from MIPS stores at PCs `0x800761D8`, `0x80076204`, and `0x80076214`. GTE consumers were observed, but exclusive ownership by the camera has not been proven.
- **CONFIRMED:** the R3 orbit feature has not passed the full acceptance criteria. Prior experiments were inconsistent and sometimes reduced performance or lost the player framing.

## 3. Matrix writer investigation

A diagnostic capture previously recorded three MIPS `SH` instructions writing nine halfwords in the region `0x800A57CC–0x800A57DD`.

The recompilation source maps those stores to a loop inside `func_80075D0C`. The loop performs fixed-point multiplication and stores components into three destinations. GTE operations occur before the loop.

**Interpretation limit:** this pattern is compatible with a matrix transformation, but it does not establish that the matrix is exclusively the camera orientation. It may be used by a broader rendering path.

Previously recovered call information:
- one caller/call site was recorded at PC `0x8002BE0C`;
- A0 is preserved in S5;
- A1 is preserved in S4;
- A2 is preserved and saved at GP+0x1150.

The complete cross-reference and all call contexts still need to be reconciled. Do not assume `0x8002BE0C` is the only caller.

## 4. New finding — global pointer at 0x800B54A8 (2026-11-19)

A temporary diagnostic was enabled with `SPIDEY_TRACE_POINTER=1` while the player moved using L3, R3 was centered, and L1 was not intentionally pressed.

**CONFIRMED from the captured write trace:**
- RAM corresponding to virtual address `0x800B54A8` is written by MIPS PC `0x800612D8`.
- The instruction is `SW`.
- The stored pointer alternates between `0x8009A6E4` and `0x8009A75C`.
- The two addresses differ by `0x78` (120 bytes).

The corresponding recompilation routine is `func_800612B8`:

```c
V0 = 0x800A0000;
V1 = *(GP + 0xCB4);
BASE = V0 - 0x591C;          // 0x8009A6E4
if (V1 != BASE)
    SELECTED = BASE;
else
    SELECTED = BASE + 0x78;  // 0x8009A75C
*(GP + 0xCB4) = SELECTED;
A0 = *(SELECTED + 0x70);
A1 = 0x1000;
ClearOTagR(A0, A1);
```

The actual generated C# should be consulted for exact branch semantics; the arithmetic yields the two observed structure addresses:
- `0x800A0000 - 0x591C = 0x8009A6E4`
- `0x8009A6E4 + 0x78 = 0x8009A75C`

**CONFIRMED:** this routine selects/alternates between two structures, updates the global at `0x800B54A8`, and then calls `ClearOTagR`.

**LIKELY, NOT YET FULLY PROVEN:** this is part of a double-buffered rendering / ordering-table path. `ClearOTagR` is the PS1 ordering-table clear operation, so the pointer must not be described as a camera pointer merely because another routine dereferences it while preparing arguments for `func_80075D0C`.

The generated source contains:
- call site: `SpiderMan.ClearOTagR(c, m)` around line 187688;
- wrapper: `public static void ClearOTagR(CpuContext c, IMemory m)` around line 251354, forwarding to `Recompiled.GpuPatches.ClearOTagR(c, m)`.

A repository-wide source search for the implementation is still pending. A search limited to `tools/RecompOne` returned no output, which only means the implementation was not found in that search scope.

## 5. Important instrumentation caveats

- In `CameraInputTrace`, the L1 bit for that controller layout is `0x0010`, not `0x0400`. A trace using the old mask can report a false negative.
- `PadTrace` uses a different layout where `0x0400` is valid. Do not mix these masks.
- Some `objectPos/objectRot` fields in older camera traces were read from incorrect offsets and are not reliable.
- `instruction-write` and `updater-matrix-write` are independent mechanisms/counters.
- Logs from different runs/scenes/frames must not be merged into a single geometric calculation.
- A prior experimental `CameraOrbitController` used unvalidated assumptions (including a radius of 600); do not treat those values as established game constants.

## 6. External references and limits

- `krystalgamer/spidey-decomp` is a decompilation of the PC port, not proven equivalent source for the PS1 executable. Its camera/math code is comparative reference only.
- PsyQ documentation describes `RotMatrixYXZ` as `RY(r1) · RX(r0) · RZ(r2)`, with 4096 units per full turn and Q12 matrix conventions. This documents the API, not necessarily every game-specific call path.
- No external project has yet been shown to provide a drop-in, validated PS1 camera-orbit implementation for this project.

## 7. Next research steps

1. Locate the actual implementation of `GpuPatches.ClearOTagR` and document its memory effects.
2. Trace the structures at `0x8009A6E4` and `0x8009A75C`, especially the pointer stored from offset +0x70.
3. Determine exactly why `func_80075D0C` consumes the dereferenced global pointer and enumerate all its call sites.
4. Cross-reference all readers/writers of `0x800A57CC–0x800A57DD` to separate camera, rendering, shadow, and other uses.
5. Correlate player position, camera position, GTE matrix inputs/outputs, frame, and callback in one controlled run.
6. Validate axes, angle conventions, and matrix layout before implementing any R3 camera writes.
7. Only then design and test an isolated orbit controller.

## 8. Test environment

Maintainer's current test laptop:
- Ubuntu 26.04.1 LTS, x86_64
- Intel Celeron CPU
- Intel UHD Graphics (Tiger Lake-LP GT2)
- 4 GB RAM
- Mesa 26.2.3

This describes the test machine, not a minimum requirement.
