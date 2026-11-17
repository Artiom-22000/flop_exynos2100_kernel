# O1S SN100U NFC validation notes

This note records the kernel-side NFC configuration for the Galaxy S21
Exynos 2100 (SM-G991B, device codename `o1s`) and the limits of what this
kernel change provides.

## Kernel configuration

The `exynos2100-r9sxxx_defconfig` used by this tree enables:

- `CONFIG_COMPAT=y`
- `CONFIG_NFC_PN547=y`
- `CONFIG_NFC_FEATURE_SN100U=y`
- `CONFIG_NFC_PN547_ESE_SUPPORT=y`
- `CONFIG_SEC_NFC_COMPAT_IOCTL=y`

The last option exposes the driver's existing `.compat_ioctl` entry when
the kernel is built with `CONFIG_COMPAT`. The driver already recognizes
the 32-bit-sized ioctl command variants in its ioctl switch. This is a
32-bit userspace ABI compatibility change; it does not add NFC protocols
or card-emulation features.

## Layer boundaries

The Samsung `pn547` driver is an I2C transport, power/reset, IRQ and
secure-element coordination driver. Its presence does not by itself
implement ISO-DEP, NDEF, HCE, MIFARE Classic emulation, or key recovery.
Those protocol and application features belong in the supported controller
firmware and Android NFC HAL/framework stack.

For normal, authorized card-emulation testing, use Android's supported
Host Card Emulation APIs and a test reader/card profile. Do not treat the
kernel's firmware-download mode or raw device node as a general-purpose
card emulator.

## Validation checklist

These checks still need to be performed on a build and physical O1S device:

1. Build the kernel using the project's normal O1S build procedure and
   confirm the final config retains all five options listed above.
2. Boot the kernel and confirm the PN547 I2C device probes without errors.
3. Confirm the NFC device node is created and the stock/vendor NFC service
   starts successfully.
4. Exercise the vendor NFC service with its matching 32-bit or 64-bit
   userspace ABI, as applicable; verify power, IRQ, and normal tag discovery.
5. Test Android HCE with a self-owned test app and a test reader. Verify
   ISO-DEP APDU exchange and NDEF handling at the Android framework layer.
6. Re-test eSE/SPI access and NFC power transitions because the controller
   shares power/reset resources with the secure element.

No build, boot, RF, HCE, or eSE hardware test is claimed by this note.
