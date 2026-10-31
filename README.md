# HakiOS MT6785 — SALAA and Nemo

An experimental HakiOS port for the Realme 7 (RMX2151, SALAA; Helio G95) and Realme 6 (RMX2001,
Nemo; Helio G90T). It reuses HakiOS's MediaTek/XNU integration while maintaining separate board
profiles.

**This is source bring-up work, not a bootable release.** The MT6785 EFI platform table and maximum
CPU/read-only UFS configuration are in place, but neither phone has been booted with HakiOS. No
local build or hardware run was performed, as requested. Do not flash donor MT6768 images or treat
these profiles as validated.

## Status

| Area | SALAA | Nemo |
|---|---|---|
| SoC | MT6785, Helio G95 | MT6785, Helio G90T |
| CPU | 6+2 MPIDR map; profile requests 7 CPUs and boot core 7; PSCI startup unverified | Same shared SoC profile |
| Display | 1080×2400 Truly DSI reference; EFI loader uses GOP if firmware provides it | 1080×2400 Novatek panel reference; EFI loader uses GOP if available |
| Touch | Novatek NF NT36672C over SPI3 in SALAA stock DT; no HakiOS NVT protocol driver | Nemo reference selects Novatek NVT SPI; exact module variant and HakiOS driver unverified |
| Keys | No MT6785 PMIC/key driver | No MT6785 PMIC/key driver |
| Storage | UFS read-only service/root requested; firmware Block I/O and `xnusys` partition are prerequisites | Same shared UFS path; firmware handoff not verified |
| GPU | Mali-G76 MC4 identified; no MT6785 power/clock or G76 driver | Same |
| Wi-Fi / Bluetooth | MT6785 CONNSYS and firmware path not ported | Same |
| Sensors | SCP driver not adapted | SCP driver not adapted |
| Audio / camera / battery / haptics | Unsupported; not in donor's working MT6768 feature set | Same |
| Boot/recovery | No tested UEFI setup, HakiOS image, flash recipe, or recovery recipe | Same |

Detailed device notes are in [`devices/salaa`](devices/salaa) and [`devices/nemo`](devices/nemo).
Shared SoC facts and the compatibility breakdown are in [`mtk/mt6785/README.md`](mtk/mt6785/README.md).

## How MT6768 helps

The Infinix Hot 20 MT6768 port is a valuable reference for the HakiOS boot arguments, XNU handoff,
read-only block-service protocol, kernel/userland wiring, diagnostics, and Mali Bifrost software
interfaces. Helio G85, G90T and G95 share MediaTek design patterns, but MT6785 changes the board
register maps, storage controller, CPU cluster affinity, GPU generation/configuration, power
domains, touch controller and firmware details. Donor sequences are reused only where the shared
interface is actually applicable.

The separate [`hakios-metal`](https://github.com/oosexgang/hakios-metal) project targets Adreno/MSM
and does not provide a Mali-G76 backend.

## What this tree configures

- `xnuloader` has a compile-time MT6785 platform description: GICv3 address range, 6+2 CPU map,
  UFSHCI base `0x11270000`, and direct use of a firmware GOP framebuffer. It uses PSCI to start
  additional cores.
- `mtk/mt6785/configs/mt6785.config` requests seven XNU CPUs (`bootcpu=7`), the UEFI UFS block
  service (`ufs=2`), and read-only `xnusys` root selection (`ufs_root=1`). The `UFS_WRITE` and
  persistent-write paths are not enabled.
- `make loader DEVICE=salaa` and `make loader DEVICE=nemo` select the MT6785 EFI platform. `make
  esp` creates a UEFI volume containing the EFI loader and payload. `make boot-image`, `make phone`,
  and `make system-flash` are not MT6785 install paths.

These are requested capabilities, not proven results. UFS root requires firmware exposing UFS
Block I/O and a correctly provisioned `xnusys` GPT partition. The kernel does not initialize the
MT6785 UFS PHY/controller itself. The loader passes the MT6785 UFSHCI address to the existing
checked read-only path when native controller handoff is available.

The linked [SALAA UEFI project](https://github.com/iSciFiLite/rmx2151-salaa-uefi) is an experimental
EDK II package, not a ready HakiOS firmware. Its current platform DSC does not include UFS DXE or
GOP/display drivers, so it does not currently meet the HakiOS loader prerequisites. Nemo has no
UEFI platform profile here. A usable device boot path still needs firmware integration and hardware
validation.

## Source and compatibility

- Kernel base: [`oosexgang/hakios-xnu-kernel`, `mtk` branch](https://github.com/oosexgang/hakios-xnu-kernel/tree/mtk), snapshot `db08f63ca513b711d562e4bd20d41fd9ab559f4e`.
- Userland: source revision `7caa4576e6e1be6d63f963b106b73480d6539a9b`; the former [standalone repository](https://github.com/oosexgang/hackios-xnu-userland) points to its replacement.
- Donor device/reference: [`oosexgang/hakios-mt6768`](https://github.com/oosexgang/hakios-mt6768), specific to X6826B/MT6768, eMMC, 720×1640 display, Mali-G52 and its SPI TCM touchscreen.
- SALAA UEFI reference: [`iSciFiLite/rmx2151-salaa-uefi`](https://github.com/iSciFiLite/rmx2151-salaa-uefi).
- Nemo kernel reference: [`iSciFiLite/android_kernel_realme_nemo`](https://github.com/iSciFiLite/android_kernel_realme_nemo).

Vendor device-tree references informed hardware addresses and board variants; they are not copied
into this repository. No proprietary firmware, Darwin image, key material, or generated binary is
included.

## Repository map

```text
mtk/mt6785/       shared MT6785 facts and default profile
mtk/mt6768/       donor MT6768 drivers retained as porting references
mtk/loader/       donor boot_a loader, still X6826B/MT6768-specific
devices/salaa/    Realme 7 SALAA profile
devices/nemo/     Realme 6 Nemo profile
xnuloader/        UEFI loader with QEMU, Redwood, and MT6785 platform descriptions
drivers/          shared XNU drivers, including checked read-only UFS support
userland/         pinned source-only HakiOS userland
xnu/              XNU patch and HakiOS overlay sources
```

## Configuration and development

The profiles aim for maximum currently implemented bring-up: multi-core enumeration plus
read-only UFS service/root. Unavailable GPU, touch, PMIC, SCP and CONNSYS hardware drivers are not
pretended into existence by enabling unrelated Qualcomm or MT6768 options. Turning on an absent
driver would either leave a stub in place or run unverified register code; it would not make that
feature work. As each MT6785 driver is ported, add it once to the shared SoC layer when hardware
matches both devices, or to the specific device profile when it does not.

Configuration checks:

```sh
tools/hakios-device.py check salaa nemo
tools/hakios-config.py defconfig salaa
# or: tools/hakios-config.py defconfig nemo
tools/hakios-config.py show
```

Building is intentionally not documented as a validated install procedure. For future developers,
the source build is `xnu/apply.sh`, `xnu-deps/setup.sh`, then `make DEVICE=salaa kernel` (or
`DEVICE=nemo kernel`). A kernel build does not prove bootability. `make loader DEVICE=salaa` builds
the EFI app for the MT6785 platform; the EFI app still needs a UEFI environment with the required
filesystem, UFS Block I/O, PSCI and preferably GOP services. A UEFI ESP can be assembled with
`make esp DEVICE=salaa PAYLOAD=...` once a payload exists.

The UFS profile expects a read-only `xnusys` partition. No repartitioning or flashing utility is
provided. Keep stock boot, vendor_boot, vbmeta and recovery images before any future device test.
This source tree has no validated install or recovery recipe. Obtain proprietary firmware only from
a device/firmware build you are authorized to use; keep extracted files local and ignored, and
record their source and license before adding a driver dependency.

## Feature roadmap

1. Pair the EFI loader with UEFI firmware that offers UFS Block I/O, GOP and working PSCI; validate
   one-core boot/recovery first, then the requested seven-core map.
2. Verify the read-only UFS service and GPT geometry; exercise `xnusys` root before any write path.
3. Add Novatek NVT SPI input and board-specific keys for the exact SALAA/Nemo revisions.
4. Port Mali-G76 power domains/clocks/MMU using MT6785 references; do not use Adreno Metal code.
5. Adapt SCP sensors and CONNSYS Wi-Fi/Bluetooth with matching firmware. Audio and camera remain
   outside the donor's proven feature set.

Boot, storage, display, touch, GPU and other peripherals remain unverified. Update this README only
after logs and device-specific test evidence are available.

## Licensing and notices

Preserve [`LICENSE`](LICENSE), [`LICENSE.APSL`](LICENSE.APSL), [`NOTICE`](NOTICE),
[`THIRD_PARTY_LICENSES.md`](THIRD_PARTY_LICENSES.md), and source-file notices. XNU patch, overlay
and driver code follows its file-level/APSL terms; project loader, tools, documentation and
userland retain their stated licenses. No stock device tree, phone firmware, Darwin image, or
generated binary is distributed here.
