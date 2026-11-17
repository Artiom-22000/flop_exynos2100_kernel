# o1s SN100U NFC transport diagnostics

## Scope

This is a diagnostic plan for a Galaxy S21 5G (Exynos 2100, o1s) using
the PN547 Linux device driver and SN100U controller. It is intended for
controlled interoperability tests with cards owned by the tester.

The driver is an I2C transport layer. It does not expose a separate RF
monitor interface. Mirroring the complete I2C payload into a userspace
capture file would also collect arbitrary NFC exchanges, including
application data and authentication material. This project therefore
does not add an unrestricted raw-payload capture interface.

## What can be measured safely

A future opt-in diagnostic hook can record metadata only:

- monotonic timestamp;
- direction (host-to-controller or controller-to-host);
- I2C transfer result and byte count;
- NCI message class/opcode when the transfer is a complete, recognizable
  NCI packet;
- whether the packet is a command, response, notification, or data packet.

It must not store payload bytes, UIDs, AIDs, NDEF contents, keys, or
authentication challenge/response values. It must be disabled by default,
bounded in memory, and must not change the bytes, timing, or return values
of the normal HAL path. It must not add a second reader of /dev/pn547.

## Why this matters for MIFARE Classic

The NXP SNxxx HAL implementation uses a proprietary MIFARE authentication
request for Classic authentication (the HAL maps the Classic AUTH A/B
commands to an NFCC MIFARE-auth request). The controller performs the RF
authentication and the host receives an authentication result. Raw
transceive operations use a different path. Consequently, an I2C trace
cannot be assumed to contain the RF nonce exchange required by offline
MIFARE authentication analysis.

Reference implementation:
https://android.googlesource.com/platform/hardware/nxp/nfc/+/9c20c2d13bbb5ec090f4335a0627ffb26c137344/snxxx/halimpl/mifare/NxpMfcReader.cc

The exact Samsung SN100U HAL and controller firmware on a particular
device may differ from the public reference implementation.

## Controlled test procedure

1. Build and boot the kernel with the existing o1s NFC configuration.
2. Confirm Android NFC is healthy before enabling any diagnostic facility.
3. Use a disposable, tester-owned MIFARE Classic card and a simple reader
   operation (poll, identify technology, then a normal read of a known
   test block where the card's access conditions permit it).
4. Record metadata-only diagnostics and the corresponding Android NFC
   result. Do not use payment cards, access badges, transit cards, or
   cards belonging to anyone else.
5. Compare the NCI message sequence and response status with the HAL's
   expected command flow. Metadata can establish which controller
   operations occur; it cannot establish that RF authentication nonces
   were exposed to the host.
6. If a later experiment requires payload-level RF analysis, use a
   dedicated NFC development platform and test fixtures designed to
   expose RF-layer frames, rather than adding a general-purpose capture
   facility to the phone's production NFC path.

## Current status

No metadata hook is implemented by this document. No kernel build, boot,
or hardware validation is claimed. The existing driver still forwards
ordinary I2C bytes between the HAL and controller, subject to its
documented internal eSE response handling.
