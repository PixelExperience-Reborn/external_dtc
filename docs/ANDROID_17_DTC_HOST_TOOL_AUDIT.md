# Android 17 P2: Device Tree Compiler source and host-tool ownership

**Date:** 2026-11-04. **Audit:** source comparison, not an Android17 compile or device test.

## Exact source boundaries

* Archive PE branch `seventeen`: `e5ca104ae3a9dce9fcd5cd18ee23e4a23ef337a7` (last change October 2023). Its `README.version` still identifies a 1.4.2-based upstream lineage.
* Official AOSP17 baseline: `platform/external/dtc`, tag `android-17.0.0_r1`, commit `38031c62ca17368fa8a2358ce562782f63402d15`, tree `5f9605429dc73bffb1c8273f3bbbbf279abc60fe`: https://android.googlesource.com/platform/external/dtc/+/refs/tags/android-17.0.0_r1
* The AOSP17 reference manifest already checks out **`platform/external/dtc`** at `external/dtc`.

## PE-specific engineering worth retaining in history

PE commit [`e5ca104`](https://github.com/PixelExperience-Reborn/external_dtc/commit/e5ca104ae3a9dce9fcd5cd18ee23e4a23ef337a7) added two host binaries for an earlier Qualcomm DTB merge script:

| Historical module | AOSP17 official `Android.bp` | Decision |
| --- | --- | --- |
| `fdtput` | `fdtput.c` exists, but no `cc_binary_host { name: "fdtput" }` in the official AOSP17 root blueprint. | Track the actual Android17 build tool consumer first; if required, add an isolated host module using current defaults and test it. |
| `fdtoverlaymerge` | Neither `fdtoverlaymerge.c` nor a Soong module is present in the official AOSP17 checkout root. | **Not drop-in portable.** Historical source invokes the nonstandard `fdt_overlay_merge` API and uses an older libfdt fork. Confirm whether Android17 uses this behavior, then separately port/review library semantics and tests. |

Official Android17 already builds `dtc`, `dtc_host_static`, `fdtget`, `fdtoverlay`, `fdtdump`, and a `dtc_static` test target. The new `dtc_version_gen.h` generator and default flags differ from the 2023 PE build file; **do not replace** it with historical `Android.bp`.

The PE `libfdt` tree also contains additional overlay helper fixes. Their ABI, input validation, phandle update and error-handling behavior require focused tests before any carry-over. Neither a historical commit message nor presence of `fdtoverlaymerge` establishes a consumer in the staged Android17 build.

## Accepted source-owner decision

Keep **official AOSP17 DTC/libfdt** selected, with the PE fork archived as a reference. Do not add legacy `fdtoverlaymerge` to the active manifest or vendor build by default. Do not remove its historical code. If a *current* Qualcomm build dependency is found, first capture its script invocation and expected DTBO semantics, then implement an independent reviewed adaptation and verify with known input/output DTS/DTBO fixtures.

**Missing tests:** effective Android17 DTBO merge, libfdt Soong compile, Qualcomm kernel packaging and physical device boot. Scope is platform-source ownership only.
