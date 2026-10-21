# Technical Notes — AULA F75 Max Fn / Right Ctrl Fix (v1.0.0)

## 1. Original firmware and device

| Parameter | Value |
| --- | --- |
| Target keyboard | AULA F75 Max |
| USB VID | `0C45` |
| USB PID | `800A` |
| Observed USB revision descriptor | `REV_0110` |
| Original Windows updater examined | `F75MAX firmware.exe` |
| SHA-256 of that original updater | `A6AC9D2E4DC1E14C6F3002F40BC889B553018DD205E857BB18283DAB3338D6AC` |
| Embedded resource type | `RCDATA` (`RT_RCDATA`) |
| Embedded resource ID | `4000` (language `0`) |
| Extracted firmware image | `4000.bin` |
| Firmware image length | `262,144` bytes (`256 KiB`) |

The SHA-256 above identifies the **original installer examined**, not the final modified executable. The USB revision descriptor is not proof of a PCB revision.

## 2. Final changes to `4000.bin`

The fix changes exactly **two bytes** in the extracted firmware image. Offsets below are hexadecimal byte positions within `4000.bin`.

| Offset (hex) | Before (hex) | After (hex) | Resulting key assignment |
| --- | --- | --- | --- |
| `0x13CDD` | `E6` | `AF` | `Fn` |
| `0x13CE5` | `AF` | `E4` | `Right Ctrl` |

These bytes correct the bottom-right functional layout from:

```text
Space | Right Alt | Fn
```

to:

```text
Space | Fn | Right Ctrl
```

**Why two changes?** The first experiment modified only `0x13CDD` (`E6 → AF`). It successfully flashed, but produced **two Fn keys**. The final fix also modifies `0x13CE5` (`AF → E4`) to restore the Right Ctrl assignment.

## 3. Editing and repacking

1. Keep an untouched copy of the original vendor updater.
2. Open a working copy of `F75MAX firmware.exe` in **Resource Hacker**.
3. Navigate to `RCDATA → 4000 → 0`; export the binary resource as `4000.bin`.
4. In a hex editor, confirm that offsets `0x13CDD` and `0x13CE5` contain `E6` and `AF`, respectively. **Do not proceed if they differ.**
5. Change those bytes to `AF` and `E4`, respectively. Do not change any other bytes. Save the edited image as a separate file.
6. In Resource Hacker, replace the `RCDATA / 4000 / 0` resource with the edited firmware image and save the resulting updater as a separate `.exe`.
7. Use that updater only on a device whose firmware and hardware compatibility have been verified.

The modification is to the **firmware image embedded inside the installer**, rather than a remapping in Windows, the keyboard driver, or an XML configuration file.

## 4. Verification

Check the original updater SHA-256 in PowerShell:

```powershell
Get-FileHash '.\F75MAX firmware.exe' -Algorithm SHA256
```

Compare the untouched and edited firmware images:

```powershell
fc.exe /b '.\4000.bin' '.\4000_fixed.bin'
```

Use **`fc.exe`**, not just `fc`, in PowerShell: `fc` can resolve to the `Format-Custom` alias. The comparison should show **only the two byte changes above**. Confirm both images remain exactly `262,144` bytes long.

The final corrected layout for this release is `Space | Fn | Right Ctrl`. Successful flashing by itself does not establish compatibility with other F75 Max revisions. Publish a SHA-256 of the *exact final release file* alongside any download so that others can verify its integrity.

## 5. Project scope

This documents the final **two-byte Fn / Right Ctrl fix** for the tested AULA F75 Max layout. It does not claim compatibility with the standard AULA F75, F75 Pro, or every F75 Max hardware revision. This project is unofficial and is not affiliated with AULA or EPOMAKER.
