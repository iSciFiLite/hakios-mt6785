# Device repositories

`devices/manifest.conf` names every device. QEMU, SALAA and Nemo profiles are kept in this tree:
QEMU is the runnable virtual reference; SALAA and Nemo are conservative MT6785 port profiles whose
boot and hardware support are not yet validated. Other device repositories may be cloned with
`tools/hakios-device.py sync`. A device of your own: `tools/hakios-device.py new`
([Port a device](../docs/porting/index.md)).
