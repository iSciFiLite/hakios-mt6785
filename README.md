# HakiOS MT6785 — SALAA and Nemo

An experimental HakiOS port workspace for MediaTek MT6785 devices, initially targeting the Realme 7
(RMX2151, SALAA) and Realme 6 (RMX2001, Nemo). This repository is based on the HakiOS kernel's
`mtk` branch and keeps a shared MT6785 layer with separate device profiles.

**This is an incomplete source port, not a bootable release.** The SALAA and Nemo profiles are
conservative configuration scaffolds. Boot handoff, UFS loader integration, display setup and all
phone peripherals remain unverified or unavailable. No build was run for this port, and no image is
provided. Do not flash it.

## Status

| Area | SALAA | Nemo |
|---|---|---|
| SoC identification | MT6785, Helio G95 | MT6785, Helio G90T |
| CPU | Topology recorded; only one CPU selected in the baseline profile | Same |
| Display | 1080×2400 panel reference; framebuffer handoff unverified | 1080×2400 panel reference; framebuffer handoff unverified |
| Touch and keys | Not enabled; SALAA stock references show controller variants | Not enabled; reference names Novatek NT36672C |
| Storage | UFS detected in references; HakiOS loader handoff not implemented | Same |
| GPU | Mali-G76 identified; no G76 driver port | Same |
| Wi-Fi / Bluetooth | Unsupported | Unsupported |
| Audio / camera / battery / haptics | Unsupported | Unsupported |
| Boot and recovery | No supported HakiOS image or install procedure | No supported HakiOS image or install procedure |

Status is also recorded in [`devices/salaa/device.conf`](devices/salaa/device.conf) and
[`devices/nemo/device.conf`](devices/nemo/device.conf). The shared hardware notes are in
[`mtk/mt6785/README.md`](mtk/mt6785/README.md).

## Upstream and compatibility notes

- Kernel base: [`oosexgang/hakios-xnu-kernel`, `mtk` branch](https://github.com/oosexgang/hakios-xnu-kernel/tree/mtk), pinned source snapshot `db08f63ca513b711d562e4bd20d41fd9ab559f4e`.
- Userland: the current source revision from [`oosexgang/hakios-xnu-kernel/userland`](https://github.com/oosexgang/hakios-xnu-kernel/tree/main/userland), commit `7caa4576e6e1be6d63f963b106b73480d6539a9b`; the [former standalone repository](https://github.com/oosexgang/hackios-xnu-userland) points to its replacement.
- The donor's `mtk/loader/` and `mtk/mt6768/` are X6826B/MT6768-specific. They assume eMMC, a 720×1640 framebuffer and Mali-G52. This repo guards the Makefile so an MT6785 profile cannot accidentally use that loader or its flash scripts.
- The generic XNU UFSHCI path now takes its controller address from the loader handoff, and XNU has a read-only `ufs_root=1` hook for a loader-provided `xnusys` entry. `CONFIG_UFS_ROOT` requires the UFS driver and `CONFIG_UFS=2`. The MTK loader does not yet create the required checked `ufs_shared_pa` page or hand off UFSHCI state, so the feature stays disabled.
- [`oosexgang/hakios-metal`](https://github.com/oosexgang/hakios-metal) is an Adreno/MSM backend. It is not applicable to the Mali-G76 and is not included.

## Repository layout

```text
mtk/mt6785/       shared MT6785 facts and conservative SoC configuration
mtk/mt6768/       donor MT6768 implementation, retained for reference
mtk/loader/       donor loader, still limited to the X6826B/MT6768 path
devices/salaa/    Realme 7 SALAA profile
devices/nemo/     Realme 6 Nemo profile
drivers/          common HakiOS kernel drivers, including generic UFS code
userland/         source-only HakiOS userland at the pinned upstream revision
xnu/              XNU patch and HakiOS overlay sources
```

The profiles do not enable MT6768, Qualcomm or Adreno device drivers. They use a single-CPU,
firmware-free baseline and leave unported phone drivers out of the kernel configuration.

## Development setup

Use a Linux or macOS development environment with Git, Python 3, Clang/LLD, Make, Flex, Bison and
M4. XNU's pinned source and host tools are fetched by the repository scripts; Apple system software
and phone firmware are not included.

```sh
git clone https://github.com/iSciFiLite/hakios-mt6785.git
cd hakios-mt6785
tools/hakios-device.py check salaa nemo
tools/hakios-config.py defconfig salaa
# or: tools/hakios-config.py defconfig nemo
```

These commands check repository/configuration structure only. This repository has not been built.
The full XNU build path is `xnu/apply.sh`, `xnu-deps/setup.sh`, then `make DEVICE=salaa kernel` (or
`DEVICE=nemo`). Those steps compile code and are intentionally not part of the port review recorded
here. A successful compile would not establish that the phone boots.

The current `salaa` and `nemo` profile does not select a loader target. `make loader`, `make payload`,
`make boot-image`, `make phone` and `make system-flash` must remain unavailable until the MT6785
loader, UFS service page, board handoff and recovery path are implemented and reviewed. Do not use
MT6768 flashing instructions or images on these devices.

## Firmware, setup and recovery

The baseline has `CONFIG_FIRMWARE="none"`; it does not request proprietary phone firmware. If a
future touch, GPU, Wi-Fi, audio or sensor driver needs a vendor blob, obtain it only from a device
and firmware version you are authorized to use, keep it in a local ignored directory, and document
its origin and license before enabling that feature. Do not commit extracted firmware, Darwin
images, keys, certificates, or generated binaries.

There is currently no HakiOS setup or boot procedure for SALAA or Nemo. No partition changes,
boot-image replacement, or flashing steps are supported by this source tree. Keep verified stock
boot, vendor_boot, vbmeta and recovery images before experimenting. If recovery is needed, restore
the device's stock images through the established Realme/MediaTek recovery procedure for that exact
model and firmware release; this project does not yet provide a validated recovery recipe.

## Porting order

1. Implement an MT6785 loader/UEFI handoff and validate a one-core XNU console boot without writing
   partitions.
2. Have the loader publish and validate the UFS page, controller state and `xnusys` GPT geometry;
   then exercise `ufs_root=1` read-only.
3. Add separate SALAA and Nemo framebuffer/display and touch/key drivers from their exact board
   revisions.
4. Port Mali-G76 power domains, clocks and MMU safely; keep the Adreno backend disabled.
5. Add MT6785 sensor and CONNSYS work only after firmware provenance and hardware behavior are
   understood. Audio and camera are not in the donor MT6785 feature set and remain unsupported.

Boot, storage, display and peripheral operation are unverified. The donor's MT6768 feature list is a
porting target only, not a claim of parity.

## Licensing and notices

Preserve the repository's [`LICENSE`](LICENSE), [`LICENSE.APSL`](LICENSE.APSL), [`NOTICE`](NOTICE),
[`THIRD_PARTY_LICENSES.md`](THIRD_PARTY_LICENSES.md), and file-level upstream notices. XNU patch,
overlay and driver code are under APSL-2.0; project-authored loader, tools, documentation and
userland follow the licenses stated in each file and the top-level license. No stock device tree,
phone firmware, Darwin image or generated binary is distributed here.
