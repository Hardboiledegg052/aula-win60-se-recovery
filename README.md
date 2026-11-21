# AULA WIN 60 SE — recovery after failed firmware update

Community-documented USB bootloader recovery for **one confirmed AULA WIN 60 SE hardware revision**, using the manufacturer's **WIN 60 SE V0.1.0.1** application firmware and an independent browser-based PhotonLink flasher.

> **Status: one successful real-world recovery (November 21, 2026).** The original keyboard had no RGB and no working keys after a failed update, but Windows still enumerated it over USB. After flashing the matching official image, its owner confirmed **all keys and RGB work again**. This is not an official AULA tool and has not been validated on other keyboards.

## First: check that your keyboard is actually the same revision

| Item | Known recovered keyboard |
| --- | --- |
| Label/model | **WIN 60 SE** (not WIN60 HE PRO/MAX or an older Standard board) |
| USB VID / PID | `38A6:291A` |
| HID vendor interface | Usage page `FF00`, usage `0001` (`MI_02`) |
| HID report | 64-byte input/output |
| Bootloader board ID | `0x0204010440000000` |
| Bootloader run mode | `0` |
| Original bootloader version | `0.1.0.0` |
| Recovered application | **WIN 60 SE V0.1.0.1** |

**A matching USB VID/PID is necessary but NOT sufficient.** You must connect with the flasher and confirm the **exact eight-byte board ID** and bootloader mode before it enables writing. Do not assume firmware is interchangeable between similarly named AULA keyboards.

## Required firmware (not redistributed here)

Obtain the firmware for your exact model from the **official AULA HUB update service** or request it from AULA support. The specific firmware that restored the keyboard was listed as:

```
C002P004_WIN60SE_AppPacket_v0.1.0.1_20260929_a.bin
Size:   104632 bytes
SHA256: 99fa9df5f59ce752ab1bdf208f6144c1b5c2dcc4455bbdd69831d57285a4db9b
```

For verification, this BIN begins with a packet header: bytes at offset `0x04` are `5A BC DC 5A`. Its embedded board ID at offsets `0x08`–`0x0F` is `04 01 04 02 00 00 00 40` (two little-endian 32-bit words), which corresponds to the board ID reported over HID: `02 04 01 04 40 00 00 00`.

**The HTML flasher has an allowlist. It refuses all firmware files except the exact SHA-256 hash, size and header above.** Renaming another file cannot make it pass.

**Do not flash** the file `WIN60HE_STANDARD_RAW.bin` circulated for older WIN60HE Standard boards. It is from a different hardware/protocol generation. Our recovery did *not* use it.

## Recovery steps — Windows 11

**Risk warning:** Firmware writes can irreversibly damage a device, especially if power is lost or protocol assumptions differ between revisions. The flasher was tested with a simulated HID device and **one real keyboard recovery**. Proceed at your own risk; seek official support when possible.

1. Confirm that Windows still detects the keyboard as `USB\VID_38A6&PID_291A`. If it does not enumerate, **stop**; these steps have not been validated for that failure mode.
2. Obtain the exact BIN described above from AULA. Verify its filename, size and SHA-256.
3. Close AULA HUB and any other program using the keyboard's vendor HID interface.
4. Open [`WIN60_SE_PhotonLink_Guarded_Flasher.html`](WIN60_SE_PhotonLink_Guarded_Flasher.html) locally in the current version of **Microsoft Edge** or **Google Chrome** on Windows 11.
5. Click **Connect keyboard**, select the matching `38A6:291A` device, then click **Read board ID and bootloader mode**.
6. **Stop unless** it shows board `0x0204010440000000` and **BOOTLOADER mode `0`**.
7. Click **Choose file** and select the official `C002P004_WIN60SE_AppPacket_v0.1.0.1_20260929_a.bin`.
8. Confirm the log explicitly says **APPROVED FILE MATCH**. If rejected, **stop** and do not try a different BIN or bypass the check.
9. Read the warning and tick the risk acknowledgment. Type `WRITE 0x0204010440000000` in the confirmation field.
10. Click **Write selected firmware to keyboard**, and do **not unplug the keyboard or close the browser** while writing.
11. Wait for **Firmware integrity check returned OK**, then allow reboot to application mode. Check typing, RGB lighting and AULA HUB configuration.

If an error appears during transfer, **save the diagnostic log** and do not retry blindly. The keyboard may have a partially programmed application. Contact AULA support or open an issue with the *error text and non-personal technical data only*.

## What the flasher actually does

The single-file HTML utility uses Chromium WebHID; it makes **no network requests** and does not upload firmware/logs anywhere.

- Talks to the `38A6:291A / FF00:0001` HID interface.
- Sends a `SYNC` request, checking the board ID and bootloader run mode.
- Checks the selected BIN's exact hash, 104632-byte length, header magic and embedded board ID.
- Requires an explicit acknowledgment and typed board-ID confirmation.
- Rechecks the **live** board ID and firmware hash immediately before writing.
- Streams firmware in 58-byte chunks inside 64-byte HID frames; checks device acknowledgment after each chunk.
- Requests an integrity/commit check after all chunks; optionally asks the keyboard to reboot.

The source is inspectable in the HTML file. There is no downloaded executable, service, installer or bypass of another device's board restrictions.

## Known limitations

- **Not officially endorsed by AULA.** Recovery was confirmed for *one* WIN 60 SE with the exact hardware identifiers above.
- Tested against simulated HID responses during development; only the first owner-reported hardware recovery establishes real-world success.
- Doesn't fix boards not enumerating as USB, damaged bootloaders, electrical faults, or other board IDs.
- Doesn't unlock or patch incompatible images. It will not accept a different firmware version or filename unless it has the exact approved bytes.
- Errors after the first data packet can leave the application partially written.

## Responsible sharing

If reporting an issue, include the *model name*, *VID/PID*, *board ID*, *run mode*, *firmware hash* and *non-sensitive error messages*. **Remove keyboard serial numbers, Windows device instance IDs, and identifying paths** from any screenshots or logs you share publicly.

Manufacturer firmware is not included in this repository. Obtain it directly from AULA, and follow its support guidance if your board identifiers are different.
