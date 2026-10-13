# Project-KARA Kara-KSU rebased onto LineageOS onclite

**Base:** [LineageOS/android_kernel_xiaomi_onclite](https://github.com/LineageOS/android_kernel_xiaomi_onclite) `lineage-22.2` (`8aab452e4`, Linux 4.9.337)  
**Source:** [gen69x/Project-KARA](https://github.com/gen69x/Project-KARA) branch `Kara-KSU` (`5a86f9bb6`)  
**Device:** Xiaomi Redmi 7 / Y3 (`onclite` / `onc`)  
**Branch on this tree:** `kara-ksu-los`

`lineage-22.2` and `lineage-21` of the LineageOS kernel are identical.

## What this rebase is

Kara-KSU and LineageOS do **not** share a linear git history (fork of PapiLer/Project-KARA vs Lineage CAF). A literal `git rebase` of 700k+ commits is not viable.

This tree takes **LineageOS as the new upstream** (newer CAF, WLAN, camera, SELinux, DRM) and ports Kara-KSU's unique features on top.

The original `drivers/staging/ksu` in Kara-KSU was incomplete (Makefile referenced objects that were not in the tree). KernelSU is therefore imported from **[backslashxx/KernelSU](https://github.com/backslashxx/KernelSU) v3.3.0+**, the same driver family Kara-KSU already hardcoded (`KSU_VERSION=11563` → now `32663`).

## Commits on top of LOS

| Commit | What |
|---|---|
| `kernelsu: import backslashxx KernelSU v3.3.0+ for 4.9` | In-tree `drivers/kernelsu`, expose `policy_rwlock` |
| `selinux: allow init to exec ksud under nosuid` | Original Kara-KSU F-19-F hook |
| `misc: port FORCE_FAST_CHARGE` | sysfs + smb-lib 500→900 mA SDP |
| `fs: port sdFAT` | Samsung sdFAT as exFAT |
| `ARM64: onclite: enable Kara-KSU features` | defconfig |
| `kara: port Kara-KSU performance and power patches` | 39/48 unique patches that still apply |

## Defconfig (`onclite-perf_defconfig`)

- `CONFIG_LOCALVERSION="-Kara-KSU"`
- `CONFIG_KSU=y`
- `CONFIG_KSU_TAMPER_SYSCALL_TABLE=y` (recommended by backslashxx for 3.0–4.14)
- `CONFIG_KSU_LSM_SECURITY_HOOKS=y`
- `CONFIG_FORCE_FAST_CHARGE=y`
- `CONFIG_SDFAT_FS=y` (+ USE_FOR_EXFAT)
- `CONFIG_WIREGUARD=y`
- `CONFIG_KPROBES=y` / `CONFIG_KRETPROBES=y`
- Cypress CYTTSP5 (onclite panel variant)

LOS security defaults are kept (stack protector, UTS_NS, CAF extras). Kara's 4.14 RCU rewrite was **not** overlaid.

## Performance patches that did not apply

LOS `schedutil` / kgsl A6xx / some idle code already diverged:

- schedutil iowait boost series (Kara-KSU itself **reverted** the last of these)
- empty lpm-levels / camera qos patches
- regmap Makefile
- A6xx CP context size (onclite GPU is Adreno 506 / A5xx)

## Build

Same as LineageOS onclite + Proton Clang (what Kara_build_action used):

```bash
git clone -b lineage-22.2 https://github.com/LineageOS/android_kernel_xiaomi_onclite
cd android_kernel_xiaomi_onclite
git am /path/to/patches/*.patch

# or, if you have this tree:
# git checkout kara-ksu-los

make O=out ARCH=arm64 onclite-perf_defconfig
make O=out ARCH=arm64 \
  CROSS_COMPILE=aarch64-linux-gnu- \
  CROSS_COMPILE_ARM32=arm-linux-gnueabi- \
  CC=clang \
  -j$(nproc)
```

Pack `out/arch/arm64/boot/Image.gz-dtb` with AnyKernel3
([gen69x/AnyKernel3](https://github.com/gen69x/AnyKernel3)).

Manager: official KernelSU / backslashxx-compatible manager (signature
`EXPECTED_SIZE=0x033b`, hash in `drivers/kernelsu/Makefile`).

## Apply the patch series onto a fresh LOS clone

```bash
git clone --depth 1 -b lineage-22.2 \
  https://github.com/LineageOS/android_kernel_xiaomi_onclite
cd android_kernel_xiaomi_onclite
git am /path/to/kara-ksu-los-patches/*.patch
```
