# Windows C Drive Cleanup Skill

[中文说明](README.zh-CN.md)

A Codex skill for safely diagnosing, cleaning, and junction-migrating Windows C drive storage.

Stop guessing what to delete from `C:`. This skill guides Codex to scan first, classify risk, explain what must not be touched, and ask for explicit approval before deleting caches or moving folders to another drive.

## Why This Exists

Windows C drive cleanup is risky because the largest folders are often system-managed. Deleting the wrong thing can break Windows updates, uninstallers, development tools, browser profiles, or security software.

This skill turns cleanup into a conservative workflow:

- Read-only scan first
- Clear separation between system files, caches, logs, and migration candidates
- No deletion without explicit user approval
- One junction migration requires one separate approval
- Copy and verify before replacing a folder with a junction
- Skip locked files instead of force-closing apps

## What It Can Help With

- Inspect C drive free space and top-level usage
- Identify real C drive usage while avoiding junction double-counting
- Explain which Windows files and folders must not be manually deleted
- Find low-risk cleanup targets such as Temp, pip cache, npm cache, browser component caches, and updater caches
- Plan junction migrations for editor extensions and development tool caches
- Report before/after free space after each approved action

## What It Will Not Do

- It will not manually delete `C:\Windows\WinSxS`, `C:\Windows\Installer`, `System32`, `SysWOW64`, `pagefile.sys`, `swapfile.sys`, or `hiberfil.sys`.
- It will not bulk-delete OEM or security software data without review.
- It will not migrate multiple folders after a single approval.
- It will not force-close applications unless the user explicitly authorizes it.

## Example Result

A real cleanup session using this workflow improved a Windows C drive from low free space to a healthier state:

| Step | Free Space |
|---|---:|
| Initial scan | 15.84 GB |
| After disabling hibernation manually and approved cache cleanup | 35.23 GB |
| After approved junction migrations | 39.40 GB |

See [example scan report](examples/scan-report.en.md) for a sanitized walkthrough.

## Installation

Clone this repository into your Codex skills directory:

```powershell
git clone https://github.com/Esonk/C-drive-cleanup-skill `
  "$env:USERPROFILE\.codex\skills\windows-c-drive-cleanup"
```

Restart Codex if needed so the skill can be discovered.

## Example Prompts

```text
My C drive is almost full. Use the Windows C Drive Cleanup skill to scan it and give me a safe cleanup plan. Do not modify anything before I approve it.
```

```text
Scan my C drive, tell me which files are system files, which caches can be cleaned, and which folders can be moved to D with junctions.
```

```text
Help me migrate one approved editor extension folder from C to D using a junction. Ask me before each migration.
```

## Safety Model

The skill is designed around consent gates:

1. Scan first, using read-only commands.
2. Classify findings by risk.
3. Present a phased plan.
4. Ask for explicit approval for each cleanup phase or each individual junction migration.
5. For junction migration, copy to the destination first, verify file count and bytes, then create the junction.
6. Report exact before/after numbers.

## Recommended GitHub Topics

Add these topics to improve discoverability:

`codex-skill`, `windows`, `disk-cleanup`, `powershell`, `junction`, `storage-management`, `c-drive`, `developer-tools`

## License

MIT License. See [LICENSE](LICENSE).
