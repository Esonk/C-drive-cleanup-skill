---
name: windows-c-drive-cleanup
description: Safely inspect and free space on a Windows C drive by performing read-only disk usage scans, classifying system files versus removable caches, and using explicit consent gates before deleting, cleaning, or junction-migrating directories to another drive. Use when a user says their C drive is full, asks what can be deleted, asks what must not be touched, or wants to move application caches/extensions to D or another data drive with junctions.
---

# Windows C Drive Cleanup

## Overview

Use this skill to guide Windows C drive cleanup without damaging the operating system, installed software, or user data. The default posture is read-only diagnosis first, then user-approved changes in small, reversible units.

## Safety Rules

- Start with read-only scans. Do not delete, move, compress, disable features, or create junctions until the user explicitly approves that exact action.
- For junction migrations, get approval one source directory at a time. Do not treat approval for one directory as approval for the next.
- Prefer native PowerShell/.NET APIs for size scans. Skip reparse points when computing real C drive usage, or junction targets on D may be double-counted as C usage.
- Never manually delete `C:\Windows\WinSxS`, `C:\Windows\Installer`, `C:\Windows\System32`, `C:\Windows\SysWOW64`, `C:\Recovery`, `C:\System Volume Information`, `C:\Program Files` application folders, or `C:\Program Files (x86)` application folders.
- Do not manually delete `C:\hiberfil.sys`, `C:\pagefile.sys`, or `C:\swapfile.sys`. Explain system-controlled options instead, such as disabling hibernation if the user chooses.
- Treat security software and OEM utility folders under `C:\ProgramData` as higher risk. Prefer vendor UI cleanup or only remove clearly identified old logs after separate confirmation.
- If a file is locked, skip it. Do not force-close processes unless the user explicitly authorizes that.

## Workflow

1. Measure free space and top-level usage.
2. Break down the largest directories: usually `Windows`, `Users`, root system files, `Program Files`, `ProgramData`, and `Program Files (x86)`.
3. Separate real C usage from existing junctions by detecting reparse points.
4. Classify findings into:
   - must not touch
   - safe cache/temp cleanup
   - risky logs/vendor data
   - junction migration candidates
5. Present a phased plan with estimated reclaimable space.
6. Execute only the phase or individual target the user approves.
7. After each change, report before/after size, final free space, and what was skipped.

## Read-Only Scan Pattern

Use `.NET` `DriveInfo` if CIM/WMI access is denied:

```powershell
$drive = [System.IO.DriveInfo]::new('C')
[pscustomobject]@{
  Drive = 'C:'
  TotalGB = [math]::Round($drive.TotalSize / 1GB, 2)
  FreeGB = [math]::Round($drive.AvailableFreeSpace / 1GB, 2)
  UsedGB = [math]::Round(($drive.TotalSize - $drive.AvailableFreeSpace) / 1GB, 2)
  FreePct = [math]::Round(100 * $drive.AvailableFreeSpace / $drive.TotalSize, 1)
}
```

When calculating directory sizes, skip child reparse points:

```powershell
function Get-TreeSizeBytes {
  param([string]$Path)
  if (-not (Test-Path -LiteralPath $Path)) { return [int64]0 }
  $item = Get-Item -LiteralPath $Path -Force -ErrorAction SilentlyContinue
  if (-not $item) { return [int64]0 }
  if (($item.Attributes -band [System.IO.FileAttributes]::ReparsePoint) -ne 0) { return [int64]0 }
  $sum = [int64]0
  Get-ChildItem -LiteralPath $Path -Force -Recurse -File -ErrorAction SilentlyContinue |
    ForEach-Object { $sum += $_.Length }
  return $sum
}
```

Also list reparse points before making recommendations:

```powershell
Get-ChildItem -LiteralPath 'C:\Users\<user>' -Force -Directory -Recurse -ErrorAction SilentlyContinue |
  Where-Object { ($_.Attributes -band [System.IO.FileAttributes]::ReparsePoint) -ne 0 } |
  Select-Object Mode, LinkType, Target, FullName
```

## Typical Classifications

System-controlled or do-not-touch:

- `C:\Windows\WinSxS`: component store; use Disk Cleanup or DISM only.
- `C:\Windows\Installer`: repair/uninstall cache; do not hand-delete.
- `C:\Windows\System32`, `C:\Windows\SysWOW64`, `C:\Windows\servicing`: core OS.
- `C:\hiberfil.sys`: hibernation file; explain hibernation versus sleep and let the user decide.
- `C:\pagefile.sys`: page file; changing it is optional and not a first-line cleanup.

Lower-risk cleanup candidates:

- `%LOCALAPPDATA%\Temp`
- `%LOCALAPPDATA%\npm-cache`
- `%LOCALAPPDATA%\pip\cache`
- Browser model/component caches such as Chrome `OptGuideOnDeviceModel`
- App updater caches named like `*-updater`
- Crash dumps and old logs only when clearly identified as logs

Junction migration candidates:

- Editor extension folders such as `.vscode\extensions`, `.cursor\extensions`, `.antigravity\extensions`
- Large tool caches such as `.cache\codex-runtimes`, but avoid migrating a cache while the current agent/session is using it
- Development tool metadata such as `.p2`, after confirming related apps are closed

Higher-risk candidates:

- `C:\ProgramData\Package Cache`
- OEM utility data under `C:\ProgramData\Comms`, `HonorSDK`, PC manager folders
- Security software logs under vendor folders
- Browser full user profiles, unless the user accepts profile migration risk and closes browsers

## Safe Cache Cleanup

Before deleting, show exact target paths and estimated sizes. After approval, remove only approved targets. For Temp, delete children rather than the Temp directory itself.

Example target set:

```powershell
$targets = @(
  "$env:LOCALAPPDATA\Temp",
  "$env:LOCALAPPDATA\npm-cache",
  "$env:LOCALAPPDATA\pip\cache",
  "$env:LOCALAPPDATA\Google\Chrome\User Data\OptGuideOnDeviceModel"
)
```

Deletion pattern:

```powershell
foreach ($t in $targets) {
  if (Test-Path -LiteralPath $t) {
    if ($t -ieq "$env:LOCALAPPDATA\Temp") {
      Get-ChildItem -LiteralPath $t -Force -ErrorAction SilentlyContinue |
        Remove-Item -Recurse -Force -ErrorAction SilentlyContinue
    } else {
      Remove-Item -LiteralPath $t -Recurse -Force -ErrorAction SilentlyContinue
    }
  }
}
```

Report before/after size for each target, not just total drive free space.

## Junction Migration Procedure

Use this only after the user approves one exact source directory and destination. The safest destination pattern is a dedicated folder such as `D:\AppDataMove\<name>`.

Before migration:

- Confirm the source exists and is not already a reparse point.
- Confirm the destination is absent or empty.
- Confirm the destination drive has enough free space.
- Check likely owning processes and abort if they are running, for example `Code`, `Cursor`, `antigravity`, or `eclipse`.

Migration steps:

1. Copy with `robocopy /MIR /XJ /COPY:DAT /DCOPY:DAT /R:2 /W:2`.
2. Compare source and destination file count and total bytes.
3. Rename the source to a timestamped backup.
4. Create the junction with `cmd /c mklink /J`.
5. Verify the source is now a reparse point.
6. Delete the backup only after verification succeeds.
7. Report destination, junction target, copied file count, size, and C free space.

Do not use `Move-Item` as the first operation. Copy and verify first so the original directory can be restored if junction creation fails.

## Consent Language

For cleanup, ask for approval like:

`Please confirm: delete only these cache/temp targets: <list>.`

For migration, ask for approval like:

`Please reply exactly: agree to migrate <directory description>.`

After one migration completes, stop and ask for the next directory. Do not continue automatically.

## Reporting Template

Include:

- Current C total, used, free, and free percentage.
- Largest real C directories and any skipped access errors.
- Existing junctions that should not be counted as C usage.
- A do-not-touch list.
- A low-risk cleanup list with estimated GB.
- A junction migration list with estimated GB and required app closures.
- Final before/after numbers after every approved action.
