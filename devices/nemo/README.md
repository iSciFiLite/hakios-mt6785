# Realme 6 — Nemo (RMX2001)

This is the device profile for the MediaTek MT6785 HakiOS port. It deliberately selects the
single-CPU, no-phone-firmware baseline. It does not yet provide a bootable loader or working
peripheral drivers.

## Hardware references

The local Realme 6 port reference identifies RMX2001/Nemo and MT6785. Its board DTS overlay reports
a 1080×2400 Truly display and a Novatek NT36672C touchscreen on SPI3. These are identification
references only; this project has not validated the framebuffer or touch handoff. The shared
MT6785 facts are in [`mtk/mt6785`](../../mtk/mt6785/README.md).

## Current state

See `device.conf` for machine-readable status. The donor MTK loader assumes an X6826B eMMC, fixed
720×1640 framebuffer and Mali-G52. It cannot be used on Nemo as-is. A separate Linux port boots to
initramfs/USB networking, but its display is still unported; that work is not an XNU boot path.

No firmware, generated image, or stock DT is included here. Do not flash a build from this profile;
there is no supported boot or recovery image yet. Keep the original boot, vendor_boot and recovery
images available for restoration using the device's established Realme/MediaTek recovery method.
