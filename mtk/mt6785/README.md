# MediaTek MT6785 shared layer

This layer records SoC-wide facts for the Realme 7 (SALAA/RMX2151) and Realme 6 (Nemo/RMX2001).
The current configuration is intentionally conservative: it records the eight-core topology and
leaves all MTK phone drivers disabled until their register sequences and boot handoff are ported.

## Hardware contract gathered from local references

| Block | Reference |
|---|---|
| CPU | 6 Cortex-A55 + 2 Cortex-A76; one-CPU baseline until the MTK loader's CPU map is ported |
| GPU | Mali-G76 MC4 / Bifrost, register base `0x13040000`, IRQs 309–313 |
| UFSHCI | `0x11270000`, register region `0x2300`; UFS M-PHY at `0x11fa0000` |
| Display | MediaTek DSI/OVL path; panel and framebuffer allocation are device-specific |
| SALAA panel | Stock references report a 1080×2400 Truly TD4330 variant |
| Nemo panel | Local Android DTS overlay reports 1080×2400 Truly TD4330 |
| Nemo touch | Local Android DTS overlay names Novatek NT36672C on SPI3 |

Sources are the local `rui2-nemo-kernel` MT6785 DTS files and the supplied SALAA stock-DT/UEFI
reference. Those files are not copied into this repository. SALAA touch bindings vary across the
available stock trees; the profile does not choose or enable a controller.

## Why the donor drivers stay off

`mtk/mt6768/` is not a generic MTK implementation: its Mali driver is for G52, the touch driver is
for an SPI TCM controller, and its loader reads eMMC and assumes the X6826B panel/framebuffer.
MT6785 uses UFS and G76. The kernel's generic UFS driver can consume a checked loader handoff page,
and this port adds a `ufs_root=1` selection hook for `xnusys`; the MTK loader still needs to
produce that page and preserve the active UFS controller state. Until that exists, UFS root remains
disabled in both profiles.

The independent `hakios-metal` backend targets Adreno/MSM DRM. It is not part of this layer. Mali
support needs a separate G76 power-domain, clock, MMU and userland implementation.
