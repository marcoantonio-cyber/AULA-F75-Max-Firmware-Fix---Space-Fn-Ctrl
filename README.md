# AULA F75 Max — Fn / Right Ctrl Firmware Fix

**Final release: v1.0.0**

This project documents a two-byte modification to the **original AULA F75 Max firmware** that restores the bottom-right key assignments to **Fn | Right Ctrl**.

## The issue

On my AULA F75 Max, an official firmware update changed the keyboard's functional layout:

```text
Original layout:        Space | Fn        | Right Ctrl
After official update:  Space | Right Alt | Fn
Final corrected layout: Space | Fn        | Right Ctrl
```

## The fix

The firmware image is stored inside the manufacturer's Windows updater (`F75MAX firmware.exe`) as an `RCDATA` resource with ID `4000` (`4000.bin`, **262,144 bytes / 256 KiB**).

Two key-mapping bytes in the original firmware image were changed:

| Hex offset | Original byte | Corrected byte | Corrected key |
| --- | --- | --- | --- |
| `0x13CDD` | `E6` | `AF` | `Fn` |
| `0x13CE5` | `AF` | `E4` | `Right Ctrl` |

Both changes are needed. Changing only the first byte produced **two Fn keys** in an earlier test. The final version includes **both modifications** and restores the intended `Fn | Right Ctrl` assignment on the tested layout.

## How it was made

1. Open the original AULA F75 Max firmware updater in **Resource Hacker**.
2. Locate `RCDATA → 4000 → 0` and extract its `4000.bin` firmware image.
3. Change the **two bytes** at the offsets listed above, leaving the rest of the firmware image unchanged.
4. Replace the `4000` resource in a copy of the updater with the modified `4000.bin`.
5. Save the updated executable and use it to update the compatible keyboard.

See [TECHNICAL_NOTES.md](TECHNICAL_NOTES.md) for the original file identifier, verification commands, and more precise details.

## Compatibility and warning

- **Keyboard model:** AULA **F75 Max** (not F75 or F75 Pro).
- **USB VID/PID observed on the tested device:** `0C45:800A`.
- **USB revision descriptor observed:** `REV_0110` (not necessarily the physical PCB revision).
- **Other hardware revisions have not been verified.** Flashing incompatible firmware can damage or disable a keyboard.

This is an **unofficial community fix**, not a release supported or endorsed by AULA or EPOMAKER. Original firmware and updater rights remain with their respective owners.
