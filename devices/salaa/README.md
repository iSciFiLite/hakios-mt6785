# Realme 7 — SALAA (RMX2151)

This is the device profile for the MediaTek MT6785 HakiOS port. It deliberately selects the
single-CPU, no-phone-firmware baseline. It does not yet provide a bootable loader or working
peripheral drivers.

## Hardware references

The repository's supplied SALAA UEFI source identifies the RMX2151 and MT6785. The local stock
device-tree references report a 1080×2400 Truly display and contain Goodix and Novatek touch
bindings in different trees/overlays. The exact panel/touch pairing must be read from the target
phone before enabling a driver. The shared MT6785 facts are in [`mtk/mt6785`](../../mtk/mt6785/README.md).

## Current state

See `device.conf` for machine-readable status. The donor MTK loader assumes an X6826B eMMC, fixed
720×1640 framebuffer and Mali-G52. It cannot be used on SALAA as-is. The SALAA UEFI project is a
reference for firmware packaging and MT6785 initialization; its current SEC-only port is not a
drop-in HakiOS XNU loader and does not provide the HakiOS UFS page.

No firmware, generated image, or stock DT is included here. Do not flash a build from this profile;
there is no supported boot or recovery image yet. Keep the original boot, vendor_boot and recovery
images available for restoration using the device's established Realme/MediaTek recovery method.
